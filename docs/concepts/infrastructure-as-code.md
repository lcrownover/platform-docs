# Infrastructure as Code

Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through machine-readable definition files rather than manual processes or interactive configuration tools.

## A Brief History

### The Before Times: Manual Administration

In the early days of system administration, every server was a unique snowflake. Administrators would SSH into machines, run commands, edit configuration files, and hope they remembered what they did. Documentation, if it existed at all, was a Word document or wiki page that was perpetually out of date.

This worked when you had a handful of servers. A skilled admin could keep the state of a few dozen machines in their head, and recovering from problems meant drawing on experience and institutional knowledge.

But this approach had fundamental problems:

- **Configuration drift.** Two servers that started identical would slowly diverge as administrators made one-off changes, applied patches at different times, or troubleshot issues in slightly different ways.
- **The documentation gap.** Written documentation required discipline to maintain and was almost always incomplete or outdated. The real documentation lived in people's heads.
- **Knowledge silos.** When experienced administrators left, their knowledge left with them. New team members faced archeological expeditions through undocumented systems.
- **Unreliable recovery.** Rebuilding a server from scratch meant remembering every manual step, every config file tweak, every package that was installed. In practice, this meant nobody wanted to rebuild anything.

### The Rise of Scripting

The first response to these problems was scripting. Instead of typing commands interactively, administrators wrote shell scripts to automate common tasks. This was a significant improvement—scripts could be saved, shared, and rerun.

But scripts had limitations:

- **Procedural thinking.** Scripts describe *how* to do something, not *what* the end result should be. Run a script twice and you might get different results depending on the starting state.
- **Error handling complexity.** Making scripts robust against failures—network timeouts, package conflicts, partial completions—required significant effort.
- **No built-in idempotency.** If a script failed halfway through, running it again might duplicate work or fail in new ways.

### Configuration Management Emerges

In the mid-2000s, a new generation of tools emerged to address these limitations. CFEngine had existed since 1993, but tools like Puppet (2005), Chef (2009), and later Ansible (2012) and Salt (2011) brought configuration management to the mainstream.

These tools introduced key concepts:

- **Declarative configuration.** Instead of describing steps, you describe the desired end state. "This server should have nginx installed and running." The tool figures out how to get there.
- **Idempotency.** Apply the same configuration multiple times and you get the same result. If nginx is already installed and running, the tool does nothing.
- **Convergence.** Systems continuously move toward their defined state. If someone manually changes a configuration, the tool changes it back.
- **Abstraction.** High-level resources (packages, services, files) hide platform-specific details. The same configuration can work across different operating systems.

### Cloud and the Explosion of Scale

The rise of cloud computing in the late 2000s and 2010s changed the game again. Suddenly, infrastructure wasn't just configuration—it was the servers themselves. You could spin up hundreds of virtual machines with an API call.

This drove the development of provisioning tools that extended IaC principles to infrastructure provisioning itself. Now you could define not just how servers should be configured, but what servers should exist, what networks they should be connected to, and what security policies should apply.

The scale also forced a philosophical shift. When you have thousands of servers, you can't treat them as pets with names and individual care. They become cattle—identical, replaceable, and disposable. If a server misbehaves, you don't debug it; you destroy it and spin up a new one.

## Core Principles

### Declarative Over Procedural

The most important principle of IaC is declarative configuration. You describe *what* you want, not *how* to get there.

**Procedural approach (shell script):**
```bash
if ! rpm -q nginx; then
    yum install -y nginx
fi
if ! systemctl is-active nginx; then
    systemctl start nginx
fi
if ! systemctl is-enabled nginx; then
    systemctl enable nginx
fi
```

**Declarative approach (Puppet):**
```puppet
package { 'nginx':
  ensure => installed,
}
service { 'nginx':
  ensure => running,
  enable => true,
}
```

The declarative version is shorter, but that's not the point. The point is that it describes the goal without encoding assumptions about the current state. The tool handles the logic of "if not already installed, install it."

### Idempotency

An operation is idempotent if applying it multiple times produces the same result as applying it once. This is crucial for IaC because configurations are applied repeatedly—on schedule, after changes, during recovery.

Idempotency means you can safely run your configuration management at any time without fear of side effects. If everything is already in the desired state, nothing happens. If something has drifted, it gets corrected.

### Version Control as the Source of Truth

IaC without version control is incomplete. When infrastructure is defined in code, that code should live in a version control system like Git. This provides:

- **History.** Every change is recorded with who made it, when, and why.
- **Auditability.** You can trace any configuration back to the commit that introduced it.
- **Rollback.** Reverting to a previous state means reverting to a previous commit.
- **Collaboration.** Multiple people can work on infrastructure using branches and pull requests.
- **Review.** Changes can be reviewed before they're applied, catching problems early.

The version control repository becomes the single source of truth. The question "what should this server look like?" is answered by the code in the repository, not by inspecting the server itself.

### Immutable Infrastructure

An evolution of IaC principles is immutable infrastructure—the idea that servers should never be modified after they're created. Instead of updating a running server, you build a new image with the changes and replace the old server entirely.

This approach:

- Eliminates configuration drift entirely
- Makes rollback trivial (deploy the previous image)
- Ensures development, staging, and production are truly identical
- Simplifies debugging (the server matches exactly what was tested)

Immutable infrastructure is most practical in cloud environments where creating and destroying servers is fast and cheap. It's enabled by image-building tools and container technologies.

## Benefits and Tradeoffs

### Benefits

**Consistency.** Every server built from the same code is identical. No more "works on my machine" or mysterious differences between environments.

**Speed.** Provisioning a new server takes minutes instead of hours or days. Scaling up means running more instances of the same code.

**Reliability.** Automated processes don't forget steps or make typos. Recovery from failures is faster because rebuilding is automated.

**Collaboration.** Infrastructure changes go through the same review processes as application code. Knowledge is captured in code, not trapped in individuals' heads.

**Compliance.** IaC provides an audit trail of every change. You can prove that systems are configured according to policy because the policy is encoded in version-controlled code.

### Tradeoffs

**Learning curve.** IaC tools have their own languages, concepts, and best practices. Teams need time to build expertise.

**Upfront investment.** Writing good IaC takes longer than making a quick manual change. The payoff comes with scale and over time.

**Complexity.** IaC adds layers of abstraction. When something goes wrong, you need to understand both the infrastructure and the tools managing it.

**State management.** Some IaC tools maintain state files that must be carefully managed. State drift or corruption can cause serious problems.

**Not everything fits.** Some infrastructure components—legacy systems, one-off configurations, vendor appliances—may not fit neatly into IaC models.

## Cultural Implications

IaC isn't just a technical practice; it's a cultural shift. It requires thinking about infrastructure differently:

- **Servers are disposable.** Don't fix a broken server; replace it with a new one built from code.
- **Changes go through process.** No more SSH-ing in to make a quick fix. Changes go through version control and review.
- **Documentation is code.** The authoritative description of your infrastructure is the code that builds it.
- **Everyone can contribute.** When infrastructure is code, developers can propose infrastructure changes using the same tools they use for application code.

This cultural shift can be challenging. Experienced administrators may resist losing direct access to systems. Processes slow down when every change needs a pull request. But organizations that embrace these changes find their infrastructure becomes more reliable, more scalable, and more understandable over time.
