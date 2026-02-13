# Puppet 102 Lab

## Prerequisites

| Tool | Why You Need It |
|------|-----------------|
| Terminal with SSH | You'll connect to a remote lab server over SSH. macOS/Linux: use the built-in terminal. Windows: use [WSL](../tools/wsl.md) or [Windows Terminal](../tools/windows-terminal.md). |
| [Git](../tools/git.md) | You'll clone and push changes to a shared Puppet team module |
| [Command-line editor](../tools/cli-editors.md) | You'll edit manifests, templates, and Hiera data on your local machine. [Nano](../tools/cli-editors.md#nano) is the easiest option. |

| Lab | Why |
|------------------|-----|
| [Puppet 101](puppet-101.md) | You should understand resources, templates, facts, and `puppet apply` |
| [Git 101](git-101.md) | You'll need to clone, edit, commit, and push |

## Setup

- You've been assigned a number (1-40). Throughout this lab, replace `XX` with your zero-padded number (e.g., 5 → `05`)
- You're in a group (1-5 = group1, 6-10 = group2, etc.). Replace `N` with your group number.
- SSH into your server:

    ```bash
    ssh participantXX@is-puppetlabXX.uoregon.edu
    ```

    Password: `UOISLabNumber@XX`

- Clone your group's team module:

    ```bash
    git clone URL/groupN/puppet_groupN.git
    cd puppet_groupN
    ```

- Nginx is already installed and running from Puppet; visit `http://is-puppetlabXX.uoregon.edu` in your browser to confirm

## Part 1: Understanding the Team Module

!!! note "Talk about: Delegated administration and team modules"
    - **Delegated administration** -- each team owns their own Puppet module where they manage their servers
    - **Team modules** -- contain roles, profiles, Hiera data, files, and templates for a team's servers
    - **Control repo** -- the central repo that ties everything together (read-only to most users)
    - **Separation of concerns** -- your team module is your space; you don't need to touch the control repo for day-to-day work

1. **Explore the repo structure:**

    ```bash
    ls -R
    ```

    You should see something like:

    ```
    puppet_groupN/
    ├── data/
    │   ├── nodes/
    │   │   └── is-puppetlabXX.yaml   # (one per group member)
    │   ├── clusters/
    │   │   └── groupN/
    │   │       └── ...
    │   └── groupN_common.yaml
    ├── files/
    ├── templates/
    └── manifests/
        ├── profile/
        │   ├── base.pp
        │   ├── nginx.pp
        │   └── webserver.pp
        └── role/
            └── webserver.pp
    ```

2. **Read the role file** (`manifests/role/webserver.pp`) and identify the includes
3. **Read the profiles** that the role includes and understand what each one does
4. **Read the Hiera data** in `data/groupN_common.yaml` to see what defaults are set

!!! tip "Review: What we learned"
    - The team module is a single Git repo containing everything for your team's servers
    - `manifests/` holds only two subdirectories: `role/` and `profile/`
    - `data/` holds Hiera YAML files at three levels: node, cluster, and common
    - `templates/` and `files/` hold content that profiles deploy to servers

## Part 2: Roles, Profiles, and How They Connect

!!! note "Talk about: The roles and profiles pattern"
    - **Roles** define *what* a server is -- they only contain `include` statements
    - **Profiles** define *how* something is configured -- they contain the actual Puppet code
    - **Server profiles** -- a profile with the same name as the role, for configuration specific to that server
    - **Reusable profiles** -- shared configuration that multiple roles can include (e.g., `profile::nginx`)
    - The layering: **Server → Role → Profiles → Modules**
    - Roles stay clean and readable; all configuration logic lives in profiles

1. **Open the role file** and trace each `include` to its profile
2. **Open the server profile** (`manifests/profile/webserver.pp`) and note how it differs from the reusable profiles
3. **Discuss with your group:** Which profiles are reusable? Which are server-specific?

!!! tip "Review: What we learned"
    - A role is a list of `include` statements -- nothing else
    - Profiles contain the actual Puppet resources and logic
    - Server profiles handle one-off config; reusable profiles are shared across roles
    - The `pp_role` fact on each server determines which role class gets applied

## Part 3: Hiera and Node-Level Data

!!! note "Talk about: Hiera and the data hierarchy"
    - **Hiera** -- Puppet's built-in key/value lookup system for separating data from code
    - **Hierarchy** -- Hiera checks multiple YAML files in order and uses the first match
    - **Lookup order** for team modules: node → cluster → common
    - **Node-level data** -- overrides for a specific server (in `data/nodes/<hostname>.yaml`)
    - Why separate data from code -- the same profile works across servers; only the data changes

1. **Check your node data file** (`data/nodes/is-puppetlabXX.yaml`) and note any values set there
2. **Create a template** that displays a value from Hiera. Add a file to `templates/`:

    ```erb
    <%# templates/info.html.erb %>
    <h1><%= @hostname %></h1>
    <p>Welcome message: <%= @welcome_message %></p>
    ```

3. **Add the Hiera key** to your node data file (`data/nodes/is-puppetlabXX.yaml`):

    ```yaml
    puppet_groupN::profile::webserver::welcome_message: "Hello from participantXX"
    ```

4. **Update the server profile** (`manifests/profile/webserver.pp`) to use the Hiera value and serve the template:

    ```puppet
    class puppet_groupN::profile::webserver (
      String $welcome_message = 'Default welcome',
    ) {
      file { '/var/www/html/index.html':
        ensure  => file,
        content => template('puppet_groupN/info.html.erb'),
      }
    }
    ```

5. **Commit and push:**

    ```bash
    git add .
    git commit -m "Add welcome message from node Hiera data"
    git push
    ```

6. **Run Puppet on your server** and check your browser:

    ```bash
    sudo puppet agent -t
    ```

!!! tip "Review: What we learned"
    - Hiera automatically maps YAML keys to class parameters (automatic parameter lookup)
    - Node-level data in `data/nodes/<hostname>.yaml` applies to one specific server
    - The same profile code serves every server -- only the data differs
    - The workflow is: edit code/data → commit → push → run Puppet

## Part 4: Common Data and Shared Profiles

!!! note "Talk about: Common Hiera data and team-wide profiles"
    - **Common data** (`data/groupN_common.yaml`) -- defaults that apply to all servers in your team module
    - A value in a node file overrides the same key in common (Hiera lookup order)
    - **Shared profiles** -- a profile included in every role so all servers get the same configuration
    - Adding a profile to all roles via the team base profile

1. **Add a Hiera key to your common data** (`data/groupN_common.yaml`):

    ```yaml
    puppet_groupN::profile::motd::motd_message: "Managed by groupN"
    ```

2. **Create a new shared profile** (`manifests/profile/motd.pp`):

    ```puppet
    class puppet_groupN::profile::motd (
      String $motd_message = 'Managed by Puppet',
    ) {
      file { '/var/www/html/motd.html':
        ensure  => file,
        content => "${motd_message}\n",
      }
    }
    ```

3. **Include the new profile in your team's base profile** (`manifests/profile/base.pp`):

    ```puppet
    include puppet_groupN::profile::motd
    ```

4. **Commit and push:**

    ```bash
    git add .
    git commit -m "Add MOTD profile to all servers via base profile"
    git push
    ```

5. **Run Puppet on your server** and verify:

    ```bash
    sudo puppet agent -t
    ```

    Visit `http://is-puppetlabXX.uoregon.edu/motd.html` to confirm the message.

6. **Check with your groupmates** -- everyone should see the same message since it comes from common data

!!! tip "Review: What we learned"
    - Common data sets defaults for every server in the team module
    - New profiles become team-wide by including them in the team's base profile
    - Node-level data overrides common data for the same key (Hiera lookup order)
    - Every group member sees the same result because they share the same common data

## Part 5: Cluster-Level Hiera with pp_cluster

!!! note "Talk about: Clusters and the pp_cluster fact"
    - **pp_cluster** -- an optional fact that groups servers into clusters
    - **Cluster-level Hiera** -- a YAML file that applies to all servers sharing a `pp_cluster` value
    - Lookup order with clusters: node → cluster → common
    - When to use clusters -- when a subset of servers need shared config that differs from the team default
    - Setting the fact with `set_pp_cluster`

1. **Set the `pp_cluster` fact on your server.** Your instructor will assign clusters within your group (e.g., half the group is cluster `web`, half is cluster `api`):

    ```bash
    sudo set_pp_cluster web    # or api, as assigned
    ```

2. **Create cluster Hiera data files.** In your repo, add files for each cluster:

    `data/clusters/groupN/web.yaml`:

    ```yaml
    puppet_groupN::profile::webserver::welcome_message: "Web cluster server"
    ```

    `data/clusters/groupN/api.yaml`:

    ```yaml
    puppet_groupN::profile::webserver::welcome_message: "API cluster server"
    ```

3. **Commit and push:**

    ```bash
    git add .
    git commit -m "Add cluster-level Hiera data for web and api clusters"
    git push
    ```

4. **Run Puppet on your server:**

    ```bash
    sudo puppet agent -t
    ```

5. **Check your browser** -- your welcome message should now reflect your cluster assignment
6. **Compare with a groupmate in the other cluster** -- same profile, same template, different data

!!! tip "Review: What we learned"
    - `pp_cluster` groups servers into clusters for shared configuration
    - Cluster-level Hiera sits between node and common in the lookup order
    - Different clusters can have different values for the same key without duplicating node files
    - The full lookup order: node → cluster → common

## Recap

- **Team modules** contain all the code and data for your team's servers (roles, profiles, Hiera, templates)
- **Roles** define what a server is (only `include` statements); **profiles** define how it's configured
- **Hiera** separates data from code with a layered lookup: node → cluster → common
- **Node-level data** overrides everything for a specific server
- **Common data** sets defaults for all servers in your team module
- **Cluster-level data** (via `pp_cluster`) lets a subset of servers share config that differs from the common defaults
- The workflow: edit code/data in the team module → commit → push → run Puppet on the server

---

## Instructor Resources

### Server Requirements

Each lab server (`is-puppetlabXX`) needs the following before the session:

| Requirement | Details |
|-------------|---------|
| Puppet agent | Configured and able to reach the Puppet server |
| `pp_role` fact | Set to `puppet_groupN::role::webserver` for the participant's group |
| User accounts | `participantXX` created and allowed to log in |
| Sudo access | `participantXX` must be able to run `sudo puppet agent -t` and `sudo set_pp_cluster` |
| Firewall | Port 22 (SSH) and port 80 (HTTP) open |
| DNS | `is-puppetlabXX.uoregon.edu` resolves to the correct server |
| Nginx | Pre-installed and running via Puppet |

### Repository Requirements

| Requirement | Details |
|-------------|---------|
| Group repos | `groupN/puppet_groupN` for each group (1-8) |
| Pre-configured structure | Each repo should contain the full team module layout: `manifests/role/`, `manifests/profile/`, `data/`, `templates/`, `files/` |
| Existing role | `puppet_groupN::role::webserver` with includes for `profile::base` and `profile::nginx` |
| Existing profiles | `profile::base` (team base), `profile::nginx` (nginx setup), `profile::webserver` (server profile stub) |
| Node data files | `data/nodes/is-puppetlabXX.yaml` for each participant in the group |
| Common data | `data/groupN_common.yaml` with sensible defaults |
| Puppet server | Repos must be wired into the Puppet server so pushes are picked up |

### Finished State

After the lab, each group's `puppet_groupN` repo should have:

1. A `templates/info.html.erb` template using `@welcome_message`
2. Node-level Hiera data with per-participant welcome messages
3. A `profile::motd` included in `profile::base`
4. Common Hiera data for the MOTD message
5. Cluster-level Hiera data for `web` and `api` clusters
