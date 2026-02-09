# Puppet

You've probably dealt with this: you SSH into a server to tweak a config file, fix a permission, or install a package. It works. Then six months later, you rebuild the server and spend hours remembering what you changed. Or worse, someone else makes a "quick fix" that breaks something, and nobody knows what the system is supposed to look like anymore.

Puppet solves this by letting you describe what your systems should look like, then continuously enforcing that state. Instead of running commands and hoping they stick, you declare "this file should exist with these contents" or "this service should be running," and Puppet makes it so, over and over again.

## Why You Should Care

If you manage more than a handful of servers (or even just a few that matter), Puppet gives you:

- **Self-healing infrastructure.** Someone manually edits a config file they shouldn't have? Puppet notices and fixes it on the next run. Your systems stay in the state you defined, not the state they drifted into.
- **Documentation that actually works.** Your Puppet code *is* your documentation. Instead of tribal knowledge about how servers are configured, you have readable, versioned code that describes exactly what's on each system.
- **Reproducible environments.** Need to rebuild a server? Stand up a new one? Puppet applies the same configuration automatically. No more snowflake servers that nobody dares touch.
- **Scalable change management.** Update a config in one place, and Puppet rolls it out to hundreds of servers. No more SSH-in-a-loop scripts or hoping you didn't miss a box.

## The Agent-Based Approach

Puppet's agent-based architecture is what sets it apart. A small agent runs on every managed server, checking in with a central Puppet server every 30 minutes (by default). The agent asks "what should I look like?" and the server responds with a catalog of resources. The agent then enforces that state locally, reporting back what it changed.

This means Puppet catches and corrects configuration drift automatically. If a process crashes and corrupts a config file, Puppet restores it. You don't have to remember to run anything; the system continuously converges toward your defined state.

Other configuration management tools like Ansible, Chef, and SaltStack exist and have their place. But Puppet's continuous enforcement model is particularly powerful for environments where you need confidence that servers stay configured correctly without constant human intervention. You set the desired state, and Puppet keeps it that way.

**One important thing to understand:** Puppet only manages what you tell it to manage. If you don't define a resource, Puppet won't touch it. This is a feature, not a limitation. It means you can adopt Puppet incrementally, starting with a few critical configs and expanding over time. But it also means you get out what you put in. A server with one Puppet-managed file and dozens of hand-edited configs will still drift in all the places Puppet isn't watching.

## Topics

Work through these in order:

1. **Puppet Fundamentals**: Understand Puppet's architecture, resource types, and how it models system state
2. **Writing Manifests**: Create your first Puppet code to manage files, packages, and services
3. **Classes and Modules**: Organize code into reusable, shareable components
4. **Hiera and Data Separation**: Keep your code generic and your environment-specific data separate
5. **Roles and Profiles**: Structure your Puppet codebase for real-world complexity
6. **Testing and Development**: Write tests, use local VMs, and develop safely before pushing to production
7. **Custom Facts**: Expose environment-specific information to your Puppet code

## Tips for Success

- **Think declaratively.** Puppet isn't a scripting language. Instead of "run these commands in order," think "these resources should exist in this state." It takes some mental adjustment, but it's more powerful once it clicks.
- **Start small.** Don't try to Puppetize your entire infrastructure at once. Pick one service on one server, get it working, then expand.
- **Use version control.** Your Puppet code is infrastructure-as-code. Treat it like any other codebase: commit changes, write meaningful messages, use branches, review before merging.
- **Read the catalog, not just the code.** When debugging, remember that Puppet compiles your code into a catalog before applying it. Understanding this two-phase process helps when things don't behave as expected.
