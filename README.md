# Task 1: Linux User Management and SSH Key Authentication on AWS EC2

## 1. Overview

This lab demonstrates basic Linux user administration and secure remote access on an Amazon Linux 2023 EC2 instance.

We created two Linux users (`devuser` and `opsuser`), configured SSH public-key authentication, set up sudo permissions for `opsuser`, checked important SSH file permissions, and tested that a separate, unauthorized SSH key was rejected.

## 2. Objectives

- Create separate Linux accounts for different responsibilities.
- Understand Linux home directories and account verification.
- Configure SSH public-key authentication for each account.
- Apply restrictive permissions to SSH configuration files.
- Grant administrative privileges to `opsuser` using a dedicated sudoers file.
- Verify successful login, sudo access, and rejection of an unapproved SSH key.
- Capture real terminal screenshots as evidence.

## 3. Lab environment

- **Cloud:** AWS
- **Compute:** Amazon EC2
- **Operating system:** Amazon Linux 2023
- **Client:** Windows PowerShell
- **Region used:** `ap-south-1`
- **Users created:** `devuser`, `opsuser`

> The public IP used during the lab was `3.110.115.111`. An EC2 public IP can change after a stop/start unless an Elastic IP is associated. Replace it with the current public IP when repeating the lab.

## 4. What we are proving

1. Both Linux user accounts exist.
2. Each account can authenticate using its configured SSH public key.
3. The SSH directory and `authorized_keys` permissions are appropriately restrictive.
4. `opsuser` can run an administrative command through `sudo`.
5. `devuser` does not have passwordless sudo access under the tested configuration.
6. A separate SSH key that is not authorized on the server is rejected.

## 5. Implementation steps

### Step 1 — Create Linux users

**Run on the EC2 instance** from an account with sudo privileges:

```bash
sudo useradd -m -s /bin/bash devuser
sudo useradd -m -s /bin/bash opsuser
```

**Explanation**

- `sudo`: runs the command with administrative privileges.
- `useradd`: creates a Linux account.
- `-m`: creates the user's home directory.
- `-s /bin/bash`: sets Bash as the login shell.

Verify the accounts:

```bash
id devuser
id opsuser
ls -ld /home/devuser /home/opsuser
```

`id` displays the user's UID, primary group, and group memberships. `ls -ld` shows the home directories and their permissions.

### Step 2 — Generate an SSH key pair on Windows

**Run in Windows PowerShell, not in the EC2 Linux shell:**

```powershell
ssh-keygen -t ed25519 -C "ops-lab"
```

The lab key pair was stored under the Windows user's `.ssh` directory.

- `id_ed25519`: private key. Keep it on the client computer and never commit it to GitHub.
- `id_ed25519.pub`: public key. This is the key that can be installed on the server.

The public key is not a password. The server uses it to check whether a client possesses the matching private key.

### Step 3 — Prepare SSH directories and install the public key

**Run on the EC2 instance.**

Create each user's SSH directory with restrictive permissions:

```bash
sudo install -d -m 700 -o devuser -g devuser /home/devuser/.ssh
sudo install -d -m 700 -o opsuser -g opsuser /home/opsuser/.ssh
```

The public key was temporarily placed on the server at `/tmp/ops-lab.pub`. Install it as each user's `authorized_keys` file:

```bash
sudo install -o devuser -g devuser -m 600 /tmp/ops-lab.pub /home/devuser/.ssh/authorized_keys
sudo install -o opsuser -g opsuser -m 600 /tmp/ops-lab.pub /home/opsuser/.ssh/authorized_keys
sudo rm -f /tmp/ops-lab.pub
```

**Important:** `/tmp/ops-lab.pub` must contain the actual public key generated for this lab before running the installation commands. Do not place the private key on the server.

Verify the configuration:

```bash
sudo ls -ld /home/devuser/.ssh /home/opsuser/.ssh
sudo ls -l /home/devuser/.ssh/authorized_keys
sudo ls -l /home/opsuser/.ssh/authorized_keys
```

Expected permissions:

| Path | Permission | Reason |
|---|---:|---|
| `/home/username/.ssh` | `700` | Only the owner has directory access. |
| `authorized_keys` | `600` | Only the owner can read and write the file. |

The files should also be owned by the matching user.

### Step 4 — Test SSH login from Windows

**Run in Windows PowerShell.** Replace the example IP if the instance's public IP has changed:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" devuser@3.110.115.111
```

Test the second user:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" opsuser@3.110.115.111
```

After connecting, run these commands **inside the EC2 Linux shell**:

```bash
whoami
hostname
id
```

`whoami` should display the account used for the login. `hostname` shows the server name, and `id` displays account and group details.

### Step 5 — Configure sudo access for `opsuser`

**Run on EC2** from an account that already has administrative access:

```bash
sudo visudo -f /etc/sudoers.d/opsuser
```

Add this line to the file:

```text
opsuser ALL=(ALL) ALL
```

Save and exit the editor. Validate the file:

```bash
sudo visudo -cf /etc/sudoers.d/opsuser
sudo chown root:root /etc/sudoers.d/opsuser
sudo chmod 440 /etc/sudoers.d/opsuser
sudo ls -l /etc/sudoers.d/opsuser
sudo cat /etc/sudoers.d/opsuser
```

**Meaning of the rule**

- `opsuser`: the user receiving the permission.
- `ALL=(ALL)`: permits commands to be run as any target user, subject to sudo policy.
- Final `ALL`: applies to all commands.

This is broad administrative access. In production, grant only the permissions needed for the role. Use `visudo` to edit sudoers files because it validates syntax.

### Step 6 — Verify sudo behavior

**Log in as `opsuser` and run inside EC2:**

```bash
whoami
sudo -v
sudo id
```

Enter the `opsuser` password when prompted. Successful `sudo id` output should include `uid=0(root)`.

**Test `devuser`:**

```bash
sudo -n id
```

In this lab, the command returned `sudo: a password is required`. This shows `devuser` could not run that command without a password. It does not, by itself, prove that `devuser` has no sudo privileges at all; the effective sudo policy must be checked if that distinction matters.

### Step 7 — Test rejection of an unauthorized SSH key

Generate a separate test key **in Windows PowerShell**:

```powershell
ssh-keygen -t ed25519 -f "$env:TEMP\unauthorized-test" -C "negative-test"
```

Test `devuser` using only that key and public-key authentication:

```powershell
ssh -o IdentitiesOnly=yes -o PreferredAuthentications=publickey -o PasswordAuthentication=no -i "$env:TEMP\unauthorized-test" devuser@3.110.115.111
```

Repeat with `opsuser` in place of `devuser`.

Expected result:

```text
Permission denied (publickey,...)
```

This test is meaningful when the temporary key's public key has not been added to the target account's `authorized_keys` file. The SSH options restrict the test to the supplied identity and public-key authentication, rather than allowing a password fallback.

Remove the temporary test key files from Windows PowerShell after testing:

```powershell
Remove-Item "$env:TEMP\unauthorized-test","$env:TEMP\unauthorized-test.pub"
```

Do not delete the original `id_ed25519` private key needed for your normal lab login.

## 6. Troubleshooting reference

| Symptom | Checks |
|---|---|
| SSH connection times out | Verify the EC2 instance is running, the public IP is current, routing is correct, and the security group permits TCP 22 from your trusted client IP. |
| `Permission denied (publickey)` | Verify the username, private-key path, public key in the correct `authorized_keys`, ownership, and file permissions. |
| SSH complains about permissions | Check `.ssh` ownership/mode (`700`) and `authorized_keys` ownership/mode (`600`). |
| `sudo` asks for a password | Confirm which user is logged in and inspect the effective sudoers policy. A password prompt is not necessarily an error. |
| `visudo` reports a syntax error | Reopen the file with `sudo visudo -f /etc/sudoers.d/opsuser` and correct the reported line. |
| `id username` says the user does not exist | Check the username spelling and whether account creation succeeded. |

## 7. Evidence screenshots

Save real terminal screenshots in this folder. Do not use mock or generated screenshots.

| Screenshot | Evidence to capture |
|---|---|
| `screenshots/01-user-accounts.png` | `id devuser`, `id opsuser`, and home directory listing. |
| `screenshots/02-ssh-permissions.png` | SSH directory and `authorized_keys` ownership and permissions. |
| `screenshots/03-sudo-validation.png` | `visudo` validation, sudoers file permissions, and rule. |
| `screenshots/04-opsuser-sudo-access.png` | `whoami` and successful `sudo id` as `opsuser`. |
| `screenshots/05-unauthorized-key-denied.png` | Both rejected temporary-key attempts from Windows PowerShell. |

Screenshots should avoid exposing private keys, passwords, tokens, or other sensitive information.

## 8. Security notes

- Never upload private keys such as `id_ed25519` to GitHub.
- Do not share passwords or include them in documentation or screenshots.
- Restrict inbound SSH (TCP 22) to a trusted source IP rather than opening it to the whole internet.
- Use individual accounts instead of sharing a common administrator login.
- Grant sudo access according to least privilege.
- Remove temporary test keys after the lab.

## 9. Interview explanation

**Question: What did you implement in this task?**

I created separate Linux users on an Amazon Linux EC2 instance, configured SSH public-key authentication for both accounts, checked SSH file ownership and permissions, and granted sudo access to the operations user through a dedicated sudoers file. I verified login and administrative access, then tested that a separate unauthorized SSH key was rejected.

**Question: Why use SSH keys?**

SSH keys provide a way to authenticate a client without sending a reusable account password over the connection. The private key stays with the client, while the corresponding public key is authorized on the server.

**Question: Why use `visudo`?**

`visudo` helps safely edit sudoers configuration and validates syntax, reducing the risk of a malformed rule preventing intended sudo access.

**Question: What is least privilege?**

It means granting only the access a user needs to perform their job, instead of giving every account full administrator access.

## 10. Lab completion checklist

- [ ] `devuser` and `opsuser` exist.
- [ ] Both users have home directories.
- [ ] Public-key SSH login works for both users.
- [ ] SSH file permissions and ownership are verified.
- [ ] The `opsuser` sudoers file validates successfully.
- [ ] `opsuser` can execute an administrative command with sudo.
- [ ] Unauthorized-key rejection is demonstrated for both users.
- [ ] Actual evidence screenshots are saved in `screenshots/`.
- [ ] No private keys or passwords are included in the repository.
