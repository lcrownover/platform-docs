# Puppet Labs

Hands-on labs covering Puppet configuration management, from writing your first manifest to advanced topics.

## Available Labs

| Lab | Description | Duration |
|-----|-------------|----------|
| [Puppet 101](puppet-101.md) | Install packages, manage files, use templates, and explore facts with `puppet apply` | Half day |
| [Puppet 102](puppet-102.md) | Team modules, roles and profiles, Hiera data layers, and cluster-level configuration | Half day |

## What You'll Learn

**Puppet 101** starts from scratch on a fresh Linux server. You'll install nginx with a package resource, manage files with both inline content and ERB templates, wire up the Package/File/Service pattern with `notify`, and explore system facts with Facter. By the end, you'll have a working manifest that installs, configures, and manages a web server.

**Puppet 102** moves from standalone manifests to working with shared Puppet code in a team module. You'll explore the roles and profiles pattern, learn how Hiera separates data from code at the node, cluster, and common levels, and make changes that affect individual servers or entire groups.
