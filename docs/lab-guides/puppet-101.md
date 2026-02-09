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

## Recap

- `puppet apply` runs a manifest locally
- The `package` resource installs software, the `service` resource manages daemons
- The `file` resource manages file content (with `content` or `template()`)
- ERB templates pull in facts (like `@hostname`) and manifest variables (like `@nginx_worker_connections`)
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
