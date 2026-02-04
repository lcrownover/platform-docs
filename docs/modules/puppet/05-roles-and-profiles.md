# Roles and Profiles

By the end of this section, you'll understand how to use the roles and profiles pattern to organize Puppet code into manageable, reusable layers.

## The Problem

You've learned about modules that manage individual technologies (nginx, mysql, ssh) and Hiera that provides configuration data. But how do you assemble these pieces into complete server configurations?

Consider a web server. It needs:

- nginx for serving traffic
- PHP for the application
- Monitoring agent for observability
- SSH configuration for access
- Firewall rules for security
- NTP for time sync

You could put all of this directly in a node definition:

```puppet
node 'web01.example.com' {
  include nginx
  include php
  include monitoring::agent
  include ssh
  include firewall
  include ntp
}
```

This works, but it doesn't scale. When you have 50 web servers, you're duplicating this list. When you need to add log shipping to all web servers, you're editing 50 node definitions. When a new person asks "what makes a web server a web server?", the answer is scattered across dozens of files.

**Roles and profiles** solve this by adding two layers between your modules and your nodes.

## The Pattern

The roles and profiles pattern has three layers:

```
Nodes → Roles → Profiles → Modules
```

- **Modules** manage individual technologies (nginx, mysql, ssh)
- **Profiles** wrap modules with your site-specific configuration
- **Roles** describe what a server does by including profiles
- **Nodes** are assigned exactly one role

Each layer has a clear purpose:

| Layer | Purpose | Example |
|-------|---------|---------|
| Module | Manages one technology | `nginx`, `mysql` |
| Profile | Configures one technology for your site | `profile::webserver`, `profile::database` |
| Role | Describes one type of server | `role::web_application`, `role::database_primary` |

## Profiles

A **profile** is a wrapper class that configures a technology for your environment. Profiles live in a `profile` module and include the technology modules with appropriate settings.

Here's a simple profile for a web server:

```puppet
# site-modules/profile/manifests/nginx.pp
class profile::nginx {
  include nginx

  firewall { '100 allow http':
    dport  => 80,
    proto  => 'tcp',
    action => 'accept',
  }

  firewall { '100 allow https':
    dport  => 443,
    proto  => 'tcp',
    action => 'accept',
  }
}
```

This profile does two things: includes the nginx module and opens the firewall. Every server that needs nginx gets both.

### Profiles and Hiera

Profiles typically use `include` rather than passing parameters directly. This lets Hiera provide the configuration:

```puppet
# site-modules/profile/manifests/nginx.pp
class profile::nginx {
  include nginx
}
```

```yaml
# data/roles/webserver.yaml
nginx::worker_processes: 8
nginx::worker_connections: 4096
```

The profile stays simple, and all the tuning happens in Hiera data. Different roles can have different Hiera values for the same profile.

### When Profiles Need Parameters

Sometimes a profile needs to make decisions based on what role it's in. Use Hiera lookups:

```puppet
class profile::nginx {
  $ssl_enabled = lookup('profile::nginx::ssl_enabled', Boolean, 'first', false)

  include nginx

  if $ssl_enabled {
    include nginx::ssl
  }
}
```

Or make the profile accept parameters (which Hiera can provide):

```puppet
class profile::nginx (
  Boolean $ssl_enabled = false,
) {
  include nginx

  if $ssl_enabled {
    include nginx::ssl
  }
}
```

### Profile Examples

**Base profile** (included by every server):

```puppet
class profile::base {
  include ntp
  include ssh
  include monitoring::agent
  include security::baseline
}
```

**Database profile:**

```puppet
class profile::mysql {
  include mysql::server
  include mysql::backup

  firewall { '100 allow mysql':
    dport  => 3306,
    proto  => 'tcp',
    action => 'accept',
  }
}
```

**Application profile:**

```puppet
class profile::myapp {
  include myapp

  file { '/var/log/myapp':
    ensure => directory,
    owner  => 'myapp',
    group  => 'myapp',
  }
}
```

## Roles

A **role** describes what a server is by including profiles. Roles live in a `role` module and should contain only `include` statements.

```puppet
# site-modules/role/manifests/webserver.pp
class role::webserver {
  include profile::base
  include profile::nginx
  include profile::php
}
```

```puppet
# site-modules/role/manifests/database.pp
class role::database {
  include profile::base
  include profile::mysql
}
```

```puppet
# site-modules/role/manifests/app_server.pp
class role::app_server {
  include profile::base
  include profile::nginx
  include profile::php
  include profile::myapp
}
```

### The One Role Rule

Each node gets exactly **one role**. This is a fundamental constraint of the pattern.

Why? Because a role answers the question "what is this server?" A server can't be two things at once. If you think you need two roles, you actually need a new role that combines those profiles.

```puppet
# Wrong: trying to assign two roles
node 'server01.example.com' {
  include role::webserver
  include role::database  # Don't do this
}

# Right: create a combined role
class role::web_and_database {
  include profile::base
  include profile::nginx
  include profile::mysql
}

node 'server01.example.com' {
  include role::web_and_database
}
```

### Roles Should Be Simple

Roles should contain only `include` statements for profiles. No resources, no logic, no parameters. All the work happens in profiles.

```puppet
# Good role
class role::webserver {
  include profile::base
  include profile::nginx
  include profile::php
}

# Bad role (too much logic)
class role::webserver {
  include profile::base
  include profile::nginx

  if $facts['os']['family'] == 'RedHat' {
    include profile::php
  }

  file { '/var/www':  # Resources don't belong in roles
    ensure => directory,
  }
}
```

If a role needs conditional logic, that logic should move into a profile.

## Assigning Roles to Nodes

The final step is connecting nodes to roles. There are several approaches:

### Node Definitions

The traditional approach uses node definitions in `manifests/site.pp`:

```puppet
node 'web01.example.com' {
  include role::webserver
}

node 'db01.example.com' {
  include role::database
}
```

### Hiera and a Role Fact

A more flexible approach uses a custom fact to identify each server's role, then looks up the role in Hiera or uses an External Node Classifier (ENC).

```puppet
# manifests/site.pp
node default {
  $server_role = $facts['role']
  include "role::${server_role}"
}
```

The `role` fact might come from a file on disk, a cloud tag, or a custom fact. This approach makes it easier to manage large fleets without maintaining long lists of node definitions.

## Putting It Together

Here's how the layers connect for a web server named `web01.example.com`:

1. **Node** `web01.example.com` is assigned `role::webserver`
2. **Role** `role::webserver` includes `profile::base`, `profile::nginx`, `profile::php`
3. **Profile** `profile::nginx` includes the `nginx` module and opens firewall ports
4. **Module** `nginx` manages the package, config, and service
5. **Hiera** provides parameters like `nginx::worker_processes` based on the role

Each layer has one job:

- The node says what role it is
- The role says what profiles it needs
- The profile says how to configure a technology
- The module does the actual work

## Directory Structure

A typical control repository with roles and profiles:

```
control-repo/
├── data/
│   ├── common.yaml
│   ├── roles/
│   │   ├── webserver.yaml
│   │   └── database.yaml
│   └── nodes/
│       └── web01.example.com.yaml
├── hiera.yaml
├── manifests/
│   └── site.pp
└── site-modules/
    ├── profile/
    │   └── manifests/
    │       ├── base.pp
    │       ├── nginx.pp
    │       ├── mysql.pp
    │       └── myapp.pp
    └── role/
        └── manifests/
            ├── webserver.pp
            ├── database.pp
            └── app_server.pp
```

## Exercises

1. **Map existing servers.** List three types of servers in your environment. For each, identify what profiles they would need (what technologies do they run?).

2. **Write a base profile.** Create a `profile::base` class that includes the common modules every server in your environment needs.

3. **Write a role.** Pick one server type from exercise 1 and write a role class that includes `profile::base` plus the technology-specific profiles it needs.

4. **Trace the layers.** Pick a server in your Puppet codebase. Follow the chain from node → role → profiles → modules. Draw a diagram showing what gets included.
