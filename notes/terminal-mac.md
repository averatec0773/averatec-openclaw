# macOS Terminal Common Commands

## Navigation

```bash
pwd               # show current directory
ls                # list files
ls -la            # list all files with details
cd ~/Desktop      # go to Desktop
cd ..             # go up one level
cd -              # go back to previous directory
```

## Files and Directories

```bash
# Create
mkdir my-folder
mkdir -p a/b/c        # create nested directories at once
touch file.txt        # create empty file

# Copy
cp file.txt backup.txt
cp -r folder/ backup/ # copy directory

# Move / Rename
mv old.txt new.txt
mv file.txt ~/Desktop/

# Delete
rm file.txt
rm -r folder/         # delete directory
rm -rf folder/        # force delete (no confirmation — be careful)

# Show file content
cat file.txt
less file.txt         # scroll through large files (q to quit)
head -20 file.txt     # first 20 lines
tail -20 file.txt     # last 20 lines
tail -f log.txt       # follow file in real time
```

## Search

```bash
# Find files by name
find . -name "*.json"
find ~ -name "config.yml"

# Search text inside files
grep "keyword" file.txt
grep -r "keyword" ./folder    # search recursively
grep -rn "keyword" ./folder   # show line numbers

# Search command history
history | grep docker
```

## Process Management

```bash
# Show running processes
ps aux
ps aux | grep node

# Show real-time resource usage
top
htop              # nicer version (install via brew)

# Kill a process by PID
kill 1234
kill -9 1234      # force kill

# Kill by name
pkill node
```

## Network

```bash
# Check your IP address
ifconfig | grep "inet "
ipconfig getifaddr en0    # Wi-Fi IP
ipconfig getifaddr en1    # Ethernet IP

# Test connection
ping google.com
ping -c 4 google.com      # send only 4 packets

# Check if a port is in use
lsof -i :8080
lsof -i :18789

# Show open network connections
netstat -an | grep LISTEN

# HTTP requests
curl https://example.com
curl -I https://example.com         # headers only
curl -o file.zip https://example.com/file.zip  # download
```

## Homebrew (Package Manager)

```bash
# Install a package
brew install git
brew install node
brew install htop

# Uninstall
brew uninstall package-name

# Update Homebrew and all packages
brew update && brew upgrade

# List installed packages
brew list

# Search for a package
brew search package-name
```

## Environment Variables

```bash
# Show all env vars
env

# Show a specific var
echo $PATH
echo $HOME

# Set temporarily (current session only)
export MY_VAR=hello

# Set permanently — add to ~/.zshrc
echo 'export MY_VAR=hello' >> ~/.zshrc
source ~/.zshrc         # apply changes now
```

## Permissions

```bash
# Show file permissions
ls -la

# Make a file executable
chmod +x script.sh

# Change file owner
chown user:group file.txt
```

## Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Stop running command |
| `Ctrl+Z` | Pause process (to background) |
| `Ctrl+L` | Clear screen |
| `Ctrl+A` | Jump to start of line |
| `Ctrl+E` | Jump to end of line |
| `Ctrl+R` | Search command history |
| `Tab` | Auto-complete |
| `↑` / `↓` | Navigate command history |

## Useful One-Liners

```bash
# Show directory size
du -sh folder/
du -sh *          # size of each item in current directory

# Disk usage
df -h

# Copy text to clipboard
cat file.txt | pbcopy
echo "hello" | pbcopy

# Open current directory in Finder
open .

# Open a file with default app
open file.pdf

# Extract a .tar.gz
tar -xzf file.tar.gz

# Create a .tar.gz
tar -czf archive.tar.gz folder/

# Repeat a command every 2 seconds
watch -n 2 docker ps
```
