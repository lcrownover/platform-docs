# eyaml

eyaml (encrypted YAML) is a tool for encrypting sensitive values in Hiera data files. It lets you store secrets like passwords and API keys in version control safely. Values are encrypted with a public key that anyone can use, but only the Puppet server (which has the private key) can decrypt them.

## Installation

eyaml is a Ruby gem. Install it using the `gem` command:

```bash
gem install hiera-eyaml
```

If you're using PDK, eyaml may already be available in PDK's Ruby environment.

## Verify Installation

```bash
eyaml version
```

You should see output like `hiera-eyaml version 3.x.x`.

## Configuration

To avoid specifying the public key path every time, create a configuration file at `~/.eyaml/config.yaml`:

```yaml
---
pkcs7_public_key: '/path/to/your/repo/keys/public_key.pkcs7.pem'
```

Replace the path with the actual location of your team's public key. Common locations include:

- `keys/public_key.pkcs7.pem` in your Puppet control repository
- A shared team directory
- A path provided by your team's documentation

After creating this file, eyaml commands will use this key automatically.

## Common Commands

### Encrypt a Password

Encrypt a single-line secret (like a password) interactively:

```bash
eyaml encrypt -p -o string
```

The `-p` flag prompts for input so the secret doesn't appear in your shell history. Type or paste the secret and press Enter. You'll see output like:

```
ENC[PKCS7,MIIBiQYJKoZIhvcNAQcDoIIBejCCAXYCAQAxggEhMIIBHQIBADAF...]
```

Copy the entire `ENC[PKCS7,...]` block (including `ENC[` and `]`) into your Hiera YAML file.

### Encrypt a File

For multi-line secrets like certificates or private keys:

```bash
eyaml encrypt -f /path/to/secret.pem -o block
```

The `-o block` option formats the output for multi-line YAML values.

### Encrypt from stdin

Pipe a value directly:

```bash
echo -n 'my-secret' | eyaml encrypt -o string --stdin
```

The `-n` flag on `echo` prevents a trailing newline from being included in the encrypted value.

### Edit an Encrypted File

Open a Hiera file for editing with encrypted values shown in decryptable form:

```bash
eyaml edit data/secrets.yaml
```

This opens the file in your default editor (`$EDITOR`). Encrypted values appear as `DEC::PKCS7[the secret]!`. Edit them as needed, and eyaml re-encrypts when you save and close.

!!! note
    Full decryption requires the private key (which you likely don't have). Without it, you can still add new encrypted values or replace existing ones using the public key.

### Decrypt a Value

If you have access to the private key (usually only the Puppet server does):

```bash
eyaml decrypt -s 'ENC[PKCS7,MIIBiQ...]'
```

### Validate Encrypted Files

Check that a file's encrypted values are valid:

```bash
eyaml validate data/secrets.yaml
```

## Using Encrypted Values in Hiera

Encrypted values go directly in your YAML files:

```yaml
# data/common.yaml
---
myapp::database_password: ENC[PKCS7,MIIBiQYJKoZIhvc...]
```

For multi-line encrypted values, use YAML's folded style:

```yaml
myapp::ssl_private_key: >
  ENC[PKCS7,MIIBmQYJKoZIhvcNAQcDoIIBijCCAYYCAQAxggEhMIIBHQIBADAF
  MAACAQEwDQYJKoZIhvcNAQEBBQAEggEAh8aP3M6Oq9b5vTtB7Q4n4R8K...]
```

The Puppet server decrypts these values automatically when compiling catalogs. Your Puppet code receives the decrypted string with no special handling required.

## Quick Reference

| Task | Command |
|------|---------|
| Encrypt a password | `eyaml encrypt -p -o string` |
| Encrypt a file | `eyaml encrypt -f file.pem -o block` |
| Edit encrypted file | `eyaml edit data/secrets.yaml` |
| Validate file | `eyaml validate data/secrets.yaml` |
| Show version | `eyaml version` |
