# HEX - Grey Hack Automation Tool

> ⚠️ **DISCLAIMER**: This tool is designed exclusively for the video game **Grey Hack** and is intended for use within the game's simulated hacking environment. This is NOT a real hacking tool and should NEVER be used for any malicious or illegal purposes. Grey Hack is a sandbox hacking simulator game where players learn cybersecurity concepts in a safe, virtual environment.

## Overview

HEX is an automation and penetration testing toolkit built for the game [Grey Hack](https://store.steampowered.com/app/605230/Grey_Hack/). It streamlines common in-game hacking tasks, vulnerability scanning, exploitation, and network management by providing a command-line interface with various tools and utilities.

### Key Features

- **Automated vulnerability scanning** - Scan remote and local libraries for exploits
- **One-command exploitation** - Automatically exploit discovered vulnerabilities
- **Context stack management** - Manage multiple shells and computer connections
- **Proxy chain support** - Route attacks through proxy chains for anonymity
- **Remote shell (rshell) management** - Deploy and manage reverse shells on targets
- **Network-wide operations** - Perform actions across entire networks
- **Password cracking** - Rainbow table generation and password decryption
- **File transfer** - Upload and download files between machines
- **Process monitoring** - Built-in htop-style process viewer

---

## Installation

### Prerequisites

- [VS Code](https://code.visualstudio.com/) with the [Greybel VS extension](https://marketplace.visualstudio.com/items?itemName=ayecue.greybel-vs) [Greybel-VS Documentation](https://github.com/ayecue/greybel-vs)

### Compiling with Greybel VS

1. Open this project folder in VS Code
2. Open the main file: `src/hex.src`
3. Use the Greybel VS extension to build:
   - Press `Ctrl+Shift+P` (or `Cmd+Shift+P` on Mac)
   - Type and select: `Greybel: Build`
   - The compiled output will be placed in the `build/` folder
4. Import the compiled script into Grey Hack using the game's code editor
5. Compile and run the script in-game

### Alternative: Manual Build

You can also use the [greybel-js CLI](https://github.com/ayecue/greybel-js) to compile:

```bash
npx greybel-js build src/hex.src --out build/
```

---

## Getting Started

After launching HEX in Grey Hack, you'll be presented with the HEX banner and command prompt. Type `help` to see all available commands.

### Basic Workflow

1. **NMAP a target**: `nmap <ip>` - Discover vulnerabilities on a target IP
2. **Exploit vulnerabilities**: `hack <ip>` - Attempt to exploit discovered vulnerabilities
3. **View connections**: `stack` - See all active shells/connections
4. **Switch context**: `swap <index>` - Switch to a different connection
5. **Explore files**: `tree` - View file system structure
6. **Clean up**: `log` - Corrupt logs to cover tracks

---

## Tools

Tools are the primary hacking commands used for reconnaissance and exploitation.

### `scan`
Scans for vulnerabilities at a specified IP address and port.

```
scan [ip] {port}     - Scan for vulnerabilities at the specified IP/port
scan [library.so]    - Scan a local library file for vulnerabilities
scan -h              - Display help text
```

**Examples:**
- `scan 192.168.1.1` - Scan all open ports on target
- `scan 192.168.1.1 22` - Scan only port 22
- `scan libssh.so` - Scan local library

---

### `nmap`
Displays comprehensive network information including ports, whois data, and firewall rules.

```
nmap {ip}    - Display network information for the specified IP
nmap -h      - Display help text
```

**Output includes:**
- Public IP, kernel version, domain information
- Admin contact, email, phone
- ESSID/BSSID (WiFi info)
- Firewall rules
- All devices and open ports on the network

---

### `hack`
Exploits vulnerabilities discovered by the `scan` tool.

```
hack [-l/local] {libname.so}  - Exploit local library vulnerabilities
hack [-l/local] {lan ip}      - Exploit local libs targeting specific LAN IP
hack [ip] {port}              - Exploit remote vulnerabilities at IP/port
hack [ip] {lan ip}            - Exploit remote vulnerabilities targeting LAN device
hack -h                       - Display help text
```

**Examples:**
- `hack 192.168.1.1` - Exploit all open ports on target
- `hack 192.168.1.1 22` - Exploit only port 22
- `hack -l` - Exploit all libraries in /lib locally

---

### `decipher`
Searches for and decrypts passwords files (mail, bank, passwd).

```
decipher                      - Find and decrypt all passwd files
decipher bank                 - Find and decrypt Bank.txt files
decipher mail                 - Find and decrypt Mail.txt files  
decipher all                  - Decrypt all credential files
decipher [encrypted_password] - Decrypt a specific password hash
decipher -h                   - Display help text
```

---

### `rshell`
Manages reverse shells for persistent access.

```
rshell                              - Collect new rshells and add to stack
rshell infect {ip} {port} {name}    - Deploy rshell client on current target
rshell init {port}                  - Set current context as rshell server
rshell info                         - Display rshell server configuration
rshell start                        - Start rshell server service
rshell stop                         - Stop rshell server service
rshell -r [#]                       - Remove/kill specific rshell by index
rshell -c                           - Clear/kill all rshell processes
rshell -h                           - Display help text
```

---

### `poison`
Uploads vulnerable/insecure libraries to a target for exploitation.

```
poison    - Upload insecure libraries from InsecureLibs folder to target /lib
poison -h - Display help text
```

**Setup:** Create a folder named `InsecureLibs` in your home directory and place vulnerable `.so` files there.

---

### `net`
Performs network-wide operations when connected to a router. If run from a computer or any device that isn't a router, the operations apply only to the current device.

```
net           - Search for interesting files across the network
net -a/all    - Print all interesting files (bank, mail, passwd)
net -b/bank   - Print all bank credentials in the network
net -l/log    - Clear logs on every device in the network
net -m/mail   - Print all mail credentials in the network
net -n/nuke   - Nuke all systems in the network (not implemented)
net -p/pass   - Print all passwd file credentials in the network
net -s/secure - Delete /home, /passwd, and chmod every system (not implemented)
net -u/users  - List all users and credentials (not implemented)
net -z/zday   - List mail accounts with inbox From/Subject details; highlights computers without Mail.txt
net -h        - Display help text
```

**Note:** When run from a shell on a router, operations apply to all devices on the network. When run from a regular computer, operations apply only to that device.

---

## Utilities

Utilities are helper commands for navigation, file management, and system operations.

### `stack`
Displays and manages all active connections/contexts.

```
stack      - Print all contexts in the stack
stack -c   - Clear all contexts except home computer
stack -h   - Display help text
```

The stack shows:
- Index number
- Public IP
- Local IP  
- Context type (shell, computer, file)
- Source (how the connection was obtained)
- Current user

---

### `swap`
Switches between different contexts in the stack.

```
swap {index}  - Switch to context at specified index
swap          - Toggle to previously selected context
swap -h       - Display help text
```

---

### `jump`
Transfers essential libraries to the target for further exploitation.

```
jump          - Transfer metaxploit.so to target
jump apt      - Transfer aptclient.so to target
jump crypto   - Transfer crypto.so to target
jump -h       - Display help text
```

---

### `proxy`
Manages proxy chains for anonymous attacks.

```
proxy {#}       - Set proxy to context at specified index
proxy -s/start  - Connect through proxy chain
proxy -c/clean  - Corrupt logs on all proxies
proxy -d        - Clean proxies and remove from stack
proxy -h        - Display help text
```

---

### `tree`
Displays file system in a tree view with color-coded files.

```
tree {path}  - Display tree from specified path (default: root)
tree -h      - Display help text
```

**Color coding:**
- Green: Important files (passwd, Mail.txt, Bank.txt) with read permission
- Orange: Important files without read permission
- Blue: Regular text files
- Different shades for binaries, libraries, and executables

---

### `log`
Manages log files on target systems.

```
log          - Corrupt log on current context
log -a/all   - Corrupt logs on all contexts in stack
log -d       - Download log file from current context
log -h       - Display help text
```

---

### `terminal`
Opens a native terminal session on the current context.

```
terminal    - Start terminal on current context
terminal -h - Display help text
```

---

### `lockdown`
Secures a system by changing permissions and deleting sensitive files.

```
lockdown    - Remove permissions and delete sensitive files
lockdown -h - Display help text
```

**Actions performed:**
- Changes all file permissions to `u-rwx, g-rwx, o-rwx`
- Preserves essential programs on home systems (Terminal, sudo, Mail, etc.)
- Deletes Mail.txt, Bank.txt, and passwd files

---

### `apt-get`
Manages package installation from in-game repositories.

```
apt-get -i [lib.so] {path}  - Install library at specified path
apt-get -l                  - List all available packages
apt-get -s                  - Show all configured repositories  
apt-get -a {ip}             - Add repository (or random hackshop)
apt-get -r [ip]             - Remove repository
apt-get -h                  - Display help text
```

---

### `download`
Downloads files from the target to your home computer.

```
download [path or filename]  - Download specified file(s)
download -h                  - Display help text
```

**Note:** Supports wildcards (`*`) for pattern matching.

---

### `upload`
Uploads files from your home computer to the target.

```
upload [path or filename]  - Upload specified file(s)
upload -h                  - Display help text
```

---

### `sudo`
Attempts to escalate privileges on the current context.

```
sudo                         - Brute-force root password using rainbow tables
sudo [password]              - Get root shell with specified password
sudo [username] [password]   - Get shell with specified credentials
sudo -h                      - Display help text
```

---

### `rtgen`
Generates rainbow tables for password cracking.

```
rtgen    - Generate rainbow tables using Markov chain password generator
rtgen -h - Display help text
```

---

### `htop`
Displays real-time process and resource monitoring.

```
htop    - Start process monitor (exit with Ctrl+C)
htop -h - Display help text
```

Shows:
- Running processes
- Online users
- CPU utilization bar
- RAM utilization bar
- Process details (user, PID, CPU%, memory%, command)

---

### `hackshop`
Gets a random hackshop IP address for package downloads.

```
hackshop    - Print a random hackshop IP
hackshop -h - Display help text
```

---

### `ip`
Generates a random IP address for target discovery.

```
ip    - Generate random IP address
ip -h - Display help text
```

---

### `restore`
Restores original binaries on NPC systems.

```
restore -s/shop  - Restore original binaries in NPC shop
restore -b/bin   - Restore /bin directory (not implemented)
restore -h       - Display help text
```

---

### `clear`
Clears the screen and displays the HEX banner.

```
clear    - Clear screen and show banner
clear -h - Display help text
```

---

### `quit`
Exits the HEX program.

```
quit    - Exit HEX
quit -h - Display help text
```

---

## File System Commands

### `cd`
Changes the current working directory.

```
cd [folder]  - Change to specified directory
cd ..        - Go up one directory
cd /         - Go to root directory
cd -h        - Display help text
```

---

### `rm`
Removes files and directories without leaving logs.

```
rm [file/pattern]      - Delete file(s) matching pattern
rm -r [folder]         - Recursively delete folder and contents
rm -h                  - Display help text
```

**Note:** Supports wildcards (`*`) for pattern matching.

---

## Project Structure

```
hex-tool/
├── build/                    # Compiled output
│   ├── hex.src               # Main compiled script
│   └── hex/                  # Compiled modules
├── src/                      # Source code
│   ├── hex.src               # Main entry point
│   └── hex/
│       ├── banner.src        # ASCII art banner
│       ├── colors.src        # Color definitions
│       ├── constants.src     # Global constants
│       ├── context.src       # Context management
│       ├── extensions.src    # Type extensions
│       ├── functions.src     # Core utility functions
│       ├── json.src          # JSON parsing
│       ├── logs.src          # Log management
│       ├── mail.src          # Mail handling
│       ├── menu.src          # Menu system
│       ├── nuke.src          # System destruction
│       ├── prompt.src        # Command prompt
│       ├── proxies.src       # Proxy chain management
│       ├── rshell.src        # Reverse shell server
│       ├── tools.src         # Tool imports
│       ├── utilities.src     # Utility imports
│       ├── fs/               # File system commands
│       │   ├── cd.src
│       │   └── rm.src
│       ├── tools/            # Hacking tools
│       │   ├── decipher.src
│       │   ├── hack.src
│       │   ├── missions.src
│       │   ├── net.src
│       │   ├── nmap.src
│       │   ├── nuke.src
│       │   ├── poison.src
│       │   ├── rshell.src
│       │   └── scan.src
│       └── utilities/        # Helper utilities
│           ├── apt-get.src
│           ├── clear.src
│           ├── download.src
│           ├── hackshop.src
│           ├── help.src
│           ├── htop.src
│           ├── ip.src
│           ├── jump.src
│           ├── lockdown.src
│           ├── log.src
│           ├── proxy.src
│           ├── quit.src
│           ├── restore.src
│           ├── rtgen.src
│           ├── stack.src
│           ├── su.src
│           ├── sudo.src
│           ├── swap.src
│           ├── terminal.src
│           ├── tree.src
│           └── upload.src
└── reference/                # Reference scripts and examples
```

---

## Tips for New Users

1. **Start with reconnaissance**: Use `nmap` and `scan` before attempting exploitation
2. **Build rainbow tables early**: Run `rtgen` to generate password tables for faster cracking
3. **Use the stack**: The context stack is powerful - use `stack`, `swap`, and multiple connections
4. **Cover your tracks**: Always run `log` after exploitation to corrupt logs
5. **Set up proxies**: Use `proxy -s` to route attacks through compromised machines
6. **Transfer tools**: Use `jump` to transfer metaxploit to targets for further exploitation

---

## Contributing

Contributions are welcome! Please feel free to submit pull requests or open issues for bugs and feature requests.

---

## License

See [LICENSE](LICENSE) for details.

---

## Author

Created by **redit0**

---

> 🎮 **Remember**: This tool is for educational and entertainment purposes within the Grey Hack game only. Happy (virtual) hacking!
