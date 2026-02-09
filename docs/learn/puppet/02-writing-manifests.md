# Writing Manifests

By the end of this section, you'll know how to write Puppet manifests from scratch, including variables, conditionals, loops, and templates.

## What Is a Manifest?

A **manifest** is a file containing Puppet code. Manifests have the `.pp` extension. When you write Puppet code, you're writing manifests.

At its simplest, a manifest is just a collection of resource declarations:

```puppet
package { 'vim':
  ensure => installed,
}

file { '/etc/motd':
  ensure  => file,
  content => "Welcome to this server.\n",
}
```

Save this as `site.pp` and you have a working manifest. But real-world manifests need more: variables to avoid repetition, conditionals to handle different systems, loops to manage multiple similar resources, and templates to generate complex configuration files.

## Duplicate Resources

Every resource in a Puppet catalog must be unique. Puppet identifies resources by their **type** and **title** combined. You cannot declare two resources with the same type and title:

```puppet
# This will fail to compile
package { 'nginx':
  ensure => installed,
}

package { 'nginx':  # Error: duplicate resource
  ensure => latest,
}
```

This applies across your entire codebase, not just within a single file. If two different modules both try to manage `package { 'nginx': }`, Puppet raises a duplicate resource error during compilation.

This is one reason why organizing code into well-designed modules matters. When multiple parts of your infrastructure need the same package, they should all use the same class rather than each declaring the resource independently.

!!! note
    The title doesn't have to match the resource's name. You can use the `name` attribute to manage the same underlying thing with different titles, though this is rarely a good idea and usually indicates a design problem.

## Variables

Variables in Puppet start with a `$` and are assigned with `=`:

```puppet
$package_name = 'nginx'
$config_dir   = '/etc/nginx'

package { $package_name:
  ensure => installed,
}

file { $config_dir:
  ensure => directory,
}
```

Once assigned, variables cannot be reassigned. Puppet is not a scripting language where you modify state as you go. You declare what things should be, and variables are part of that declaration.

### Data Types

Puppet supports several data types:

| Type | Example |
|------|---------|
| String | `'hello'` or `"hello"` |
| Integer | `42` |
| Float | `3.14` |
| Boolean | `true`, `false` |
| Array | `['one', 'two', 'three']` |
| Hash | `{ 'key' => 'value', 'another' => 'thing' }` |
| Undef | `undef` |

**Strings** can use single or double quotes. Double-quoted strings allow variable interpolation:

```puppet
$username = 'deploy'
$home_dir = "/home/${username}"  # Evaluates to /home/deploy
```

Use single quotes when you don't need interpolation. It's clearer and avoids accidental interpretation of special characters.

**Arrays** are useful when you need to manage multiple similar things:

```puppet
$packages = ['vim', 'git', 'curl', 'wget']

package { $packages:
  ensure => installed,
}
```

This creates four package resources, one for each element. Puppet expands the array automatically.

**Hashes** map keys to values and are useful for structured data:

```puppet
$users = {
  'alice' => { 'uid' => 1001, 'shell' => '/bin/bash' },
  'bob'   => { 'uid' => 1002, 'shell' => '/bin/zsh' },
}
```

You'll see how to iterate over hashes in the [Iteration](#iteration) section.

## Conditionals

Real infrastructure isn't uniform. You need to handle different operating systems, environments, and configurations.

### if/elsif/else

The most common conditional:

```puppet
if $facts['os']['family'] == 'RedHat' {
  $apache = 'httpd'
} elsif $facts['os']['family'] == 'Debian' {
  $apache = 'apache2'
} else {
  fail("Unsupported OS family: ${facts['os']['family']}")
}

package { $apache:
  ensure => installed,
}
```

The `fail()` function stops catalog compilation with an error message. Use it when Puppet encounters a situation it can't handle.

### Comparison Operators

| Operator | Meaning |
|----------|---------|
| `==` | Equal |
| `!=` | Not equal |
| `<`, `>`, `<=`, `>=` | Numeric comparison |
| `=~` | Regex match |
| `!~` | Regex doesn't match |
| `in` | Membership (string contains, array includes) |

Examples:

```puppet
if $facts['os']['release']['major'] >= '8' {
  # RHEL 8 or newer
}

if $facts['networking']['fqdn'] =~ /^web\d+\./ {
  # Hostname starts with web followed by digits
}

if 'web' in $facts['networking']['hostname'] {
  # Hostname contains 'web'
}
```

### case Statements

When you have multiple conditions to check against a single value, `case` is cleaner than chained `if` statements:

```puppet
case $facts['os']['family'] {
  'RedHat': {
    $apache_package = 'httpd'
    $apache_service = 'httpd'
    $apache_config  = '/etc/httpd/conf/httpd.conf'
  }
  'Debian': {
    $apache_package = 'apache2'
    $apache_service = 'apache2'
    $apache_config  = '/etc/apache2/apache2.conf'
  }
  default: {
    fail("Unsupported OS: ${facts['os']['family']}")
  }
}
```

Case values can be strings, regular expressions, or arrays of either:

```puppet
case $facts['os']['name'] {
  'Ubuntu', 'Debian': {
    # Debian-based
  }
  /^(RedHat|CentOS|Rocky|Alma)/: {
    # RHEL-family
  }
  default: {
    # Everything else
  }
}
```

### Selector Expressions

For simple value assignment based on conditions, selectors are more concise:

```puppet
$apache_package = $facts['os']['family'] ? {
  'RedHat' => 'httpd',
  'Debian' => 'apache2',
  default  => fail("Unsupported OS"),
}
```

The selector returns the value on the right side of `=>` when the condition on the left matches. Think of it like a ternary operator that handles multiple cases.

## Iteration

When you need to create multiple similar resources from structured data, Puppet provides iteration functions.

### each

The `each` function iterates over arrays and hashes:

```puppet
$users = ['alice', 'bob', 'carol']

$users.each |String $username| {
  user { $username:
    ensure     => present,
    managehome => true,
  }
}
```

For hashes, you receive both key and value:

```puppet
$packages = {
  'vim'  => 'latest',
  'git'  => 'installed',
  'curl' => '7.68.0',
}

$packages.each |String $name, String $version| {
  package { $name:
    ensure => $version,
  }
}
```

### Practical Example: Managing Multiple Config Files

Iteration shines when you need to manage multiple similar things:

```puppet
$vhosts = {
  'example.com'     => { 'docroot' => '/var/www/example' },
  'blog.example.com' => { 'docroot' => '/var/www/blog' },
  'api.example.com'  => { 'docroot' => '/var/www/api' },
}

$vhosts.each |String $domain, Hash $config| {
  file { "/etc/httpd/conf.d/${domain}.conf":
    ensure  => file,
    content => template('mymodule/vhost.conf.erb'),
  }

  file { $config['docroot']:
    ensure => directory,
    owner  => 'apache',
    group  => 'apache',
  }
}
```

## Templates

Hardcoding configuration content in manifests works for simple cases, but real configuration files need dynamic content. Templates let you generate file contents from variables and logic.

!!! tip
    Always include a header comment in Puppet-managed files indicating they're managed by Puppet. Something like `# Managed by Puppet - do not edit` helps anyone troubleshooting on the system understand at a glance that manual changes will be reverted on the next Puppet run.

Puppet supports two template formats:

- **ERB** (Embedded Ruby): The original format, uses `<%= %>` syntax
- **EPP** (Embedded Puppet): Newer format, uses Puppet syntax instead of Ruby

Both work well. ERB is more common in existing code. EPP is easier if you're not familiar with Ruby.

### ERB Templates

ERB templates mix literal text with embedded Ruby code:

```erb
# Managed by Puppet - do not edit
# nginx.conf

worker_processes <%= @worker_count %>;

events {
    worker_connections <%= @worker_connections %>;
}

http {
    <% @upstreams.each do |name, servers| -%>
    upstream <%= name %> {
        <% servers.each do |server| -%>
        server <%= server %>;
        <% end -%>
    }
    <% end -%>
}
```

Key ERB syntax:

| Syntax | Purpose |
|--------|---------|
| `<%= expr %>` | Output the result of the expression |
| `<% code %>` | Execute code without output |
| `<%- code %>` | Execute code, suppress leading whitespace |
| `<% code -%>` | Execute code, suppress trailing newline |

In ERB templates, variables from your manifest are available with `@` prefix: `$worker_count` in Puppet becomes `@worker_count` in ERB.

To use an ERB template in your manifest:

```puppet
$worker_count       = $facts['processors']['count']
$worker_connections = 1024
$upstreams = {
  'backend' => ['10.0.1.10:8080', '10.0.1.11:8080'],
}

file { '/etc/nginx/nginx.conf':
  ensure  => file,
  content => template('mymodule/nginx.conf.erb'),
}
```

The `template()` function looks for templates in your module's `templates/` directory. We'll cover module structure in the next lesson.

### EPP Templates

EPP uses Puppet syntax instead of Ruby:

```epp
<%- | Integer $worker_count, Integer $worker_connections, Hash $upstreams | -%>
# Managed by Puppet - do not edit
# nginx.conf

worker_processes <%= $worker_count %>;

events {
    worker_connections <%= $worker_connections %>;
}

http {
    <% $upstreams.each |$name, $servers| { -%>
    upstream <%= $name %> {
        <% $servers.each |$server| { -%>
        server <%= $server %>;
        <% } -%>
    }
    <% } -%>
}
```

The first line declares parameters the template expects. This makes dependencies explicit and provides type checking.

To use an EPP template:

```puppet
file { '/etc/nginx/nginx.conf':
  ensure  => file,
  content => epp('mymodule/nginx.conf.epp', {
    'worker_count'       => $facts['processors']['count'],
    'worker_connections' => 1024,
    'upstreams'          => { 'backend' => ['10.0.1.10:8080', '10.0.1.11:8080'] },
  }),
}
```

### inline_template and inline_epp

For simple cases where creating a separate file feels like overkill, you can embed templates directly:

```puppet
$message = "Server ${facts['networking']['fqdn']} is managed by Puppet."

file { '/etc/motd':
  ensure  => file,
  content => inline_epp('<%= $message %>', { 'message' => $message }),
}
```

Use this sparingly. Once a template is more than a line or two, put it in a file.

## Putting It Together: A Complete Example

Let's write a manifest that configures a web server. This example demonstrates variables, conditionals, templates, and the package-file-service pattern:

```puppet
# Variables
$document_root = '/var/www/html'
$server_name   = $facts['networking']['fqdn']

# OS-specific values
case $facts['os']['family'] {
  'RedHat': {
    $package_name = 'httpd'
    $service_name = 'httpd'
    $config_file  = '/etc/httpd/conf/httpd.conf'
    $config_dir   = '/etc/httpd/conf.d'
  }
  'Debian': {
    $package_name = 'apache2'
    $service_name = 'apache2'
    $config_file  = '/etc/apache2/apache2.conf'
    $config_dir   = '/etc/apache2/sites-enabled'
  }
  default: {
    fail("Unsupported OS family: ${facts['os']['family']}")
  }
}

# Install the package
package { $package_name:
  ensure => installed,
}

# Main config file
file { $config_file:
  ensure  => file,
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  content => template('webserver/httpd.conf.erb'),
  require => Package[$package_name],
  notify  => Service[$service_name],
}

# Document root directory
file { $document_root:
  ensure => directory,
  owner  => 'root',
  group  => 'root',
  mode   => '0755',
}

# Default index page
file { "${document_root}/index.html":
  ensure  => file,
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  content => "<html><body><h1>Welcome to ${server_name}</h1></body></html>\n",
  require => File[$document_root],
}

# Ensure the service is running
service { $service_name:
  ensure  => running,
  enable  => true,
  require => Package[$package_name],
}
```

This manifest:

1. Sets up variables for the document root and server name
2. Uses a case statement to handle Red Hat vs. Debian differences
3. Installs the appropriate package
4. Manages the main config file using a template
5. Creates the document root and a default index page
6. Ensures the service runs and restarts when config changes

## Validating Your Code

Before applying manifests, validate them:

```bash
puppet parser validate manifest.pp
```

This checks for syntax errors. For more thorough checking, use `puppet-lint`:

```bash
puppet-lint manifest.pp
```

Puppet-lint checks for style issues and common mistakes. It's not installed by default but is worth adding to your workflow.

## Alternative Syntax

As you read existing Puppet code, you'll encounter some syntax variations that aren't always covered in tutorials. This section covers patterns you're likely to see.

!!! tip
    Just because a pattern exists doesn't mean you should use it. Simpler is almost always better. Stick to basic resource declarations with `require` and `notify` until you have a clear reason to do otherwise. These patterns are documented so you can recognize them, not because they're recommended.

### Multiple Resources in One Block

When declaring several resources of the same type, you can combine them into a single block using semicolons:

```puppet
file {
  '/etc/app':
    ensure => directory,
    owner  => 'root',
    mode   => '0755';
  '/etc/app/config.yml':
    ensure => file,
    owner  => 'root',
    mode   => '0644';
  '/etc/app/secrets.yml':
    ensure => file,
    owner  => 'root',
    mode   => '0600';
}
```

Each resource title is followed by a colon, its attributes, and a semicolon (except the last one, where the semicolon is optional). This is equivalent to writing three separate `file { }` blocks but keeps related resources visually grouped.

### Resource Defaults

When you're declaring many similar resources, you can set defaults to avoid repetition:

```puppet
File {
  owner => 'root',
  group => 'root',
  mode  => '0644',
}

file { '/etc/app/config.yml':
  ensure  => file,
  content => template('mymodule/config.yml.erb'),
}

file { '/etc/app/secrets.yml':
  ensure => file,
  mode   => '0600',  # Override the default
}
```

Notice the capitalized `File` for the defaults block. Defaults apply to all resources of that type in the current scope. Individual resources can override any default.

Resource defaults can make code harder to follow because the actual values aren't visible at the resource declaration. If you use them, put them at the top of a manifest where they're easy to spot.

### Chaining Arrows

Instead of using `require` and `notify` inside resource declarations, you can express relationships with chaining arrows:

```puppet
package { 'nginx':
  ensure => installed,
}
->
file { '/etc/nginx/nginx.conf':
  ensure => file,
  source => 'puppet:///modules/nginx/nginx.conf',
}
~>
service { 'nginx':
  ensure => running,
}
```

The `->` (ordering arrow) means "apply the left resource before the right." The `~>` (notification arrow) means "apply left before right, and refresh the right resource if the left one changes."

You can also chain multiple resources on one line:

```puppet
Package['nginx'] -> File['/etc/nginx/nginx.conf'] ~> Service['nginx']
```

Both styles (metaparameters like `require`/`notify` vs. chaining arrows) work. Teams often have a preference; use whatever your codebase already uses.

### The Spaceship Operator

For more complex ordering, the "spaceship" operator `<| |>` collects resources by attribute. You might see it used to order all resources of a type:

```puppet
Package <| |> -> File <| |> -> Service <| |>
```

This says "apply all packages, then all files, then all services." It's a blunt instrument but occasionally useful.

## Exercises

1. **Write a user manifest.** Create a manifest that manages three user accounts. Each user should have a home directory, a specific shell, and belong to a "developers" group. Use an array or hash to define the users and iterate over them.

2. **Add OS detection.** Modify your user manifest to set different default shells based on the OS family (e.g., `/bin/bash` for Red Hat, `/usr/bin/bash` for some Debian systems where the path differs).

3. **Create a template.** Write an ERB or EPP template for an `/etc/issue` file that displays the hostname, IP address, and OS version. Use the manifest to deploy it.

4. **Build a complete service.** Pick a simple service (rsyslog, crond, or similar) and write a manifest that manages its package, config file, and service. Include at least one configuration option that varies by OS.
