# Hiera and Data Separation

By the end of this section, you'll understand how Hiera separates configuration data from Puppet code, and how to use it to configure classes for different servers and environments.

## The Problem Hiera Solves

In the previous lesson, you saw two ways to pass parameters to a class:

```puppet
# Resource-like declaration with inline parameters
class { 'nginx':
  worker_processes => 4,
}

# Or just include, with parameters coming from... somewhere
include nginx
```

That "somewhere" is Hiera. But why does this matter?

Imagine you manage nginx on 50 servers. Some are production web servers needing 8 worker processes. Some are development boxes that need 2. Some have SSL, some don't. If you hardcode these values in your Puppet code, you end up with a mess of conditionals:

```puppet
# Don't do this
if $facts['hostname'] =~ /^prod-web/ {
  class { 'nginx':
    worker_processes => 8,
    ssl_enabled      => true,
  }
} elsif $facts['hostname'] =~ /^dev/ {
  class { 'nginx':
    worker_processes => 2,
    ssl_enabled      => false,
  }
}
# ... and so on for every variation
```

This mixes data (how many workers, SSL on or off) with logic (which servers get what). When requirements change, you're editing code instead of just updating configuration.

**Hiera** solves this by storing data separately from code. Your Puppet code stays generic:

```puppet
include nginx
```

And Hiera provides the right values based on facts about each server.

## How Hiera Works

Hiera is a key-value lookup system with a **hierarchy**. When Puppet needs a value, Hiera searches through a list of data sources in order, returning the first match it finds.

A typical hierarchy might look like:

1. Data specific to this exact server (`nodes/web01.example.com.yaml`)
2. Data for this server's role (`roles/webserver.yaml`)
3. Data for this operating system (`os/RedHat.yaml`)
4. Default data that applies to everything (`common.yaml`)

When a class needs a parameter value, Hiera walks down this hierarchy. If `nodes/web01.example.com.yaml` has the value, that's what gets used. If not, Hiera checks the next level, and so on.

## Automatic Parameter Lookup

Here's the key insight: when you `include` a class, Puppet automatically looks up each parameter in Hiera using the pattern `classname::parametername`.

Given this class:

```puppet
class nginx (
  Integer $worker_processes = 4,
  Boolean $ssl_enabled = false,
) {
  # ...
}
```

When you write `include nginx`, Puppet asks Hiera for:

- `nginx::worker_processes`
- `nginx::ssl_enabled`

If Hiera has values for these keys, they're used. If not, the defaults from the class definition apply.

This is why `include` is usually preferred over resource-like declaration. You write `include nginx` everywhere, and Hiera provides the right configuration for each server.

## The Hierarchy Configuration

The hierarchy is defined in `hiera.yaml`, typically at the root of your Puppet control repository:

```yaml
---
version: 5

defaults:
  datadir: data
  data_hash: yaml_data

hierarchy:
  - name: "Per-node data"
    path: "nodes/%{facts.networking.fqdn}.yaml"

  - name: "Per-role data"
    path: "roles/%{facts.role}.yaml"

  - name: "Per-OS data"
    path: "os/%{facts.os.family}.yaml"

  - name: "Common defaults"
    path: "common.yaml"
```

The `%{...}` syntax interpolates facts. For a server named `web01.example.com` running Red Hat with a `role` fact set to `webserver`, Hiera would check:

1. `data/nodes/web01.example.com.yaml`
2. `data/roles/webserver.yaml`
3. `data/os/RedHat.yaml`
4. `data/common.yaml`

Not all these files need to exist. Hiera skips missing files and continues down the hierarchy.

## Writing Hiera Data

Hiera data files are YAML. The keys are class parameter names in `classname::parametername` format:

```yaml
# data/common.yaml
---
nginx::worker_processes: 4
nginx::ssl_enabled: false

ssh::permit_root_login: 'no'
ssh::password_authentication: false
```

For a production web server, you might override some values:

```yaml
# data/roles/webserver.yaml
---
nginx::worker_processes: 8
nginx::ssl_enabled: true
nginx::ssl_certificate: '/etc/pki/tls/certs/server.crt'
nginx::ssl_key: '/etc/pki/tls/private/server.key'
```

And for a specific server that needs something unique:

```yaml
# data/nodes/web01.example.com.yaml
---
nginx::worker_processes: 16  # This server handles more traffic
```

## Data Types in YAML

YAML has its own syntax for different data types:

```yaml
# Strings
nginx::server_name: 'example.com'
nginx::server_name: "example.com"  # Quotes are optional for simple strings

# Integers
nginx::worker_processes: 8

# Booleans
nginx::ssl_enabled: true
nginx::ssl_enabled: false

# Arrays
nginx::allowed_methods:
  - GET
  - POST
  - HEAD

# Hashes
nginx::custom_headers:
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
```

!!! warning
    YAML interprets some values unexpectedly. The word `yes` becomes boolean `true`, `no` becomes `false`, and `~` becomes `null`. When in doubt, use quotes around strings to prevent type coercion: `'yes'`, `'no'`.

## Looking Up Data Explicitly

While automatic parameter lookup handles most cases, sometimes you need to look up data directly in your Puppet code:

```puppet
$message = lookup('motd::message')
```

You can provide a default value if the key doesn't exist:

```puppet
$message = lookup('motd::message', String, 'first', 'Welcome!')
```

The arguments are:
1. Key to look up
2. Expected data type
3. Merge behavior (usually `'first'` for simple values)
4. Default value

## Understanding the Data Flow

Let's trace through a complete example. You have:

**hiera.yaml:**
```yaml
hierarchy:
  - name: "Per-node data"
    path: "nodes/%{facts.networking.fqdn}.yaml"
  - name: "Common defaults"
    path: "common.yaml"
```

**data/common.yaml:**
```yaml
nginx::worker_processes: 4
```

**data/nodes/web01.example.com.yaml:**
```yaml
nginx::worker_processes: 16
```

**Your manifest:**
```puppet
include nginx
```

When Puppet runs on `web01.example.com`:

1. Puppet sees `include nginx` and needs values for `nginx` parameters
2. For `nginx::worker_processes`, Puppet asks Hiera
3. Hiera checks `data/nodes/web01.example.com.yaml` first, finds `16`
4. Hiera returns `16` (doesn't check common.yaml since it found a match)
5. The nginx class gets `worker_processes => 16`

When Puppet runs on `web02.example.com`:

1. Same process, but `data/nodes/web02.example.com.yaml` doesn't exist
2. Hiera skips to `common.yaml`, finds `4`
3. The nginx class gets `worker_processes => 4`

## Finding What Data a Class Needs

When you need to configure a class, how do you know what Hiera keys to set?

1. **Look at the class parameters:** Open the class definition and check the parameters in parentheses. Each parameter `$foo` becomes a Hiera key `classname::foo`.

2. **Check for required parameters:** Parameters without default values must be provided. If you see `String $ssl_certificate,` (no default), you need to set `classname::ssl_certificate` in Hiera.

3. **Read module documentation:** Well-maintained modules document their parameters.

## Organizing Your Data

A common pattern organizes data by what varies:

```
data/
├── common.yaml           # Defaults for everything
├── os/
│   ├── RedHat.yaml       # Red Hat-specific settings
│   └── Debian.yaml       # Debian-specific settings
├── roles/
│   ├── webserver.yaml    # Web server configuration
│   ├── database.yaml     # Database server configuration
│   └── monitoring.yaml   # Monitoring server configuration
└── nodes/
    ├── web01.example.com.yaml   # Server-specific overrides
    └── db01.example.com.yaml
```

Put widely-shared defaults in `common.yaml`. Put role-specific configuration in role files. Use node files sparingly, only for true one-off settings.

!!! tip
    If you find yourself creating lots of node-specific files, you might need a new hierarchy level. For example, if production and development differ significantly, add a `datacenter` or `environment` level rather than duplicating data across node files.

## Encrypted Data with eyaml

Hiera data often includes secrets: database passwords, API keys, SSL private keys. You don't want these stored as plain text in version control. **[eyaml](../../tools/eyaml.md)** (encrypted YAML) solves this by encrypting sensitive values while keeping the rest of your data readable.

### How eyaml Works

eyaml uses asymmetric encryption (public/private key pair):

- The **public key** encrypts data. Anyone with the public key can add new secrets.
- The **private key** decrypts data. Only the Puppet server has this key.

This means you can safely commit encrypted secrets to Git. Anyone can add or update secrets using the public key, but only the Puppet server can decrypt them when compiling catalogs.

In an established codebase, the public key is typically stored in the repository or shared by your team. Ask your team where to find it if you're not sure.

### Encrypted Values in Hiera

Encrypted values go in your YAML files just like regular values:

```yaml
# data/common.yaml
---
myapp::database_password: ENC[PKCS7,MIIBiQYJKoZIhvcNAQcDoIIBejCCAXYCAQ...]

myapp::ssl_key: >
  ENC[PKCS7,MIIBmQYJKoZIhvcNAQcDoIIBijCCAYYCAQAxggEhMIIBHQIBADAF
  MAACAQEwDQYJKoZIhvcNAQEBBQAEggEAh8aP3M6Oq9b5vTtB7Q4n4R8K...]
```

The Puppet server decrypts these automatically when compiling catalogs. Your Puppet code doesn't need any special handling. A class parameter that receives an encrypted value just sees the decrypted string.

### What Should Be Encrypted?

Encrypt any value that would be a problem if exposed:

- Passwords and passphrases
- API keys and tokens
- Private keys and certificates
- Database connection strings with credentials

Don't encrypt values that aren't sensitive. Encrypting everything makes the data harder to read and debug without adding security benefit.

See the [eyaml tool documentation](../../tools/eyaml.md) for installation and usage instructions.

## Exercises

1. **Trace a lookup.** Pick a class in your Puppet codebase. Find its parameters, then search your Hiera data for those keys. Which hierarchy level provides each value?

2. **Add a parameter override.** Find a class that's using default values. Add a Hiera entry to override one parameter for a specific role or node.

3. **Read a hiera.yaml.** Look at your organization's `hiera.yaml`. Draw out the hierarchy levels and identify what facts they use for interpolation.

4. **Find a required parameter.** Find a class with a required parameter (no default). Trace where that value comes from in your Hiera data.
