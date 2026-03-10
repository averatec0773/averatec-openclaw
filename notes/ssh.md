# SSH Common Commands

## Connect

```bash
# Basic connection
ssh user@host

# Specify port
ssh -p 2222 user@host

# Specify private key
ssh -i ~/.ssh/id_rsa user@host

# Connect and run a command
ssh user@host "ls -la /var/log"
```

## SSH Tunnels

```bash
# Local port forwarding (access a remote service locally)
ssh -N -L LOCAL_PORT:TARGET_HOST:TARGET_PORT user@JUMP_HOST
# Example: map remote port 18789 to local
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP

# Remote port forwarding (expose a local service to the remote)
ssh -N -R REMOTE_PORT:localhost:LOCAL_PORT user@host

# Dynamic proxy (SOCKS5)
ssh -N -D 1080 user@host
```

## Key Management

```bash
# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "your@email.com"

# Generate RSA key
ssh-keygen -t rsa -b 4096 -C "your@email.com"

# Upload public key to server
ssh-copy-id user@host

# Add public key manually
cat ~/.ssh/id_ed25519.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# List loaded keys
ssh-add -l

# Add key to agent
ssh-add ~/.ssh/id_ed25519
```

## ~/.ssh/config

```
Host vps
    HostName YOUR_VPS_IP
    User root
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host tunnel-openclaw
    HostName YOUR_VPS_IP
    User root
    LocalForward 18789 127.0.0.1:18789
    ServerAliveInterval 60
```

Connect using the alias:
```bash
ssh vps
```

## File Transfer

```bash
# Upload a file
scp local_file.txt user@host:/remote/path/

# Upload a directory
scp -r ./local_dir user@host:/remote/path/

# Download a file
scp user@host:/remote/file.txt ./local/

# Sync with rsync (recommended, supports resume)
rsync -avz ./local/ user@host:/remote/path/
rsync -avz user@host:/remote/path/ ./local/
```

## Debugging

```bash
# Verbose output (debug connection issues)
ssh -v user@host
ssh -vvv user@host   # more detailed

# Test config syntax
ssh -G host

# Check SSH service status on server
systemctl status sshd

# View login logs
tail -f /var/log/auth.log
```

## Keep Connection Alive

Add to `~/.ssh/config`:

```
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```
