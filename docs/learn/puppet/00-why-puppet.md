# Why Puppet?

By the end of this section, you'll understand the real problem Puppet solves and why a bash script, however familiar, isn't the right tool for keeping systems configured.

## Start with a Script

You get a ticket: deploy nginx on a web server with a standard configuration. You've done this a hundred times. You write this:

```bash
#!/bin/bash
dnf install -y nginx

cat > /etc/nginx/nginx.conf << 'EOF'
worker_processes auto;
events { worker_connections 1024; }
http {
  include       /etc/nginx/mime.types;
  default_type  application/octet-stream;
  server {
    listen 80;
    root /var/www/html;
  }
}
EOF

systemctl enable nginx
systemctl start nginx
```

Clean. Readable. It works. Ticket closed.

## Then the Requirements Change

Three weeks later: the script needs to run on an Ubuntu server. You add OS detection:

```bash
if [ -f /etc/debian_version ]; then
    apt-get install -y nginx
elif [ -f /etc/redhat-release ]; then
    dnf install -y nginx
else
    echo "Unsupported OS" >&2; exit 1
fi
```

Still manageable. But now the team wants this script to run on a schedule, nightly, to keep servers in the right state. That means it'll run on machines where nginx is *already installed*. Running `dnf install -y nginx` on an already-installed system isn't just wasteful; on some systems it can trigger config file prompts or trigger a package upgrade you didn't intend. So you need an idempotency check. But the check looks different on each platform:

```bash
if [ -f /etc/debian_version ]; then
    if ! dpkg -l nginx 2>/dev/null | grep -q '^ii'; then
        apt-get install -y nginx
    fi
elif [ -f /etc/redhat-release ]; then
    if ! rpm -q nginx &>/dev/null; then
        dnf install -y nginx
    fi
fi
```

## The Config Gets Complicated

Now for the config file. You can't blindly overwrite `/etc/nginx/nginx.conf` every run, since servers might have site-specific customizations. So initially you check if the file exists first:

```bash
if [ ! -f /etc/nginx/nginx.conf ]; then
    cat > /etc/nginx/nginx.conf << 'EOF'
    ...
    EOF
fi
```

But that creates a new problem: the script will never update the config even when you need to push a new standard config out. You switch to a checksum comparison, and since you're touching the file, you also have to explicitly set ownership and permissions:

```bash
FILES_SOURCE="/var/files-source"
DESIRED_CHECKSUM=$(sha256sum "$FILES_SOURCE/nginx.conf" | cut -d' ' -f1)
CURRENT_CHECKSUM=$(sha256sum /etc/nginx/nginx.conf 2>/dev/null | cut -d' ' -f1)
RELOAD_NEEDED=false

if [ "$CURRENT_CHECKSUM" != "$DESIRED_CHECKSUM" ]; then
    cp "$FILES_SOURCE/nginx.conf" /etc/nginx/nginx.conf
    chown root:root /etc/nginx/nginx.conf
    chmod 644 /etc/nginx/nginx.conf
    RELOAD_NEEDED=true
fi
```

And since nginx is running, you want to avoid a full restart (which drops connections). You need `reload` when the config changed, `start` if it was stopped, and nothing if it's already running and the config is unchanged:

```bash
if ! systemctl is-enabled nginx &>/dev/null; then
    systemctl enable nginx
fi

if ! systemctl is-active nginx &>/dev/null; then
    systemctl start nginx
elif [ "$RELOAD_NEEDED" = "true" ]; then
    systemctl reload nginx
fi
```

## What You've Built

Step back and look at the full script:

```bash
#!/bin/bash
set -euo pipefail

FILES_SOURCE="/var/files-source"
RELOAD_NEEDED=false

if [ -f /etc/debian_version ]; then
    if ! dpkg -l nginx 2>/dev/null | grep -q '^ii'; then
        apt-get install -y nginx
    fi
elif [ -f /etc/redhat-release ]; then
    if ! rpm -q nginx &>/dev/null; then
        dnf install -y nginx
    fi
else
    echo "Unsupported OS" >&2; exit 1
fi

DESIRED_CHECKSUM=$(sha256sum "$FILES_SOURCE/nginx.conf" | cut -d' ' -f1)
CURRENT_CHECKSUM=$(sha256sum /etc/nginx/nginx.conf 2>/dev/null | cut -d' ' -f1)
if [ "$CURRENT_CHECKSUM" != "$DESIRED_CHECKSUM" ]; then
    cp "$FILES_SOURCE/nginx.conf" /etc/nginx/nginx.conf
    chown root:root /etc/nginx/nginx.conf
    chmod 644 /etc/nginx/nginx.conf
    RELOAD_NEEDED=true
fi

if ! systemctl is-enabled nginx &>/dev/null; then
    systemctl enable nginx
fi

if ! systemctl is-active nginx &>/dev/null; then
    systemctl start nginx
elif [ "$RELOAD_NEEDED" = "true" ]; then
    systemctl reload nginx
fi
```

50 lines to manage one package, one file, and one service. And it *still* has gaps: 

- What if nginx is installed but the service is disabled? 
- What if the config has the right content but wrong permissions set by a previous admin? 
- What if someone stops the service manually between runs and your script doesn't notice because `is-active` returns false but the package and config are fine?

You could handle all of those cases. But every new case adds another branch, another state variable, another path that has to be tested. The script becomes harder to read, harder to trust, and harder to hand off to someone else.

!!! note
    This isn't a knock on bash. Bash is the right tool for plenty of tasks. The problem here is that bash describes *steps to take*, and managing system state requires something more like *describing what things should look like*.

## The Puppet Version

Here's the same thing in Puppet:

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
  notify  => Service['nginx'],
}

service { 'nginx':
  ensure => running,
  enable => true,
}
```

15 lines. Read it out loud and it almost sounds like a specification: "nginx should be installed; this config file should exist with these attributes; the nginx service should be running and enabled." That's exactly what it is.

This code handles everything the bash script does, plus the gaps we identified:

- **Cross-platform by default.** Puppet knows how to install packages and manage services on Debian, RedHat, and other platforms without a single `if` statement in your code.
- **Idempotent without effort.** Puppet only changes what's out of state. If nginx is installed and the config is correct, Puppet does nothing and moves on.
- **Handles drift.** If someone manually edits the config or stops the service, Puppet corrects it on the next run, without you writing any detection logic.
- **Permissions enforced automatically.** The `owner`, `group`, and `mode` attributes are checked and corrected every run, not just when the content changes.
- **Connects behaviors.** The `notify` relationship tells Puppet to reload nginx whenever the config file changes, and only then.

The bash script describes how to configure nginx. The Puppet code describes what a correctly configured nginx server *looks like*. That shift in perspective, from imperative steps to declarative state, is what Puppet is built around, and it's what the rest of this module is about.
