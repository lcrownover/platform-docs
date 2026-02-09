# Puppet 101 Lab

## Prerequisites

| Tool | Why You Need It |
|------|-----------------|
| Terminal with SSH | You'll connect to a remote lab server over SSH. macOS/Linux: use the built-in terminal. Windows: use [WSL](../tools/wsl.md) or [Windows Terminal](../tools/windows-terminal.md). |
| [Command-line editor](../tools/cli-editors.md) | You'll edit Puppet manifests and config files directly on the server. [Nano](../tools/cli-editors.md#nano) is the easiest option. |

## Setup

- You've been assigned a number (1-40). Throughout this lab, replace `XX` with your zero-padded number (e.g., 5 → `05`)
- SSH into your server:

    ```bash
    ssh participantXX@is-puppetlabXX.uoregon.edu
    ```

    Password: `UOISLabNumber@XX`

## Part 1: Installing a Package

!!! note "Talk about: What is Puppet and why do we use it?"
    - **Configuration management** -- managing server config as code instead of manual changes
    - **Desired state** -- you describe what the system should look like, not the steps to get there
    - **Idempotency** -- run the same manifest repeatedly, Puppet only changes what doesn't match
    - Benefits over manual work -- repeatable, version-controllable, self-documenting

1. **Create a working directory** for your manifests: `mkdir -p ~/manifests`
2. **Write a manifest** that installs nginx and ensures the service is running and enabled at boot:

    ```puppet
    # ~/manifests/site.pp
    package { 'nginx':
      ensure => installed,
    }

    service { 'nginx':
      ensure => running,
      enable => true,
    }
    ```

3. **Apply it:** `sudo puppet apply ~/manifests/site.pp`
4. **Visit** `http://is-puppetlabXX.uoregon.edu` in your browser to confirm nginx is serving the default page

## Part 2: Managing File Content

!!! note "Talk about: Resources and desired state"
    - **Resources** -- the building block of Puppet: a type (`package`, `service`, `file`), a title, and properties
    - **Abstraction** -- you say "nginx should be installed," Puppet picks the right tool (`apt`, `yum`) for the OS
    - The `file` resource -- declare what a file should contain, Puppet enforces it
    - **Idempotency in practice** -- re-run the Part 1 manifest and nothing changes

1. **Add a file resource** to your manifest that creates a file in the nginx docroot using the `content` property:

    ```puppet
    # ~/manifests/site.pp
    file { '/var/www/html/index.html':
      ensure  => file,
      content => "<h1>Hello from server XX</h1>\n",
    }
    ```

3. **Apply it:** `sudo puppet apply ~/manifests/site.pp`
4. **Check your browser** to see the change

## Part 3: ERB Templates

!!! note "Talk about: Why templates instead of hardcoded content?"
    - The scaling problem -- hardcoded content means a separate file per server
    - **Templates** -- write a config once, fill in server-specific values at apply time
    - **Facts** -- system information Puppet discovers automatically (`hostname`, `os`, `ipaddress`)
    - One template serves every server -- from 1 to 400

1. **Create a templates directory:** `mkdir -p ~/manifests/templates`
2. **Move the content into an ERB template:**

    ```erb
    <%# ~/manifests/templates/index.html.erb %>
    <h1>Hello from <%= @hostname %></h1>
    <p>Managed by Puppet</p>
    ```

3. **Update your manifest** to use the template instead of `content`:

    ```puppet
    # ~/manifests/site.pp
    file { '/var/www/html/index.html':
      ensure  => file,
      content => template('templates/index.html.erb'),
    }
    ```

4. **Apply and verify** in your browser; notice the hostname is filled in automatically from the fact

## Part 4: Templating Config Files

!!! note "Talk about: The Package, File, Service pattern and relationships"
    - **Package → File → Service** -- the pattern behind most real-world Puppet work
    - **`notify`** -- the relationship that tells Puppet "if this file changes, restart that service"
    - What happens without `notify` -- config updates silently, service keeps running the old version
    - **Manifest variables** -- keep tuneable values (`$nginx_worker_connections`) at the top of the file instead of buried in templates

1. **Copy the running nginx config** to your templates directory:

    ```bash
    cp /etc/nginx/nginx.conf ~/manifests/templates/nginx.conf.erb
    ```

2. **Edit the template** to replace the hardcoded `nginx_worker_connections` value with an ERB variable:

    ```
    worker_connections <%= @nginx_worker_connections %>;
    ```

3. **Update your manifest** to define the variable and manage the nginx config with a service restart on change:

    ```puppet
    # ~/manifests/site.pp
    $nginx_worker_connections = 512

    file { '/var/www/html/index.html':
      ensure  => file,
      content => template('templates/index.html.erb'),
    }

    file { '/etc/nginx/nginx.conf':
      ensure  => file,
      content => template('templates/nginx.conf.erb'),
      notify  => Service['nginx'],
    }

    service { 'nginx':
      ensure => running,
      enable => true,
    }
    ```

4. **Apply and verify:** `sudo puppet apply ~/manifests/site.pp`
5. **Check the rendered config:** `cat /etc/nginx/nginx.conf` and confirm `nginx_worker_connections` is set to `512`
6. **Change the variable** in your manifest to a different value (e.g., `2048`), re-apply, and confirm the config updated and the service restarted

## Part 5: Exploring Facts with Facter

!!! note "Talk about: Where do facts come from?"
    - **Facter** -- a tool bundled with Puppet that discovers system information
    - What facts cover -- hardware (CPUs, memory), networking (IP, hostname), OS details
    - Where facts are available -- in templates (`@hostname`) and in manifests (`$facts`)
    - The `facter` command -- query facts from the command line to see what's available

1. **See all available facts:** `facter`
2. **Query specific facts:**

    ```bash
    facter hostname
    facter processors.count
    facter memory.system.total
    ```

3. **Update your index template** to display system info:

    ```erb
    <%# ~/manifests/templates/index.html.erb %>
    <h1>Hello from <%= @hostname %></h1>
    <p>Managed by Puppet</p>
    <h2>System Info</h2>
    <ul>
      <li>CPUs: <%= @processors['count'] %></li>
      <li>Memory: <%= @memory['system']['total'] %></li>
    </ul>
    ```

4. **Apply and check your browser:** `sudo puppet apply ~/manifests/site.pp`
5. **Compare with your neighbor** -- everyone's page should show different values (or the same, depending on the lab VMs)

## Part 6: Conditional Logic with Facts

!!! note "Talk about: Making decisions based on the system"
    - **Conditional logic** -- Puppet supports `if`/`else` to branch based on facts
    - Same manifest, different behavior depending on hardware, OS, or role
    - Keeping logic in the manifest -- templates stay simple, decisions live in one place

1. **Replace the hardcoded `$nginx_worker_connections`** in your manifest with a conditional based on CPU count:

    ```puppet
    if $facts['processors']['count'] > 2 {
      $nginx_worker_connections = 2048
    } else {
      $nginx_worker_connections = 512
    }
    ```

2. **Apply and check the result:**

    ```bash
    sudo puppet apply ~/manifests/site.pp
    grep worker_connections /etc/nginx/nginx.conf
    ```

3. **Check how many CPUs your server has:** `facter processors.count`
4. **Confirm the value matches** what your conditional should have chosen

## Recap

- `puppet apply` runs a manifest locally
- The `package` resource installs software, the `service` resource manages daemons
- The `file` resource manages file content (with `content` or `template()`)
- ERB templates pull in facts (like `@hostname`) and manifest variables (like `@nginx_worker_connections`)
- `facter` discovers system information (CPUs, memory, OS, networking) available in templates and in manifests via `$facts`
- `if`/`else` conditionals let the same manifest behave differently based on facts
- `notify` triggers a service restart when a config file changes
- The **Package, File, Service** pattern is the foundation of most Puppet work

---

## Instructor Resources

### Server Requirements

Each lab server (`is-puppetlabXX`) needs the following before the session:

| Requirement | Details |
|-------------|---------|
| Puppet installed | `puppet` agent package available on the system |
| User accounts | `participantXX` created and allowed to log in |
| Sudo access | `participantXX` must be able to run `sudo puppet apply` and `sudo cp` without restriction |
| Firewall | Port 22 (SSH) and port 80 (HTTP) open |
| DNS | `is-puppetlabXX.uoregon.edu` resolves to the correct server |
| Nginx package available | `nginx` must be in the apt/yum repos but **not** pre-installed (participants install it in Part 1) |

### Lab Server nginx.conf

This is the nginx config that should be deployed to each lab server before participants arrive. It serves a simple docroot on port 80.

```nginx
# /etc/nginx/nginx.conf
user www-data;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    sendfile      on;
    keepalive_timeout 65;

    server {
        listen 80;
        server_name _;

        root /var/www/html;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
}
```

### Finished Files

These are the completed files participants should have at the end of each part (useful for catching people up or verifying work).

**After Part 1** -- `~/manifests/site.pp`:

```puppet
package { 'nginx':
  ensure => installed,
}

service { 'nginx':
  ensure => running,
  enable => true,
}
```

**After Part 2** -- `~/manifests/site.pp`:

```puppet
package { 'nginx':
  ensure => installed,
}

service { 'nginx':
  ensure => running,
  enable => true,
}

file { '/var/www/html/index.html':
  ensure  => file,
  content => "<h1>Hello from server XX</h1>\n",
}
```

**After Part 3** -- `~/manifests/site.pp`:

```puppet
package { 'nginx':
  ensure => installed,
}

service { 'nginx':
  ensure => running,
  enable => true,
}

file { '/var/www/html/index.html':
  ensure  => file,
  content => template('templates/index.html.erb'),
}
```

`~/manifests/templates/index.html.erb`:

```erb
<h1>Hello from <%= @hostname %></h1>
<p>Managed by Puppet</p>
```

**After Part 4** -- `~/manifests/site.pp`:

```puppet
$nginx_worker_connections = 512

package { 'nginx':
  ensure => installed,
}

service { 'nginx':
  ensure => running,
  enable => true,
}

file { '/var/www/html/index.html':
  ensure  => file,
  content => template('templates/index.html.erb'),
}

file { '/etc/nginx/nginx.conf':
  ensure  => file,
  content => template('templates/nginx.conf.erb'),
  notify  => Service['nginx'],
}
```

`~/manifests/templates/nginx.conf.erb`:

```erb
user www-data;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

events {
    worker_connections <%= @nginx_worker_connections %>;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    sendfile      on;
    keepalive_timeout 65;

    server {
        listen 80;
        server_name _;

        root /var/www/html;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
}
```

**After Part 5** -- manifest unchanged, only the template is updated.

`~/manifests/templates/index.html.erb`:

```erb
<h1>Hello from <%= @hostname %></h1>
<p>Managed by Puppet</p>
<h2>System Info</h2>
<ul>
  <li>CPUs: <%= @processors['count'] %></li>
  <li>Memory: <%= @memory['system']['total'] %></li>
</ul>
```

**After Part 6** -- `~/manifests/site.pp`:

```puppet
if $facts['processors']['count'] > 2 {
  $nginx_worker_connections = 2048
} else {
  $nginx_worker_connections = 512
}

package { 'nginx':
  ensure => installed,
}

service { 'nginx':
  ensure => running,
  enable => true,
}

file { '/var/www/html/index.html':
  ensure  => file,
  content => template('templates/index.html.erb'),
}

file { '/etc/nginx/nginx.conf':
  ensure  => file,
  content => template('templates/nginx.conf.erb'),
  notify  => Service['nginx'],
}
```

`~/manifests/templates/nginx.conf.erb` -- unchanged from Part 4.
