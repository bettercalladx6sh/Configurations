# SSH Key-Based Passwordless Authentication

This guide explains **why SSH keys are used, how they work, and how to configure passwordless SSH authentication**.

---

## What is SSH Key Authentication?

SSH normally allows you to connect to a server using a password:

```bash
ssh username@SERVER_IP
```

Instead of entering a password every time, we can use an **SSH key pair**.

An SSH key pair contains:

- **Private key** → stays on your local machine 🔐
- **Public key** → copied to the remote server

The server uses your public key to verify that you have the matching private key.

---

# 1. Check Existing SSH Keys

```bash
ls -la ~/.ssh
```

### What does it do?

Lists the files inside your SSH directory.

You may already have:

```text
id_ed25519
id_ed25519.pub
```

If you already have a key you want to use, you don't necessarily need to generate another one.

---

# 2. Generate an SSH Key Pair

```bash
ssh-keygen -t ed25519
```

### Why do we use it?

`ssh-keygen` creates a new SSH key pair.

### What does `-t ed25519` mean?

It tells `ssh-keygen` to create an **Ed25519 type key**.

Ed25519 is a modern SSH key algorithm that provides strong security with relatively small keys.

You will be asked:

```text
Enter file in which to save the key:
```

Press **Enter** to use the default:

```text
~/.ssh/id_ed25519
```

Then you will be asked for a passphrase.

A passphrase protects your private key if someone gets access to the key file.

---

# 3. Understand the Generated Files

After running `ssh-keygen`, you get:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

### Private Key

```text
~/.ssh/id_ed25519
```

This is your **private key**.

🔐 Never share it or upload it to GitHub.

### Public Key

```text
~/.ssh/id_ed25519.pub
```

This is your **public key**.

It is safe to copy to servers.

---

# 4. View Your Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

### Why use it?

Displays your public key.

It will look similar to:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... user@machine
```

This is the key that will be installed on the remote server.

---

# 5. Copy the Public Key to the Server

```bash
ssh-copy-id username@SERVER_IP
```

Example:

```bash
ssh-copy-id ubuntu@3.110.221.43
```

### Why do we use it?

It copies your public key to the remote server.

It automatically adds the key to:

```text
~/.ssh/authorized_keys
```

on the server.

You normally need to enter the server password **once** while doing this.

---

# 6. What is `authorized_keys`?

On the server:

```bash
~/.ssh/authorized_keys
```

contains the public keys that are allowed to authenticate as that user.

For example:

```text
ssh-ed25519 AAAAC3... user@machine
```

Think of it as an **allow list of SSH public keys**.

If your public key is present there, the server can recognize your corresponding private key during authentication.

---

# 7. Test the SSH Connection

```bash
ssh username@SERVER_IP
```

Example:

```bash
ssh ubuntu@3.110.221.43
```

### What happens?

Your SSH client uses your private key to prove that you own the key corresponding to the public key stored on the server.

If everything is configured correctly, you can log in without entering the server account password.

---

# 8. How SSH Key Authentication Works

The basic flow is:

```text
                LOCAL MACHINE
              ┌───────────────┐
              │ Private Key 🔐│
              └───────┬───────┘
                      │
                      │ SSH connection
                      ▼
              ┌───────────────┐
              │ Remote Server │
              │               │
              │ Public Key 🔓 │
              │       ↓       │
              │authorized_keys│
              └───────────────┘
```

The important point:

> **The private key does not get copied to the server.**

The server only needs your public key.

During authentication, SSH performs a cryptographic verification to prove that your private key matches the public key authorized on the server.

---

# 9. Check Authorized Keys on the Server

After logging into the server:

```bash
cat ~/.ssh/authorized_keys
```

### Why use it?

To check which public keys are currently allowed to access that account.

---

# 10. Set Correct SSH Permissions

On the server:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### Why do we use it?

SSH expects sensitive SSH files and directories to have secure permissions.

`700` means:

```text
Owner: read + write + execute
Others: no access
```

`600` means:

```text
Owner: read + write
Others: no access
```

Incorrect permissions can cause SSH authentication to fail.

---

# 11. Connect Using a Specific Key

```bash
ssh -i ~/.ssh/id_ed25519 username@SERVER_IP
```

### Why use `-i`?

`-i` tells SSH which private key file to use.

This is useful when you have multiple SSH keys.

Example:

```bash
ssh -i ~/.ssh/production_key ubuntu@3.110.221.43
```

---

# 12. Generate a Key With a Custom Name

```bash
ssh-keygen -t ed25519 -f ~/.ssh/production_key
```

### What does `-f` do?

`-f` specifies the filename and location for the generated key.

This creates:

```text
~/.ssh/production_key
~/.ssh/production_key.pub
```

You can then copy the public key:

```bash
ssh-copy-id -i ~/.ssh/production_key.pub username@SERVER_IP
```

And connect:

```bash
ssh -i ~/.ssh/production_key username@SERVER_IP
```

---

# 13. SSH Agent

If your private key has a passphrase, you can use `ssh-agent`.

Start the agent:

```bash
eval "$(ssh-agent -s)"
```

### What does it do?

Starts an SSH authentication agent in the background.

The agent can hold your unlocked private key temporarily.

---

# 14. Add Your Key to SSH Agent

```bash
ssh-add ~/.ssh/id_ed25519
```

### Why use it?

Loads your private key into the SSH agent.

This means you don't have to repeatedly type the key's passphrase during your session.

Check loaded keys:

```bash
ssh-add -l
```

---

# 15. Debug SSH Authentication

If SSH doesn't work:

```bash
ssh -v username@SERVER_IP
```

For more detailed information:

```bash
ssh -vvv username@SERVER_IP
```

### Why use it?

Verbose mode helps you understand what SSH is doing.

It can show:

- Which keys SSH is trying
- Whether a key is being offered
- Authentication failures
- Permission problems
- SSH configuration issues

---

# 16. Common Error

You may see:

```text
Permission denied (publickey).
```

Check:

```bash
ls -la ~/.ssh
```

On the server:

```bash
ls -la ~/.ssh
cat ~/.ssh/authorized_keys
```

Check permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Also verify:

- Correct username
- Correct server IP
- Correct public key in `authorized_keys`
- Correct private key on your local machine
- Correct SSH permissions

---

# 17. Complete Setup

### On your local machine

Generate the key:

```bash
ssh-keygen -t ed25519
```

Copy the public key:

```bash
ssh-copy-id username@SERVER_IP
```

Connect:

```bash
ssh username@SERVER_IP
```

That's it.

---

# 18. Quick Command Reference

| Command                            | Purpose                     |
| ---------------------------------- | --------------------------- |
| `ssh-keygen -t ed25519`            | Generate an SSH key pair    |
| `ls -la ~/.ssh`                    | List SSH files              |
| `cat ~/.ssh/id_ed25519.pub`        | View public key             |
| `ssh-copy-id user@server`          | Copy public key to server   |
| `ssh user@server`                  | Connect to server           |
| `cat ~/.ssh/authorized_keys`       | View authorized public keys |
| `chmod 700 ~/.ssh`                 | Secure SSH directory        |
| `chmod 600 ~/.ssh/authorized_keys` | Secure authorized keys file |
| `ssh -i key user@server`           | Use a specific private key  |
| `ssh-add key`                      | Add key to SSH agent        |
| `ssh-add -l`                       | List agent keys             |
| `ssh -v user@server`               | Debug SSH                   |
| `ssh -vvv user@server`             | Detailed SSH debugging      |

---

# 19. The Whole Process

```text
1. ssh-keygen
       ↓
Generate private + public key
       ↓
2. ssh-copy-id
       ↓
Copy public key to server
       ↓
3. Public key is stored in
   ~/.ssh/authorized_keys
       ↓
4. ssh user@server
       ↓
SSH verifies your private key
       ↓
5. Authentication succeeds
       ↓
Passwordless SSH login
```

## 🔐 Important Security Rules

**Never share your private key:**

```text
~/.ssh/id_ed25519
```

**Public key can be shared:**

```text
~/.ssh/id_ed25519.pub
```

**Never commit private keys to GitHub.**

The key concept to remember:

> **Private key stays with you. Public key goes to the server. The server uses the public key to verify that you possess the matching private key.**
