# puppet-lint

puppet-lint checks your Puppet code against the official [Puppet style guide](https://www.puppet.com/docs/puppet/latest/style_guide.html). It catches formatting issues, bad practices, and potential errors before they make it into production. Run it before every commit and push.

## Prerequisites

| Tool | Why You Need It |
|------|-----------------|
| [Ruby](ruby.md) | puppet-lint is a Ruby gem |

## Installation

```bash
gem install puppet-lint
```

## Verify Installation

```bash
puppet-lint --version
```

If you get "command not found," your gem bin directory isn't in your `PATH`. See the [Ruby tool guide](ruby.md#verify-gems-are-in-path) for troubleshooting.

## Usage

### Lint a Single File

```bash
puppet-lint manifests/profile/nginx.pp
```

### Lint an Entire Module

Navigate to the root of your Puppet module and run:

```bash
puppet-lint .
```

This recursively checks all `.pp` files in the current directory.

### Example Output

Clean code:

```
$ puppet-lint .
```

No output means no problems.

Code with issues:

```
$ puppet-lint .
manifests/profile/nginx.pp - WARNING: double quoted string containing no variables on line 8
manifests/role/webserver.pp - ERROR: trailing whitespace found on line 12
```

**Warnings** are style issues you should fix. **Errors** are problems that could cause unexpected behavior.

## Before You Commit

Get in the habit of running puppet-lint before every commit:

```bash
puppet-lint .
```

If there are any warnings or errors, fix them before you commit and push. Clean lint output means your code follows the style guide and is less likely to cause surprises in review or on the Puppet server.

!!! warning
    Our CI/CD pipeline runs puppet-lint on every push. If errors are detected, the pipeline will fail and your changes will not be deployed to the Puppet server. Running puppet-lint locally before you push saves you the round-trip of pushing, waiting for CI to fail, fixing, and pushing again.

!!! tip
    If you're using [Visual Studio Code](vscode.md) with the Puppet extension, many lint issues will be highlighted in your editor as you type. puppet-lint on the command line is still worth running as a final check before you push.

## Common Warnings and Fixes

| Warning | Fix |
|---------|-----|
| `double quoted string containing no variables` | Use single quotes for strings that don't need variable interpolation |
| `string containing only a variable` | Use the variable directly instead of wrapping it in a string (`$var` not `"${var}"`) |
| `trailing whitespace found` | Remove whitespace at the end of the line |
| `indentation of => is not properly aligned` | Align `=>` arrows in resource attributes to the same column |
| `class not documented` | Add a comment block above the class definition |
