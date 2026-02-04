# Writing Modules

By the end of this section, you'll know how to create Puppet modules from scratch, including proper structure, class organization, and best practices.

## When to Write a Module

Before creating a new module, consider whether you actually need one. You might write a module when:

- You're managing a service or application that doesn't have an existing internal module
- The existing modules don't fit your environment's needs
- You want full control over the implementation

For many tasks, you'll use existing modules provided by your team and configure them through roles and profiles. But when you do need to create a module, here's how.

## Creating the Structure

A module needs at minimum a `manifests/` directory with an `init.pp` file. Let's create a module that manages the message of the day (motd).

```bash
mkdir -p motd/manifests motd/templates
```

The main class goes in `manifests/init.pp` and must be named after the module:

```puppet
# motd/manifests/init.pp
class motd (
  String $message = 'Welcome!',
) {
  file { '/etc/motd':
    ensure  => file,
    owner   => 'root',
    group   => 'root',
    mode    => '0644',
    content => epp('motd/motd.epp', {
      'message'  => $message,
      'hostname' => $facts['networking']['hostname'],
      'os'       => $facts['os']['name'],
    }),
  }
}
```

The template goes in `templates/`:

```epp
<%- | String $message, String $hostname, String $os | -%>
# Managed by Puppet - do not edit

<%= $message %>

Hostname: <%= $hostname %>
OS:       <%= $os %>
```

Now the module can be used:

```puppet
include motd

# Or with a custom message:
class { 'motd':
  message => 'Welcome to the production environment.',
}
```

## Splitting Classes

As modules grow, splitting code into multiple classes improves organization and maintainability. A common pattern separates installation, configuration, and service management:

```puppet
# nginx/manifests/init.pp
class nginx (
  String $worker_processes = 'auto',
) {
  contain nginx::install
  contain nginx::config
  contain nginx::service

  Class['nginx::install']
  -> Class['nginx::config']
  ~> Class['nginx::service']
}
```

```puppet
# nginx/manifests/install.pp
class nginx::install {
  package { 'nginx':
    ensure => installed,
  }
}
```

```puppet
# nginx/manifests/config.pp
class nginx::config {
  file { '/etc/nginx/nginx.conf':
    ensure  => file,
    content => epp('nginx/nginx.conf.epp'),
  }
}
```

```puppet
# nginx/manifests/service.pp
class nginx::service {
  service { 'nginx':
    ensure => running,
    enable => true,
  }
}
```

### Why contain?

The `contain` function is similar to `include` but establishes a **containment relationship**. This matters when other code depends on your class.

Without containment, if another class does `require => Class['nginx']`, Puppet only waits for the `nginx` class itself, not its subclasses. The subclasses might run later, causing ordering problems.

With `contain`, dependencies on `nginx` automatically apply to all contained classes. When something requires `Class['nginx']`, it waits for `nginx::install`, `nginx::config`, and `nginx::service` to all complete.

Use `contain` when your main class coordinates subclasses. Use `include` when you just need a class applied somewhere without strict ordering requirements.

## Passing Data to Subclasses

Parameters in the main class aren't automatically available in subclasses. You have several options:

### Option 1: Access via Scope

Subclasses can read parameters from the parent class using the fully qualified variable name:

```puppet
class nginx::config {
  $workers = $nginx::worker_processes

  file { '/etc/nginx/nginx.conf':
    content => epp('nginx/nginx.conf.epp', {
      'worker_processes' => $workers,
    }),
  }
}
```

This works but creates tight coupling between classes.

### Option 2: Use Hiera

Each subclass can look up its own parameters from Hiera. This is often the cleanest approach:

```puppet
class nginx::config (
  String $worker_processes = 'auto',
) {
  file { '/etc/nginx/nginx.conf':
    content => epp('nginx/nginx.conf.epp', {
      'worker_processes' => $worker_processes,
    }),
  }
}
```

With automatic parameter lookup, Hiera provides the value for `nginx::config::worker_processes`.

### Option 3: Pass Explicitly

The main class can pass parameters when containing subclasses:

```puppet
class nginx (
  String $worker_processes = 'auto',
) {
  class { 'nginx::config':
    worker_processes => $worker_processes,
  }
  contain nginx::config
}
```

This is explicit but verbose, and using `class { }` means the subclass can only be declared once.

## Module Metadata

The `metadata.json` file describes your module:

```json
{
  "name": "myorg-nginx",
  "version": "1.0.0",
  "author": "Your Team",
  "summary": "Manages nginx web server",
  "license": "proprietary",
  "dependencies": [
    { "name": "puppetlabs-stdlib", "version_requirement": ">= 4.0.0" }
  ],
  "operatingsystem_support": [
    {
      "operatingsystem": "RedHat",
      "operatingsystemrelease": ["7", "8", "9"]
    }
  ]
}
```

Key fields:

| Field | Purpose |
|-------|---------|
| `name` | Module name in `organization-module` format |
| `version` | Semantic version number |
| `dependencies` | Other modules this one requires |
| `operatingsystem_support` | Which OSes this module supports |

## File Organization Patterns

### Simple Module

For modules managing a single simple thing:

```
motd/
├── manifests/
│   └── init.pp
└── templates/
    └── motd.epp
```

### Service Module

For modules managing a service with the package-file-service pattern:

```
nginx/
├── manifests/
│   ├── init.pp        # Main class, contains subclasses
│   ├── install.pp     # Package installation
│   ├── config.pp      # Configuration files
│   └── service.pp     # Service management
├── files/
│   └── mime.types     # Static files
├── templates/
│   └── nginx.conf.epp
└── metadata.json
```

### Complex Module

For modules with multiple related components:

```
apache/
├── manifests/
│   ├── init.pp
│   ├── install.pp
│   ├── config.pp
│   ├── service.pp
│   ├── mod/
│   │   ├── ssl.pp       # apache::mod::ssl
│   │   ├── php.pp       # apache::mod::php
│   │   └── rewrite.pp   # apache::mod::rewrite
│   └── vhost.pp         # Defined type for vhosts
├── files/
├── templates/
└── metadata.json
```

## Testing Your Module

Before deploying a new module, test it locally:

```bash
# Validate syntax
puppet parser validate manifests/*.pp

# Check style
puppet-lint manifests/

# Apply locally in noop mode
puppet apply --noop -e 'include mymodule'

# Apply for real (on a test system)
puppet apply -e 'include mymodule'
```

For more comprehensive testing with rspec-puppet and other tools, see [Testing and Development](../puppet/06-testing-and-development.md).

## Exercises

1. **Create a complete module.** Build a module that manages a simple service (perhaps rsyslog or chronyd). Include install, config, and service classes with proper containment and ordering.

2. **Add parameters.** Extend your module with at least two parameters that affect the configuration. Use appropriate data types.

3. **Write metadata.** Create a proper `metadata.json` for your module with dependencies and OS support.

4. **Test the split.** Intentionally break the containment (use `include` instead of `contain`) and observe what happens when another class tries to `require` your module.
