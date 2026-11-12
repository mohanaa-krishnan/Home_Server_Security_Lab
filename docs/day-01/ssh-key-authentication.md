# SSH Key Authentication and Hardening

## 1. Generate an Ed25519 key pair

On the client machine:

```bash
ssh-keygen -t ed25519 -C "home-server-client"
```

Follow the prompts and use a strong passphrase. The default key paths are:

- Private key: `~/.ssh/id_ed25519`
- Public key: `~/.ssh/id_ed25519.pub`

The private key stays on the client and must never be uploaded to the server or committed to a repository. The public key can be installed on the server.

## 2. Protect the private key

Check the SSH directory and key permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

Do not share the private key, its passphrase, or screenshots that expose them.

## 3. Install the public key on the server

From the client:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub <server_user>@<server_ip>
```

This appends the public key to the server account's `~/.ssh/authorized_keys` file.

Test the login:

```bash
ssh <server_user>@<server_ip>
```

The server account password and the private-key passphrase are different things. The former is used for account authentication methods such as password login; the latter unlocks the private key on the client.

## 4. Understand SSH configuration

Ubuntu commonly reads the main file `/etc/ssh/sshd_config` and configuration snippets under `/etc/ssh/sshd_config.d/`. Confirm the active settings rather than assuming a file's contents are effective.

Example hardening settings:

```text
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitRootLogin no
```

These settings should only be applied after key-based login has been tested successfully.

## 5. Validate before reloading

On the server, validate the configuration:

```bash
sudo sshd -t
```

If validation succeeds, inspect effective settings:

```bash
sudo sshd -T | grep -E '^(pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication|permitrootlogin) '
```

Then reload the service:

```bash
sudo systemctl reload ssh
sudo systemctl status ssh
```

Keep the existing SSH session open. Open a second terminal and verify that a new key-based login succeeds before closing the original session. This reduces the risk of locking yourself out.

## 6. Recovery and troubleshooting

If a local-network SSH connection fails, inspect the route and neighbour table:

```bash
ip route
ip neigh
```

If a stale neighbour entry is suspected and the correct client interface is known:

```bash
sudo ip neigh flush dev <client_interface>
```

Retry the connection after checking that both systems are online and connected to the expected network.

## Security reminders

- Never copy the private key to the server.
- Never commit private keys, passwords, recovery codes, or tokens.
- Do not disable password authentication until key login is confirmed in a separate session.
- Restrict SSH access at the firewall to the networks that need it.
- Recheck effective SSH settings after changing configuration snippets.
