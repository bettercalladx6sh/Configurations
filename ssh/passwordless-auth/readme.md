# SSH Key Setup

Simple guide to use a personal SSH key with an EC2 server.

## 1. Check Your SSH Key

```bash
ls ~/.ssh/
```

For example:

```text
id_ed25519
id_ed25519.pub
```

- `id_ed25519` → Private key
- `id_ed25519.pub` → Public key

---

## 2. Test the EC2 `.pem` Key

Use the `.pem` key to access the server:

```bash
ssh -i ~/Downloads/ssh.pem ubuntu@13.204.69.205
```

If you can log in, the `.pem` key is working.

---

## 3. Copy Your Public Key

Use `ssh-copy-id`:

```bash
ssh-copy-id -f -i ~/.ssh/id_ed25519.pub -o IdentityFile=~/Downloads/ssh.pem ubuntu@13.204.69.205
```

This copies:

```text
~/.ssh/id_ed25519.pub
```

to the server's:

```text
~/.ssh/authorized_keys
```

---

## 4. Connect Without `.pem`

After the key is copied:

```bash
ssh ubuntu@13.204.69.205
```

Your SSH client can now use:

```text
~/.ssh/id_ed25519
```

---

## Why Did `Permission denied (publickey)` Happen?

If you run:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@13.204.69.205
```

the server may reject you because it doesn't recognize your `id_ed25519` key yet.

You need to use the existing `.pem` key to get access first:

```bash
-o IdentityFile=~/Downloads/ssh.pem
```

### Remember

```text
ssh.pem
   ↓
Used to access the EC2 server initially

id_ed25519.pub
   ↓
Public key copied to the server

id_ed25519
   ↓
Your private key used for future SSH connections
```
