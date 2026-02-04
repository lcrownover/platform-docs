# Puppet Fundamentals

By the end of this section, you'll understand how Puppet thinks about system configuration, what resources are, and how the agent-server architecture works.

## Declarative vs. Imperative

Before diving into Puppet specifics, it's worth understanding the mindset shift Puppet requires.

When you configure a server by hand, you think imperatively: "Install nginx, then create this config file, then start the service." You're describing *steps to take*.

Puppet thinks declaratively: "Nginx should be installed. This config file should exist with these contents. The nginx service should be running." You're describing *how things should be*, and Puppet figures out what steps are needed to get there.

This distinction matters because:

- **Order becomes flexible.** Puppet determines the right order based on dependencies you declare, not the order you write things.
- **Idempotency is automatic.** Running Puppet twice doesn't install nginx twice. If nginx is already installed, Puppet does nothing. If the config file already has the right contents, Puppet leaves it alone.
- **Drift gets corrected.** If someone manually changes the config file, Puppet changes it back on the next run.

## Resources: The Building Blocks

Everything Puppet manages is a **resource**. A resource is a single, discrete thing on a system: a package, a file, a service, a user, a cron job. Each resource has a **type** (what kind of thing it is) and **attributes** (how it should be configured).

Here's a simple resource declaration:

```puppet
package { 'nginx':
  ensure => installed,
}
```

This declares a resource of type `package`, with the title `nginx`, and one attribute: `ensure => installed`. Puppet reads this as "the package named nginx should be installed."

Let's look at a few more:

```puppet
file { '/etc/nginx/nginx.conf':
  ensure  => file,
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  content => "worker_processes auto;\n",
}
```

```puppet
service { 'nginx':
  ensure => running,
  enable => true,
}
```

Each resource is independent. Puppet doesn't care what order you write them in your code. But of course, in the real world, order matters. You can't start nginx before it's installed. That's where relationships come in.

## Resource Relationships

Puppet provides several ways to declare that one resource depends on another. The most common are `require` and `notify`.

**require** says "don't apply this resource until that one is done":

```puppet
service { 'nginx':
  ensure  => running,
  require => Package['nginx'],
}
```

**notify** says "if this resource changes, refresh that one":

```puppet
file { '/etc/nginx/nginx.conf':
  ensure => file,
  notify => Service['nginx'],
}
```

When a file notifies a service, Puppet restarts the service whenever the file's contents change. This is how you get configuration changes to take effect.

!!! note
    Notice the capitalization difference. When you *declare* a resource, the type is lowercase: `package { 'nginx': }`. When you *reference* an existing resource, the type is capitalized: `Package['nginx']`.

## The Package → File → Service Pattern

Most system configuration follows a simple pattern:

1. **Install a package** (get the software on the system)
2. **Manage configuration files** (tell the software how to behave)
3. **Ensure the service is running** (and restart it when config changes)

Here's a complete example:

```puppet
package { 'nginx':
  ensure => installed,
}

file { '/etc/nginx/nginx.conf':
  ensure  => file,
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  source  => 'puppet:///modules/nginx/nginx.conf',
  require => Package['nginx'],
  notify  => Service['nginx'],
}

service { 'nginx':
  ensure  => running,
  enable  => true,
  require => Package['nginx'],
}
```

This pattern handles the full lifecycle:

- The package gets installed first (other resources `require` it)
- The config file is laid down (after the package creates the directory structure)
- The service runs and stays running
- When the config file changes, the service restarts automatically

You'll use this pattern constantly. Most Puppet work is some variation of "install thing, configure thing, run thing."

## How Puppet Applies Configuration

When Puppet runs on a managed server, here's what happens:

1. **The agent gathers facts** about the system (OS, IP address, memory, etc.)
2. **The agent contacts the Puppet server** and says "here are my facts, what should I look like?"
3. **The server compiles a catalog** by evaluating your Puppet code against those facts
4. **The agent receives the catalog** (a JSON document listing every resource and its desired state)
5. **The agent applies the catalog**, making changes as needed
6. **The agent sends a report** back to the server describing what it did

The catalog is key. Your Puppet code doesn't run on the managed servers. Instead, the server compiles it into a catalog, and the agent applies that catalog. This two-phase approach is why Puppet can catch syntax errors before anything happens on your servers.

By default, agents check in every 30 minutes. You can also trigger a run manually:

```bash
puppet agent -t
```

The `-t` flag (short for `--test`) runs the agent in the foreground with verbose output, which is useful for testing and debugging.

### Applying Manifests Locally with puppet apply

You don't always need a Puppet server. The `puppet apply` command compiles and applies a manifest directly on the local machine:

```bash
puppet apply manifest.pp
```

This skips the agent-server communication entirely. Puppet compiles the manifest locally, builds a catalog, and applies it right there.

!!! note
    `puppet apply` is useful for development and learning, but it has limitations. Features that depend on a Puppet server (like exported resources, PuppetDB queries, and centralized reporting) won't work. In production, you'll use the agent-server model. But for experimenting with manifests and learning the language, `puppet apply` is a great way to get fast feedback without setting up infrastructure.

## Facts and Facter

In step 1 above, the agent "gathers facts." But where do these facts come from?

**Facter** is a companion tool that ships with Puppet. It inspects the system and returns structured data about it: the operating system, IP addresses, memory, CPU count, disk space, and hundreds of other details. When the Puppet agent runs, it calls Facter first, then sends those facts to the server.

You can run Facter directly to see what it knows:

```bash
facter
```

This dumps all facts. To see a specific fact:

```bash
facter os.family
```

You'll see something like:

```
RedHat
```

Or for more detail:

```bash
facter os
```

```yaml
{
  architecture => "x86_64",
  distro => {
    codename => "Maipo",
    description => "Red Hat Enterprise Linux Server release 7.9 (Maipo)",
    id => "RedHatEnterpriseServer",
    release => {
      full => "7.9",
      major => "7",
      minor => "9"
    }
  },
  family => "RedHat",
  hardware => "x86_64",
  name => "RedHat",
  release => {
    full => "7.9",
    major => "7",
    minor => "9"
  },
  selinux => {
    enabled => true
  }
}
```

### Using Facts in Puppet Code

Facts are available as variables in your Puppet code. This lets you write conditional logic based on the system you're configuring:

```puppet
if $facts['os']['family'] == 'RedHat' {
  $apache_package = 'httpd'
  $apache_service = 'httpd'
} elsif $facts['os']['family'] == 'Debian' {
  $apache_package = 'apache2'
  $apache_service = 'apache2'
}

package { $apache_package:
  ensure => installed,
}

service { $apache_service:
  ensure => running,
  enable => true,
}
```

This single piece of code works on both Red Hat and Debian-based systems because it adapts based on facts.

### Common Facts

| Fact | What It Contains |
|------|------------------|
| `$facts['os']['family']` | OS family (RedHat, Debian, Windows, etc.) |
| `$facts['os']['release']['major']` | Major OS version (7, 8, 22, etc.) |
| `$facts['networking']['ip']` | Primary IP address |
| `$facts['networking']['fqdn']` | Fully qualified domain name |
| `$facts['memory']['system']['total']` | Total system memory |
| `$facts['processors']['count']` | Number of CPU cores |

You'll also see older code using a shorthand syntax like `$osfamily` or `$ipaddress`. This still works but the `$facts` hash is the modern, preferred approach.

### Custom Facts

You can also define your own facts. This is useful when you need information that Facter doesn't provide out of the box, like which application tier a server belongs to or what datacenter it's in. See [Custom Facts](07-custom-facts.md) for details on creating and deploying your own facts.

## Common Resource Types

Puppet has dozens of built-in resource types. Here are the ones you'll use most often:

| Type | What It Manages | Common Attributes |
|------|-----------------|-------------------|
| `package` | Software packages | `ensure` (installed, absent, latest, or a version) |
| `file` | Files and directories | `ensure`, `owner`, `group`, `mode`, `content`, `source` |
| `service` | System services | `ensure` (running, stopped), `enable` (start on boot) |
| `user` | User accounts | `ensure`, `uid`, `gid`, `home`, `shell` |
| `group` | Groups | `ensure`, `gid` |
| `cron` | Cron jobs | `command`, `hour`, `minute`, `user` |
| `exec` | Arbitrary commands | `command`, `creates`, `unless`, `onlyif` |

!!! warning
    The `exec` resource type runs arbitrary commands, which can seem like an escape hatch when you don't know how to do something the "Puppet way." Use it sparingly. Commands aren't inherently idempotent, so you need to use guards like `creates`, `unless`, or `onlyif` to prevent them from running every time. If you find yourself using `exec` frequently, there's probably a better approach.

## Exercises

1. **Explore Facter.** Run `facter` on a Puppet-managed server and browse the output. Find the facts for the OS family, IP address, and total memory. Try `facter --json` for machine-readable output.

2. **Identify resources on your system.** Pick a service running on a server you manage (sshd, httpd, whatever). List the resources Puppet would need to manage it: what packages, what config files, what service?

3. **Trace the dependencies.** For the service you picked, draw out the relationship chain. What needs to happen before what? What should trigger a restart?

4. **Read a catalog.** On a Puppet-managed server, run `puppet agent -t --noop` (no-op mode, makes no changes). Read the output. Can you identify the resources being evaluated and whether Puppet would change them?
