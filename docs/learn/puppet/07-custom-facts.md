# Custom Facts

By the end of this section, you'll know how to create your own facts to expose environment-specific information to your Puppet code.

## Why Custom Facts?

This is where some of the real magic happens in Puppet.

Facter provides a wealth of system information out of the box, but it only knows what it can discover from the system itself. It can tell you the OS and IP address, but it has no idea:

- Which application tier this server belongs to (web, app, database)
- What datacenter or cloud region it's in
- Whether it's a production, staging, or development box
- What team owns it
- Any application-specific metadata

Custom facts bridge this gap. They let you **tag your systems** with metadata that matters to your organization, then use that metadata to drive Puppet's behavior.

Think about what becomes possible:

- **Group servers to share configuration.** Tag a set of servers with `app_group=banner` and they all pick up the same Hiera data, same firewall rules, same monitoring thresholds. Add a new server to the group by adding one fact file, and it inherits everything.
- **Assign patching windows.** Tag servers with `patch_window=sunday_02` or `patch_window=wednesday_22`, then use that fact to schedule maintenance. Your patching automation reads the fact and knows exactly when each server should be updated.
- **Track ownership and responsibility.** Tag servers with `owner=middleware-team` or `cost_center=12345`. Now you can generate reports, apply team-specific sudo rules, or route alerts to the right people.
- **Control feature rollouts.** Tag a subset of servers with `feature_new_monitoring=true` to opt them into a new configuration before rolling it out everywhere.
- **Derive facts from other sources.** Query your CMDB, read from cloud instance metadata, or parse hostnames to extract meaning. A server named `web-prod-dc1-001` could have facts automatically extracted: `role=web`, `env=prod`, `datacenter=dc1`.

Custom facts turn Puppet from "configure all servers the same way" into "configure each server appropriately based on what it is." This is how you manage hundreds or thousands of servers with a single codebase. The facts carry the context, and your Puppet code responds to that context.

## Types of Custom Facts

Puppet supports several ways to create custom facts:

| Type | Where It Lives | Best For |
|------|----------------|----------|
| External facts | Files in `/etc/facter/facts.d/` | Simple key-value pairs, easy to deploy |
| Ruby facts | Ruby files in module `lib/facter/` | Complex logic, derived values |
| Executable facts | Scripts in `/etc/facter/facts.d/` | When you need shell commands |

For most use cases, **external facts** are the simplest and most maintainable option.

## External Facts

External facts are files that Facter reads from `/etc/facter/facts.d/` (or `/opt/puppetlabs/facter/facts.d/` on newer installations). The simplest format is a text file with key-value pairs.

Create a file `/etc/facter/facts.d/environment.txt`:

```
role=webserver
tier=frontend
datacenter=dc1
owner=platform-team
```

Now run Facter:

```bash
facter role tier datacenter owner
```

```
datacenter => dc1
owner => platform-team
role => webserver
tier => frontend
```

These facts are now available in your Puppet code as `$facts['role']`, `$facts['tier']`, etc.

### YAML and JSON Formats

You can also use YAML or JSON for structured data. Create `/etc/facter/facts.d/server_info.yaml`:

```yaml
---
server_info:
  role: webserver
  tier: frontend
  datacenter: dc1
  owner: platform-team
  contacts:
    - alice@example.com
    - bob@example.com
```

This creates a nested fact structure accessible as `$facts['server_info']['role']`, `$facts['server_info']['contacts']`, etc.

### Executable Facts

If you need to compute a fact dynamically, you can use an executable script. Create `/etc/facter/facts.d/appversion.sh`:

```bash
#!/bin/bash
echo "app_version=$(rpm -q myapp --qf '%{VERSION}')"
```

Make it executable:

```bash
chmod +x /etc/facter/facts.d/appversion.sh
```

Facter runs the script and parses the output as key-value pairs. Now `$facts['app_version']` contains the installed version of your application.

!!! warning
    Executable facts run on every Facter invocation, which happens at the start of every Puppet run. Keep them fast. A slow fact slows down every Puppet run on that server.

## Ruby Facts

For complex logic or when you need to derive facts from other facts, you can write facts in Ruby. These live in your Puppet module under `lib/facter/`.

Create `lib/facter/memory_tier.rb` in your module:

```ruby
Facter.add(:memory_tier) do
  setcode do
    memory_gb = Facter.value(:memory)['system']['total_bytes'] / (1024 ** 3)

    case memory_gb
    when 0..4
      'small'
    when 5..16
      'medium'
    when 17..64
      'large'
    else
      'xlarge'
    end
  end
end
```

This creates a `memory_tier` fact that categorizes servers by RAM. The fact is computed on the agent, so it's available during catalog compilation.

### Confining Facts

Sometimes a fact only makes sense on certain systems. Use `confine` to limit when a fact runs:

```ruby
Facter.add(:systemd_version) do
  confine kernel: 'Linux'
  confine do
    File.exist?('/run/systemd/system')
  end

  setcode do
    Facter::Core::Execution.execute('systemctl --version').lines.first.split.last
  end
end
```

This fact only runs on Linux systems with systemd.

## Deploying Custom Facts

How you deploy facts depends on their type:

**External facts** can be deployed by Puppet itself. Create a file resource that manages the fact file:

```puppet
file { '/etc/facter/facts.d/environment.txt':
  ensure  => file,
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  content => "role=${server_role}\ntier=${server_tier}\n",
}
```

!!! note
    There's a chicken-and-egg issue here: the fact won't exist until after the first Puppet run deploys it. Plan your code accordingly, and use `if $facts['role']` checks to handle the case where the fact doesn't exist yet.

**Ruby facts** are distributed as part of your Puppet module. Place them in `lib/facter/` and they're automatically synced to agents via pluginsync.

## Using Custom Facts in Hiera

Custom facts become powerful when combined with Hiera. You can structure your Hiera hierarchy around your custom facts:

```yaml
# hiera.yaml
---
version: 5
hierarchy:
  - name: "Per-node data"
    path: "nodes/%{trusted.certname}.yaml"

  - name: "Per-role data"
    path: "roles/%{facts.role}.yaml"

  - name: "Per-datacenter data"
    path: "datacenters/%{facts.datacenter}.yaml"

  - name: "Per-tier data"
    path: "tiers/%{facts.tier}.yaml"

  - name: "Common data"
    path: "common.yaml"
```

Now you can put role-specific configuration in `data/roles/webserver.yaml`, datacenter-specific settings in `data/datacenters/dc1.yaml`, and so on. Hiera automatically picks the right data based on each server's facts.

We cover Hiera in depth in [Hiera and Data Separation](04-hiera-and-data-separation.md).

## Best Practices

- **Keep facts simple.** A fact should return a single piece of information. If you're tempted to put logic in a fact, consider whether that logic belongs in your Puppet code instead.
- **Use consistent naming.** Decide on a naming convention (snake_case is typical) and stick with it.
- **Document your facts.** Other team members need to know what facts exist and what values they can have.
- **Avoid secrets in facts.** Facts are visible in PuppetDB and reports. Never put passwords, API keys, or other secrets in facts.
- **Test fact availability.** Always check if a fact exists before using it, especially for custom facts that might not be deployed everywhere yet.

## Exercises

1. **Create a simple external fact.** On a test server, create `/etc/facter/facts.d/test.txt` with a few key-value pairs. Run `facter` to verify they appear.

2. **Create a YAML fact.** Create a structured fact in YAML format with nested values. Access the nested values with `facter fact_name.nested_key`.

3. **Plan your facts.** Think about your environment. What information would be useful to have as facts? Server roles? Application names? Team ownership? Sketch out a fact strategy before implementing.
