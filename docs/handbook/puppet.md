# Puppet

This document covers how Puppet is organized and used at UO. Read the [Puppet training modules](../learn/puppet/00-index.md) first to understand the fundamentals.

This handbook covers:

- How the delegated administration model works
- Team module structure and organization
- Writing roles and profiles
- Hiera data and the custom facts that drive lookups
- Making and deploying changes

## Overview

UO uses a delegated administration model for Puppet. There's a central **control repository** that defines the overall structure, and each team has their own **team module** where they manage their servers.

- **Control repo** (`puppet-control-repo`): Read-only to most users, but you can view it and submit PRs. Contains the Hiera hierarchy, site manifest, and shared profiles.
- **Team modules** (`puppet_dba`, `puppet_nts`, etc.): Each team owns their module. This is where you write roles and profiles for your servers.

You'll spend most of your time working in your team's module.

### How Servers Get Their Configuration

When a Puppet agent runs:

1. The server has a `pp_role` fact set (e.g., `puppet_dba::role::loire`)
2. The site manifest includes that role class
3. The role includes profiles, which include modules
4. Hiera provides data based on the server's `pp_group` and `pp_cluster` facts

The `pp_role` fact determines everything. If it's not set, Puppet logs a warning and does nothing. See [The pp_role, pp_group, and pp_cluster Facts](#the-pp_role-pp_group-and-pp_cluster-facts) for details on these custom facts.

## Team Modules

Each team has a module named `puppet_<team>`. These are hosted on git.uoregon.edu and pulled into the control repo via the Puppetfile.

Your team module contains:

```
puppet_yourteam/
├── data/
│   ├── nodes/
│   │   └── webserver01.yaml
│   ├── clusters/
│   │   └── yourteam/
│   │       └── webcluster.yaml
│   └── yourteam_common.yaml
├── files/
│   └── ...                     # Static files to deploy
├── templates/
│   └── ...                     # ERB/EPP templates
└── manifests/
    ├── profile/
    │   ├── base.pp
    │   ├── nginx.pp
    │   ├── webserver.pp
    │   └── ...
    └── role/
        ├── webserver.pp
        └── ...
```

The `manifests/` directory contains only `profile/` and `role/` subdirectories. All your Puppet code goes in one of these two places. Use `files/` for static files and `templates/` for dynamic configuration templates.

### Module Organization

You have full flexibility within your team module. Use the autoloader naming conventions to organize code however makes sense:

```
puppet_yourteam/manifests/profile/
├── base.pp                    # puppet_yourteam::profile::base
├── nginx.pp                   # puppet_yourteam::profile::nginx
├── nginx/
│   └── ssl.pp                 # puppet_yourteam::profile::nginx::ssl
├── sshkeys/
│   ├── admins.pp              # puppet_yourteam::profile::sshkeys::admins
│   └── developers.pp          # puppet_yourteam::profile::sshkeys::developers
└── webserver.pp               # puppet_yourteam::profile::webserver (server profile)
```

## Writing Roles

A role defines what a server is. At UO, roles typically represent a cluster of servers that should be configured identically, or a single unique server.

Here's the anatomy of a role:

```puppet
class puppet_yourteam::role::webserver {
  # 1. System tags (for log aggregation, monitoring, etc.)
  system_tags::tag { 'ou': value => 'YourOU' }
  system_tags::tag { 'subou': value => 'YourSubOU' }
  system_tags::tag { 'service': value => get_role_name($title) }

  # 2. UO-wide base profile (always include this)
  include profile::base

  # 3. Your team's base profile
  include puppet_yourteam::profile::base

  # 4. Shared technical profiles (zero or more)
  include puppet_yourteam::profile::nginx
  include puppet_yourteam::profile::php

  # 5. Server profile (optional, same name as role)
  include puppet_yourteam::profile::webserver
}
```

### The Includes, In Order

1. **System tags**: Required for integration with log aggregation and other services. This is some cruft that needs to be moved elsewhere, but for now just copy the convention when you create new roles.

2. **profile::base**: The UO-wide base configuration. Always include this. It lives in the control repo at `site-modules/profile/manifests/base.pp`.

3. **Your team's base profile**: Team-specific configuration that applies to all your servers (SSH keys for your team members, sudo fragments, etc.).

4. **Shared technical profiles**: Reusable profiles for specific technologies. If multiple roles need nginx, create `puppet_yourteam::profile::nginx` and include it in each.

5. **Server profile**: A profile with the same name as the role, for configuration that isn't reusable. This is optional but common.

### Why Server Profiles?

Not everything fits neatly into a reusable profile. When a server needs one-off configuration (a specific cron job, a unique file, a package only this server needs), you might be tempted to put it directly in the role. But roles should only contain `include` statements.

Server profiles (which we sometimes lovingly call "dump profiles") solve this by giving you a place for server-specific code while maintaining the proper layering: **Server → Role → Profiles → Modules**. The role stays clean (just includes), and all the actual configuration lives in profiles where it belongs.

Naming the server profile the same as the role makes it easy to find. When you're looking at `puppet_yourteam::role::webserver`, you know the server-specific code is in `puppet_yourteam::profile::webserver`.

```puppet
# puppet_yourteam/manifests/profile/webserver.pp
class puppet_yourteam::profile::webserver {
  # Everything here is specific to this one server/cluster
  include puppet_yourteam::profile::sshkeys::webteam

  package { 'some-unique-package':
    ensure => installed,
  }

  cron { 'nightly-cleanup':
    command => '/usr/local/bin/cleanup.sh',
    hour    => 2,
    minute  => 30,
  }
}
```

### Documenting in Role Files

Since we're using infrastructure as code, keep documentation as close to the code as possible. Role files are a great place for human-readable documentation about a server or cluster:

```puppet
# puppet_yourteam/manifests/role/webserver.pp

# Webserver - Production web application servers
#
# These servers run the main customer-facing web application.
# They handle approximately 10k requests/hour during peak times.
#
# Contacts:
#   - Primary: Jane Doe (jdoe@uoregon.edu)
#   - Backup: John Smith (jsmith@uoregon.edu)
#
# Dependencies:
#   - Database: db01.example.com (PostgreSQL)
#   - Cache: redis01.example.com
#
# Notes:
#   - SSL certificate renews automatically via Let's Encrypt
#   - Maintenance window: Sundays 2-4am

class puppet_yourteam::role::webserver {
  # ... includes ...
}
```

When someone needs to understand what a server does or who to contact about it, the role file is the first place they'll look.

## Writing Profiles

Profiles in your team module fall into two categories:

1. **Server profiles** (discussed above): Server-specific code, named the same as the role
2. **Reusable profiles**: Shared configuration that multiple roles can include

When you find yourself copying the same configuration into multiple server profiles, extract it into a reusable profile:

```puppet
# puppet_yourteam/manifests/profile/nginx.pp
class puppet_yourteam::profile::nginx {
  include nginx

  # Team-specific config, monitoring, etc.
  file { '/etc/nginx/conf.d/default.conf':
    ensure  => file,
    source  => "puppet:///modules/puppet_yourteam/nginx/default.conf",
    require => Package['nginx'],
    notify  => Service['nginx'],
  }
}
```

Now any role that needs nginx can `include puppet_yourteam::profile::nginx` instead of duplicating the configuration.

Keep reusable profiles focused. If you're configuring both nginx and PostgreSQL, that's two profiles.

## Hiera Data

Hiera data for your team lives in your team module under `data/`. The hierarchy checks these locations in order:

1. `data/nodes/<hostname>.yaml` - Server-specific overrides
2. `data/clusters/<yourteam>/<cluster>.yaml` - Cluster-wide settings
3. `data/<yourteam>_common.yaml` - Defaults for all your servers

If Hiera doesn't find a value in your team's data, it falls back to the control repo's `common.yaml`.

### Example

For a server named `db01` in the `oracle` cluster managed by the `dba` team:

```
puppet_dba/data/
├── nodes/
│   └── db01.yaml              # Checked first
├── clusters/
│   └── dba/
│       └── oracle.yaml        # Checked second
└── dba_common.yaml            # Checked third
```

### Writing Hiera Data

```yaml
# data/dba_common.yaml
---
# Defaults for all DBA servers
profile::base::timezone: 'America/Los_Angeles'

# data/clusters/dba/oracle.yaml
---
# Settings for all Oracle cluster members
oracle::version: '19c'
oracle::memory_target: '4G'

# data/nodes/db01.yaml
---
# Overrides just for db01
oracle::memory_target: '8G'  # This server has more RAM
```

### Encrypted Data

For secrets like passwords and API keys, use eyaml to encrypt values in your Hiera data. See the [Hiera training module](../learn/puppet/04-hiera-and-data-separation.md#encrypted-data-with-eyaml) and [eyaml tool documentation](../tools/eyaml.md) for details.

## The pp_role, pp_group, and pp_cluster Facts

Three custom facts drive how Puppet configures each server:

| Fact | Purpose | Example |
|------|---------|---------|
| `pp_role` | Which role class to include | `puppet_dba::role::oracle` |
| `pp_group` | Which team module owns this server | `dba` |
| `pp_cluster` | Which cluster this server belongs to | `oracle` |

The `pp_role` fact is set when a server is provisioned. You can change it as root using the `set_pp_role` script:

```bash
set_pp_role puppet_yourteam::role::newrole
```

The `pp_group` fact is automatically derived from `pp_role` by extracting the team name (the part after `puppet_`). For example, if `pp_role` is `puppet_dba::role::loire`, then `pp_group` is `dba`. This is one of the many features provided by `profile::base` in the `puppet-control-repo`.

The `pp_cluster` fact is optional. Setting it enables cluster-level Hiera lookups, so all servers in a cluster can share configuration without duplicating it in node-level files. If your server belongs to a cluster, set it as root using `set_pp_cluster`:

```bash
set_pp_cluster mycluster
```

## Making Changes

### Changes to Your Team's Servers

Work in your team module. For trivial changes, most teams commit directly to master:

```bash
git clone ssh://git@git.uoregon.edu/pup/puppet_yourteam.git
cd puppet_yourteam
# Make changes
git add .
git commit -m "Add nginx profile"
git push
```

For larger or riskier changes, use a feature branch and PR:

```bash
git checkout -b feature_new_nginx_config
# Make changes
git add .
git commit -m "Add nginx profile"
git push -u origin feature_new_nginx_config
# Open PR on git.uoregon.edu
```

### Changes to UO-Wide Configuration

If you need to change something in the control repo, submit a PR to `puppet-control-repo`. These changes affect everyone, so they'll get extra review.

For concerns around `profile::base`, contact the platform team. This profile is critical infrastructure that affects every managed server. Most of `profile::base` is parameterized, so you can disable individual components via Hiera without modifying the profile itself. We encourage leaving the defaults unless you have a specific need to change them.

### Verifying Your Changes

After pushing changes to production, either run Puppet manually on an affected server or wait for the next scheduled run and check the results. You don't want to leave a server in a state where the catalog fails to compile.

```bash
# On the server, as root
puppet agent -t
```

If there's a problem, you'll see it immediately and can fix it while the change is fresh in your mind.

## Environments and Deployment

When you push changes to a team module, the Puppet server automatically updates the corresponding environment. This happens for:

- `master` branch
- `main` branch
- `feature_*` branches (any branch starting with `feature_`)

### Production Workflow

Most servers run in the `production` environment, which tracks the `master` (or `main`) branch. When your changes are pushed to master, the Puppet server picks up the changes within a few minutes. Servers will apply the new configuration on their next Puppet run.

### Testing with Feature Branches

For larger changes, use a feature branch to test before merging to production:

1. **Create a feature branch** with a name starting with `feature_`:
   ```bash
   git checkout -b feature_new_nginx_config
   git push -u origin feature_new_nginx_config
   ```

2. **Wait for the environment to be created.** The Puppet server will create an environment named `feature_new_nginx_config`.

3. **Assign a test server to the new environment.** On the test server as root, run:
   ```bash
   set_temp_environment feature_new_nginx_config
   ```

4. **Make and test your changes.** Run Puppet on the test server to pick up changes as you make commits and verify everything works.

5. **Merge to master.** Once you're confident, merge your feature branch into `master`.

6. **Switch servers back to production.** On the test server:
   ```bash
   set_temp_environment production
   ```

Feature environments are short-lived. Use them for testing, then clean up by merging or deleting the branch.

## When Things Go Wrong

Syntax errors and compilation failures can feel scary, but they're usually not catastrophic. Here's why:

**Compilation errors don't break running servers.** If Puppet can't compile a catalog (due to a syntax error, missing class, or type mismatch), the agent simply keeps the previous configuration. The server continues running as it was. Nothing changes.

**The failure is visible.** You'll see the error in the Puppet run output, which tells you exactly what went wrong and where.

**The fix is usually straightforward.** Correct the syntax error, push the fix, and run Puppet again.

The main risk isn't breaking a server. It's leaving a server in a state where Puppet can't make *any* changes because the catalog won't compile. If a real issue comes up later (security patch, configuration change), you won't be able to address it until you fix the compilation error. That's why you should verify your changes soon after pushing.

## Quick Reference

| Task | How |
|------|-----|
| Add a new server | Create a role in your team module |
| Configure a technology | Create or update a profile in your team module |
| Set Hiera data | `data/` directory in your team module |
| Change a server's role | `set_pp_role puppet_yourteam::role::newrole` (as root) |
| Set a server's cluster | `set_pp_cluster clustername` (as root) |
| Test with a feature branch | `set_temp_environment feature_branchname` (as root) |
| Run Puppet manually | `puppet agent -t` (as root) |
| View UO-wide base profile | `puppet-control-repo/site-modules/profile/manifests/base.pp` |
| View Hiera hierarchy | `puppet-control-repo/hiera.yaml` |
