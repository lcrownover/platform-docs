# Puppet Development Kit (PDK)

!!! warning
    PDK versions available for download may be outdated or unavailable for some platforms, especially macOS. Unfortunately, there is not currently a workaround.

PDK is Puppet's official toolkit for developing and testing Puppet modules. It provides a consistent environment for creating modules, writing unit tests, validating syntax, and packaging modules for publication. PDK also includes a language server that integrates with editors like VS Code, giving you autocompletion, real-time validation, and inline documentation as you write Puppet code. If you're writing Puppet code, PDK standardizes your workflow and catches errors before they hit production.

## Puppet Forge Account

You need a Puppet Forge account to download PDK. The download requires accepting Puppet's EULA, which is tied to your Forge account. You may also use this account later to publish modules and generate API keys.

[Click here for the official instructions](https://help.puppet.com/core//current/Content/PuppetCore/access_core_for_test_dev.htm), or proceed below for the abridged version.

### Creating an Account

1. Go to [forge.puppet.com](https://forge.puppet.com/)
2. Click **Sign Up** in the top right
3. Create an account using your work email
4. Verify your email address

### Forge API Key

An API key lets you authenticate with the Forge from the command line which is required for downloading the PDK and publishing modules.

1. Log in to [forge.puppet.com](https://forge.puppet.com/)
2. Click your username in the top right → **View Profile**
3. Select **API keys** from the sidebar
4. Click **Create a new key**
5. Give the key a descriptive name (e.g., "workstation-pdk")
6. Select a validity. I recommend 365 days if this is your primary key.
7. Copy the key immediately, you won't be able to see it again!

Store the key securely. You'll use it when downloading the PDK or publishing modules.

## Installation

Browse to the PDK downloads page: [https://forge.puppet.com/resources/pdk](https://forge.puppet.com/resources/pdk)

For each platform, you'll be prompted for authentication to the site.

**The Username will be `forge-key` and the password is the API key you previously generated.**

## Verify Installation

Using your terminal of choice, run the following command:

```bash
pdk --version
```

You should see output like `3.6.x`.

## Editor Integration

PDK includes a language server that provides intelligent editing features when working with Puppet code. Editors like VS Code can connect to this language server to offer:

- **Autocompletion:** suggests resource types, parameters, and variables as you type
- **Hover documentation:** displays inline help for resources, functions, and parameters
- **Real-time validation:** highlights syntax errors and style issues without leaving your editor
- **Go to definition:** jump to class and defined type declarations across your module
- **Code snippets:** insert boilerplate for common patterns like resource declarations

This feedback loop catches mistakes early, before you run `pdk validate` or push code. Instead of writing Puppet manifests blind and discovering problems later, you get immediate guidance on correct syntax, available parameters, and type mismatches.

### VS Code Setup

Install the [Puppet extension](https://marketplace.visualstudio.com/items?itemName=puppet.puppet-vscode) from the VS Code marketplace. The extension automatically detects PDK and uses it for validation and language features.

After installation:

1. Open a PDK-generated module folder in VS Code
2. Look for "Puppet" in the status bar (bottom right) to confirm the language server is running
3. Open any `.pp` file to see autocompletion and validation in action

!!! tip
    The extension works best when you open the module root folder (the one containing `metadata.json`) rather than individual files or parent directories.
