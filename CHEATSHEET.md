# Task 1 Cheat Sheet — Linux Users and SSH Keys

## Goal

On an Amazon Linux EC2 instance, create two Linux users, configure SSH key authentication, give `opsuser` sudo access, and test that an unapproved SSH key is rejected.

## Know where to run commands

- **Windows PowerShell:** prompt looks like `PS D:\...>`
- **EC2 Linux shell:** prompt looks like `[ec2-user@...]$`, `[devuser@...]$`, or `[opsuser@...]$`

Do not run PowerShell commands in the Linux shell or Linux commands in PowerShell.

## 1. Create users — run on EC2

```bash
sudo useradd -m -s /bin/bash devuser
sudo useradd -m -s /bin/bash opsuser
id devuser
id opsuser
ls -ld /home/devuser /home/opsuser
```

- `useradd`: creates a user.
- `-m`: creates a home directory.
- `-s /bin/bash`: sets the login shell.
- `id`: shows UID, GID, and groups.

## 2. Generate SSH keys — run in Windows PowerShell

```powershell
ssh-keygen -t ed25519 -C "ops-lab"
```

- Private key: `id_ed25519` — keep it secret on Windows.
- Public key: `id_ed25519.pub` — install this on EC2.
- Never upload the private key to GitHub.

## 3. Configure SSH directories and keys — run on EC2

```bash
sudo install -d -m 700 -o devuser -g devuser /home/devuser/.ssh
sudo install -d -m 700 -o opsuser -g opsuser /home/opsuser/.ssh
```

After the public key is safely placed at `/tmp/ops-lab.pub`:

```bash
sudo install -o devuser -g devuser -m 600 /tmp/ops-lab.pub /home/devuser/.ssh/authorized_keys
sudo install -o opsuser -g opsuser -m 600 /tmp/ops-lab.pub /home/opsuser/.ssh/authorized_keys
sudo rm -f /tmp/ops-lab.pub
```

Check:

```bash
sudo ls -ld /home/devuser/.ssh /home/opsuser/.ssh
sudo ls -l /home/devuser/.ssh/authorized_keys
sudo ls -l /home/opsuser/.ssh/authorized_keys
```

**Permissions to remember**

- `.ssh` directory: `700`
- `authorized_keys`: `600`
- Files should be owned by the matching user.

## 4. Test SSH login — run in Windows PowerShell

Replace the IP if your EC2 public IP has changed.

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" devuser@3.110.115.111
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" opsuser@3.110.115.111
```

After logging in, run on EC2:

```bash
whoami
hostname
id
```

- `whoami`: current username.
- `hostname`: server name.
- `id`: user and group details.

## 5. Give `opsuser` sudo access — run on EC2

```bash
sudo visudo -f /etc/sudoers.d/opsuser
```

Add:

```text
opsuser ALL=(ALL) ALL
```

Validate and set secure ownership/permissions:

```bash
sudo visudo -cf /etc/sudoers.d/opsuser
sudo chown root:root /etc/sudoers.d/opsuser
sudo chmod 440 /etc/sudoers.d/opsuser
sudo ls -l /etc/sudoers.d/opsuser
```

- `visudo`: safely edits and validates sudoers configuration.
- `sudo id` should show `uid=0(root)` when run successfully as `opsuser`.
- `chmod 440`: owner and group can read; no write permission is granted.

## 6. Verify sudo

As `opsuser` on EC2:

```bash
whoami
sudo -v
sudo id
```

As `devuser`:

```bash
sudo -n id
```

In this lab, `devuser` received `sudo: a password is required`. This proves the command could not run without a password; it does not alone prove that `devuser` has no sudo rights at all.

## 7. Test an unauthorized key — run in Windows PowerShell

Create a separate test key:

```powershell
ssh-keygen -t ed25519 -f "$env:TEMP\unauthorized-test" -C "negative-test"
```

Test `devuser`:

```powershell
ssh -o IdentitiesOnly=yes -o PreferredAuthentications=publickey -o PasswordAuthentication=no -i "$env:TEMP\unauthorized-test" devuser@3.110.115.111
```

Repeat with `opsuser` instead of `devuser`.

Expected: `Permission denied (publickey,...)`, provided the test public key was never authorized on the server.

Remove the temporary key files:

```powershell
Remove-Item "$env:TEMP\unauthorized-test","$env:TEMP\unauthorized-test.pub"
```

## Troubleshooting in one minute

| Problem | First check |
|---|---|
| SSH timeout | Instance state, current public IP, TCP 22 security-group rule, routing. |
| `Permission denied (publickey)` | Username, private-key path, `authorized_keys`, ownership and permissions. |
| Sudo password prompt | Check the current user and effective sudoers policy. |
| Sudoers syntax error | Run `sudo visudo -cf /etc/sudoers.d/opsuser`. |
| User missing | Run `id username` and verify account creation. |

## Interview answer

“I created separate Linux accounts on AWS EC2, configured SSH public-key authentication, verified SSH file permissions, granted sudo access to `opsuser` through a dedicated sudoers file, and tested that an unauthorized SSH key was rejected. This demonstrates Linux user administration and basic access control.”

## Screenshot checklist

Save actual terminal screenshots in `screenshots/`:

1. `01-user-accounts.png`
2. `02-ssh-permissions.png`
3. `03-sudo-validation.png`
4. `04-opsuser-sudo-access.png`
5. `05-unauthorized-key-denied.png`

Do not include private keys, passwords, or other secrets in screenshots or GitHub.
