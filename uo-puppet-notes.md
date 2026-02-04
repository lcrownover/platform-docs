# UO Puppet Infrastructure Notes

Reference notes for writing the Puppet handbook.

## Key Concepts

- Cattle vs. pets: UO strives for this, but environment means many one-off servers
- Modified roles/profiles: only servers that share a role are members of the same "cluster"
- Team modules: each team has their own module (puppet_dba, puppet_cas, etc.)

## hiera.yaml (puppet-control-repo)

The control repo is read-only but available to PR against and viewable to all teams.

```yaml
---
version: 5
defaults:
  # The default value for "datadir" is "data" under the same directory as the hiera.yaml
  # file (this file)
  # When specifying a datadir, make sure the directory exists.
  # See https://puppet.com/docs/puppet/latest/environments_about.html for further details on environments.
  # datadir: data
  # data_hash: yaml_data
  lookup_key: eyaml_lookup_key
  options:
    pkcs7_private_key: /etc/puppetlabs/puppet/keys/eyaml_private_key.pkcs7.pem
    pkcs7_public_key:  /etc/puppetlabs/puppet/keys/eyaml_public_key.pkcs7.pem

hierarchy:
  # There should be nothing at the environment-level for nodes and clusters
  # but this allows us to override someone else's yaml if we have to
  - name: "Environment level node data"
    path: "nodes/%{hostname}.yaml"

  # This is the main section of hiera that does lookups for each group.
  # The "pp_group" fact is set via a custom fact in the lib/ directory in the
  # control-repo/site-modules/profile module.
  - name: "puppet_%{pp_group}"
    datadir: "modules/puppet_%{pp_group}/data/"
    paths:
      - "nodes/%{hostname}.yaml"
      - "clusters/%{pp_group}/%{pp_cluster}.yaml"
      - "%{pp_group}_common.yaml"

  # Fall back to environment-level common for UO defaults
  - name: "Common data"
    path: "common.yaml"
```

## Puppetfile (team modules)

Team modules are imported from git.uoregon.edu. All team modules start with `puppet_`.

```ruby
# Role/Profiles from git.uoregon.edu
mod 'puppet_mad', :git => 'ssh://git@git.uoregon.edu/pup/puppet_mad.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_cas', :git => 'ssh://git@git.uoregon.edu/pup/puppet_cas.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_ctl', :git => 'ssh://git@git.uoregon.edu/pup/puppet_ctl.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_systems', :git => 'ssh://git@git.uoregon.edu/pup/puppet_systems.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_css', :git => 'ssh://git@git.uoregon.edu/pup/puppet_css.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_dba', :git => 'ssh://git@git.uoregon.edu/pup/puppet_dba.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_hou', :git => 'ssh://git@git.uoregon.edu/pup/puppet_hou.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_ids', :git => 'ssh://git@git.uoregon.edu/pup/puppet_ids.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_infosec', :git => 'ssh://git@git.uoregon.edu/pup/puppet_infosec.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_nts', :git => 'ssh://git@git.uoregon.edu/pup/puppet_nts.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_telecom', :git => 'ssh://git@git.uoregon.edu/pup/puppet_telecom.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_admin', :git => 'ssh://git@git.uoregon.edu/pup/puppet_admin.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_edm', :git => 'ssh://git@git.uoregon.edu/pup/puppet_edm.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_acadnorth', :git => 'ssh://git@git.uoregon.edu/pup/puppet_acadnorth.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_acadsouth', :git => 'ssh://git@git.uoregon.edu/pup/puppet_acadsouth.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_acadcentral', :git => 'ssh://git@git.uoregon.edu/pup/puppet_acadcentral.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_devservices', :git => 'ssh://git@git.uoregon.edu/pup/puppet_devservices.git', :branch => :control_branch, :default_branch => 'master'
mod 'puppet_racs', :git => 'ssh://git@git.uoregon.edu/pup/puppet_racs.git', :branch => :control_branch, :default_branch => 'main'
mod 'puppet_zfin', :git => 'ssh://git@git.uoregon.edu/pup/puppet_zfin.git', :branch => :control_branch, :default_branch => 'main'
```

## site.pp (puppet-control-repo/manifests/site.pp)

```puppet
Exec { path => '/usr/bin:/usr/sbin:/bin:/sbin:/usr/local/bin:/usr/local/sbin:/opt/puppetlabs/bin' }

# Nodes are not specifically defined in the control repo
# Instead, they should have the fact "pp_role" set to
#   whatever role they should include
# If pp_role is not defined, it will put a "missing_pp_role" fact
#   in place that we can generate a report on in PE.

node 'default' {
  include facter

  # if Linux, we configure the repo to be the puppetserver
  if $facts['kernel'] == 'Linux' {
    class { 'puppet_agent':
      package_version => 'auto',
    }
  }

  # Include the defined role
  if $facts['pp_role'] {
    include $facts['pp_role']
    facter::fact { 'missing_pp_role': ensure => absent }
  }
  # Otherwise do nothing and echo on runs
  else {
    echo { 'missing_pp_role': message => 'pp_role has not been set on this node' }
    facter::fact { 'missing_pp_role': value => true }
  }
}
```

## Base Profile Location

Managed base configuration is at `puppet-control-repo/site-modules/profile/manifests/base.pp`.
This module is included (by convention) in ALL roles in ALL team folders.

## Team Module Structure

Team modules (e.g., `puppet_dba`) contain only `profile` and `role` directories in `manifests/`.

### Example Role (puppet_dba::role::loire)

```puppet
#puppet_dba::role::loire
class puppet_dba::role::loire {
  system_tags::tag { 'ou': value => 'IS' }
  system_tags::tag { 'subou': value => 'Database_Administration' }
  system_tags::tag { 'service': value => get_role_name($title) }

  include profile::base
  include puppet_dba::profile::base
  include puppet_dba::profile::oracle::server
  include puppet_dba::profile::loire
}
```

### Role Structure Conventions

1. **System tags** - for log aggregation and other services
2. **profile::base** - always included (UO-wide base)
3. **team::profile::base** - team-specific base (SSH keys, sudo fragments, etc.)
4. **Zero or more shared profiles** - typical reusable technical capabilities
5. **"Dump profile"** (optional) - same name as role, for non-reusable config

The role and "dump profile" share the same name by design - helps when searching for code with 1-to-1 use with a given role.

### Example Dump Profile (puppet_dba::profile::loire)

```puppet
#puppet_dba::profile::loire
class puppet_dba::profile::loire {
  include puppet_dba::profile::sshkeys::aia
  include puppet_dba::profile::sshkeys::loire
...
}
```

Teams have full flexibility of the module autoloader to create reusable modules inside their module.
