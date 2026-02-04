# Classes and Modules

By the end of this section, you'll understand how Puppet code is organized into classes and modules, how to use them, and how to navigate existing module code.

## Why Classes?

In the previous lesson, you wrote manifests as flat lists of resources. That works for learning, but real infrastructure has structure. You might manage dozens of servers, each needing different combinations of configurations: web servers, database servers, monitoring agents, security baselines.

**Classes** let you group related resources into named, reusable units. Instead of copying the same nginx resources into every manifest, someone defines a class once and you include it wherever needed.

## What a Class Looks Like

A class definition wraps resources in a named container:

```puppet
class nginx {
  package { 'nginx':
    ensure => installed,
  }

  file { '/etc/nginx/nginx.conf':
    ensure  => file,
    source  => 'puppet:///modules/nginx/nginx.conf',
    require => Package['nginx'],
    notify  => Service['nginx'],
  }

  service { 'nginx':
    ensure  => running,
    enable  => true,
    require => Package['nginx'],
  }
}
```

This defines a class named `nginx` containing three resources. Defining a class is like writing a function. The resources inside won't be applied until you **use** the class.

## Using Classes

There are two ways to use a class: `include` and resource-like declaration.

### include

The simplest way to use a class:

```puppet
include nginx
```

You can include the same class multiple times safely. Puppet only applies it once. This makes `include` ideal when multiple parts of your code might need the same class.

```puppet
include nginx
include nginx  # No error, no duplicate resources
```

### Resource-like Declaration

You can also declare a class like a resource:

```puppet
class { 'nginx':
}
```

This looks similar but behaves differently. You can only declare a class this way **once**. If two parts of your code both try `class { 'nginx': }`, Puppet raises an error.

Why would you use this? Resource-like declarations let you pass parameters.

## Class Parameters

Most classes accept parameters that customize their behavior:

```puppet
class { 'nginx':
  worker_processes   => 4,
  worker_connections => 2048,
}
```

Parameters let you configure a class without modifying its code. The class defines what parameters it accepts, their types, and default values.

When you use `include`, parameters come from [Hiera](04-hiera-and-data-separation.md) rather than being specified inline. This is usually the preferred approach because it separates code from data.

### Finding Available Parameters

To know what parameters a class accepts, you need to look at its definition. Parameters are declared in parentheses after the class name:

```puppet
class nginx (
  String  $worker_processes   = 'auto',
  Integer $worker_connections = 1024,
  Boolean $ssl_enabled        = false,
) {
  # ...
}
```

This class accepts three parameters. `worker_processes` defaults to `'auto'`, `worker_connections` defaults to `1024`, and `ssl_enabled` defaults to `false`. If a parameter has no default, you must provide a value.

### Common Parameter Types

| Type | What It Accepts |
|------|-----------------|
| `String` | Any string |
| `Integer` | Whole numbers |
| `Boolean` | `true` or `false` |
| `Array[String]` | Array where each element is a string |
| `Hash` | Key-value pairs |
| `Optional[Type]` | Either the specified type or `undef` |
| `Enum['a', 'b']` | Only the listed values |

If you pass a value of the wrong type, Puppet fails at compile time with a clear error message.

## What Is a Module?

A **module** is a directory containing Puppet code, files, templates, and metadata organized in a standard structure. Modules are how Puppet code is packaged and distributed.

When you write `include nginx`, Puppet needs to find that class somewhere. It looks in its **module path** for a directory named `nginx`, then loads the class from a predictable location within it.

## Module Structure

Understanding module structure helps you navigate existing code and know where to find things.

```
nginx/
├── manifests/
│   ├── init.pp          # Contains class nginx
│   ├── config.pp        # Contains class nginx::config
│   └── service.pp       # Contains class nginx::service
├── files/
│   └── mime.types       # Static files
├── templates/
│   └── nginx.conf.epp   # Templates
└── metadata.json        # Module metadata
```

### manifests/

All Puppet code lives here. The naming convention maps class names to file paths:

| Class Name | File Location |
|------------|---------------|
| `nginx` | `nginx/manifests/init.pp` |
| `nginx::config` | `nginx/manifests/config.pp` |
| `nginx::vhost::ssl` | `nginx/manifests/vhost/ssl.pp` |

The main class (named after the module) always goes in `init.pp`. Subclasses use the double-colon `::` separator, which maps to directory structure.

This naming convention is how Puppet's **autoloader** finds classes. If you're looking for a class, you can predict exactly where its code lives.

### files/

Static files that get copied to managed nodes. When you see a `source` attribute like this:

```puppet
file { '/etc/nginx/mime.types':
  source => 'puppet:///modules/nginx/mime.types',
}
```

The URL `puppet:///modules/nginx/mime.types` maps to `nginx/files/mime.types` in the module.

### templates/

Templates for generating dynamic content. When you see:

```puppet
content => epp('nginx/nginx.conf.epp', { ... }),
```

The path `nginx/nginx.conf.epp` maps to `nginx/templates/nginx.conf.epp`.

## Reading Module Code

When you need to understand what a module does or what parameters it accepts, here's where to look:

1. **Start with `manifests/init.pp`:** This contains the main class and often includes or contains the subclasses.
2. **Check the parameters:** Look at the parentheses after `class modulename (` to see what you can configure.
3. **Read the templates:** If you need to understand what a config file will look like, templates show the structure.
4. **Check `metadata.json`:** This shows dependencies and supported operating systems.

## Subclasses

Large modules split their code into subclasses for organization. You might see classes like `nginx::install`, `nginx::config`, and `nginx::service` in a module's manifests directory.

You don't need to worry about how these are wired together. When you `include nginx`, the main class coordinates its subclasses automatically. You generally don't include subclasses directly unless the module documentation says to.

## Exercises

1. **Navigate a module.** Pick a module in your Puppet codebase. Find its main class, identify its parameters, and trace how it uses subclasses (if any).

2. **Find a template.** In the same module, find a file resource that uses a template. Locate the template file and read it to understand what the generated config will look like.

3. **Trace a file source.** Find a file resource that uses `puppet:///modules/...` as its source. Locate the actual file in the module's `files/` directory.

4. **Use a class with parameters.** Write a small manifest that includes a class with custom parameter values using resource-like declaration. Then rewrite it using `include` (parameters will need to come from Hiera, which you'll learn next).
