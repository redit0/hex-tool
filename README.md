# HEX — Grey Hack Penetration Testing Framework

> **DISCLAIMER**: HEX is designed exclusively for the video game **Grey Hack** and operates entirely within its simulated hacking environment. This is not a real hacking tool and must never be used for any malicious or illegal purpose. Grey Hack is a sandbox hacking simulator where players learn cybersecurity concepts in a safe, virtual environment.

---

## Overview

**HEX** (v0.0.826) is a comprehensive penetration testing and network exploitation framework written in Miniscript for [Grey Hack](https://store.steampowered.com/app/605230/Grey_Hack/). It provides a full command-line interface with tools for vulnerability scanning, exploit execution, credential recovery, reverse shell management, proxy chaining, and network-wide operations — all within the game's simulated environment.

HEX replaces repetitive manual game interactions with a unified, scriptable workflow. Once compiled and running in-game, it acts as your primary hacking console.

---

## Compiling with Greybel-VS

HEX is written in Miniscript source files and must be compiled before it can be run in Grey Hack. The recommended method is the **[greybel-vs](https://marketplace.visualstudio.com/items?itemName=ayecue.greybel-vs)** VS Code extension by ayecue.

### 1. Install the Extension

Open VS Code and install **Greybel VS** from the marketplace:
- Search for `greybel-vs` in the Extensions panel, or
- Install from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=ayecue.greybel-vs)

The extension activates automatically on `.src`, `.gs`, and `.ms` files.

### 2. Open the Project

Open the `hex-tool` folder in VS Code (`File > Open Folder`).

### 3. Build

Open `src/hex.src` in the editor, then run the build command:

- Press `Ctrl+Shift+P` and select **Greybel: Build file from context**, or
- Right-click in the editor and choose **Greybel: Build file from context**

The compiled output is placed in the `build/` folder. HEX uses `import_code` for its module system, so the build produces a set of linked files rather than a single monolithic script.

### Build Type Options

You can set the build type in VS Code settings under `Greybel > Transpiler > Build Type`:

| Mode | Description |
|---|---|
| **Default** | No transformation; best general-purpose output |
| **Uglify** | Minifies code and optimizes namespaces; reduces file size |
| **Beautify** | Formats code for readability |

Hex will not work if the **uglify** method is chosen - It is recommended to just leave the default setting.

### Installer Mode (No Message Hook Required)

If you don't have the Message Hook set up (see below), you can use Greybel's **Installer** mode:

1. Enable `Greybel > Transpiler > Installer > Active` in VS Code settings
2. Optionally enable `Installer > Auto Compile` to have the installer auto-compile and delete source files after running
3. Build the project — Greybel generates one or more installer `.src` files
4. Copy-paste the installer code from each installer file into Grey Hack's code editor, compile it.  Once all files are compiled, run them in order.
5. Your entire project is recreated in-game automatically

If the installer exceeds Grey Hack's ~160,000 character limit, Greybel splits it across multiple files automatically (configurable via `Installer > Max Chars`).

---

## Auto-Compile and Auto-Import into the Game

Greybel-VS supports pushing files directly into a running Grey Hack session using the **Message Hook** — a BepInEx plugin that bridges VS Code and the game client.

### Setting Up the Message Hook

**BepInEx 5.x (recommended):**

1. Download [BepInEx v5.4.23.2](https://github.com/BepInEx/BepInEx/releases/tag/v5.4.23.2) and extract it into the Grey Hack game folder (where `Grey Hack.exe` is located)
2. Download `GreyHackMessageHook5.dll` and place it in the `BepInEx/plugins/` folder
3. Launch Grey Hack via Steam — BepInEx initializes automatically on Windows

**BepInEx 6.x (pre-release, less stable):**

1. Download [BepInEx 6.0.0-pre.2 Unity.Mono](https://github.com/BepInEx/BepInEx/releases/tag/v6.0.0-pre.2) and extract into the game folder
2. Download `GreyHackMessageHook.dll` and place it in `BepInEx/plugins/`
3. Launch Grey Hack via Steam

### Enabling Auto-Import in VS Code

Once the Message Hook is installed and Grey Hack is running, configure Greybel-VS:

| Setting | Description |
|---|---|
| `Create in-game > Active` | Enable automatic file creation in-game after each build |
| `Create in-game > Auto Compile` | After creating files, auto-compile them and delete the `.src` sources |
| `Transpiler > In-game Directory` | Destination folder inside the game (default: `/root/`) |

With **Create in-game > Active** enabled, building the code will automatically import it into the game and save it in the configured directory.

### Manual Import

You can also push files without building first:

- Press `Ctrl+Shift+P` and select **Greybel: Import file into the game**

This uploads the current file directly to the in-game directory defined in `Transpiler > In-game Directory`.

---

## Installation in Grey Hack

After the files are in-game (via any method above):

1. If using the installer, run the installer files in order once to set up all files
2. Move the compiled `hex` binary to `/bin/` for easy access if it's not already set to save there
3. Run `hex` from the terminal

---

## Getting Started

When HEX launches, it displays the banner and command prompt. Type `help` to see all available commands.

### Recommended Workflow

```
1. nmap <target-ip>          — Discover open ports and network layout
2. scan <target-ip>          — Find exploitable vulnerabilities on target
3. hack <target-ip>          — Exploit vulnerabilities and add to context stack
4. swap <stack index>        — Switch context to one of the objects resulting from the exploits
5. decipher                  — Extract and decrypt credentials from target
6. log                       — Corrupt logs to cover your tracks
7. stack                     — Review all active contexts
```

### The Context Stack

HEX maintains a **context stack** — a list of all compromised shells, computers, and file objects. You can hold connections to many systems simultaneously and switch between them with `swap`. The active context is indicated by the prompt (which shows the current IP and user).

---

## Tools

Tools are the primary exploitation and reconnaissance commands.

### `scan`

Scans for exploitable memory addresses in metaxploit libraries on a local or remote target. Results are cached in the vulnerability database for use by `hack`.

```
scan [ip] {port}      Scan all open ports (or a specific port) on a remote target
scan [library.so]     Scan a specific local library file
scan -h               Display help text
```

**Examples:**
```
scan 203.0.113.5
scan 203.0.113.5 22
scan libssh.so
```

---

### `nmap`

Displays comprehensive network information for a target IP. If no IP is given, it prints the saved results of the last nmap.

```
nmap {ip}    Display network information
nmap -h      Display help text
```

**Output includes:** public IP, kernel version, domain, admin contact, email/phone, ESSID/BSSID, firewall rules, all LAN devices and their open/closed ports.

---

### `hack`

Exploits vulnerabilities discovered by `scan` using metaxploit buffer overflows. Successfully exploited targets are added to the context stack as shell, computer, or file objects.

```
hack [-l/local] {libname.so}    Exploit local library vulnerabilities
hack [-l/local] {lan-ip}        Exploit all local libs targeting a specific LAN device with a bounce exploit
hack [ip] {port}                Exploit remote target at specific port
hack [ip] {lan-ip}              Exploit remote target, pivot to LAN device with a bounce exploit
hack -h                         Display help text
```

**Examples:**
```
hack 203.0.113.5
hack 203.0.113.5 22
hack -l
hack -l 192.168.1.100
```

---

### `decipher`

Searches the current context for credential files (`passwd`, `Mail.txt`, `Bank.txt`) and decrypts them using the built-in rainbow table or the crypto library.

```
decipher                        Search and decrypt all passwd files
decipher bank                   Search and decrypt Bank.txt files
decipher mail                   Search and decrypt Mail.txt files
decipher all                    Search and decrypt all credential file types
decipher [encrypted_password]   Decrypt a specific password hash
decipher -h                     Display help text
```

---

### `rshell`

Manages the full reverse shell lifecycle: server setup, client deployment on targets, and polling for new incoming connections.

```
rshell                               Poll the rshell server for new connections; add to stack
rshell init {port}                   Configure the current context as the rshell server
rshell infect {ip} {port} {name}     Deploy the rshell client on the current target
rshell info                          Display current rshell server configuration
rshell start                         Start the rshell server service
rshell stop                          Stop the rshell server service
rshell -r [#]                        Kill the rshell process on a specific rshell by stack index
rshell -c                            Kill all rshell processes
rshell -h                            Display help text
```

**Setup:** Create `/root/SecureLibs/` on your in-game home computer and populate it with `librshell.so` in order to use `rshell start`.

---

### `poison`

Uploads insecure/vulnerable library files from your local `/root/InsecureLibs` folder to the target's `/lib` directory, enabling further exploitation via `scan` and `hack`.

```
poison       Upload insecure libraries to target /lib (will upload all libraries found in /root/InsecureLibs)
poison -h    Display help text
```

**Setup:** Create `/root/InsecureLibs/` on your in-game home computer and populate it with vulnerable `.so` files matching the versions configured in `constants.src`.

---

### `net`

Performs network-wide operations. When run from a shell on a router, operations apply to every device on the network. When run from a non-router device, operations apply only to that device.

```
net                  Scan all systems for interesting files (bank, mail, passwd)
net -a/all           Print all interesting files across the network
net -b/bank          Decipher and print all bank credentials
net -l/log           Corrupt logs on every device in the network
net -m/mail          Decipher and print all mail credentials
net -n/nuke          Hard nuke all systems in the network
net -nm/nukemail     Clear contents of Mail.txt, Bank.txt, and passwd on all systems
net -p/pass          Decipher and print all user/passwd credentials
net -s/secure        Delete sensitive files and chmod every system in the network
net -z/zday          List all mail accounts; print From/Subject of every inbox message; highlight systems missing Mail.txt
net -h               Display help text
```

**Setup:** In order to use this on a router (so that it will work network-wide), you have to create `/root/InsecureLibs/` on your home computer, and populate it with the root bounce library that matches the version and library `insecureComp` type specified in `constants.src`.

---

### `sniffer`

Launches a network traffic sniffer on the current context.

```
sniffer      Start sniffer on current context
sniffer -n   Start sniffer on current context in a new terminal window
sniffer -h   Display help text
```

---

## Utilities

Utilities are helper commands for context management, system operations, and file handling.

### `stack`

Displays all entries in the context stack. The active context is highlighted.

```
stack       Print all contexts (index, public IP, local IP, type, source, user)
stack -c    Clear all contexts except the home computer, proxy servers, and rshell servers
stack -h    Display help text
```

---

### `swap`

Switches between contexts in the stack.

```
swap {index}    Switch to the context at the given index
swap            Toggle to the previously active context
swap -h         Display help text
```

---

### `jump`

Attaches essential libraries (metaxploit, crypto, aptclient) from your home computer to the current context by copying them via SCP.

```
jump           Transfer metaxploit.so to the target and attach it to the context
jump apt       Transfer aptclient.so to the target and attach it to the context
jump crypto    Transfer crypto.so to the target and attach it to the context
jump -h        Display help text
```

---

### `proxy`

Manages proxy chains. When a proxy chain is set, all traffic routes through the configured chain for anonymity.

```
proxy {#}        Set the proxy to the context at the given stack index
proxy -s/start   Connect through the proxy chain
proxy -c/clean   Corrupt logs on all proxy nodes in the chain
proxy -d         Clean proxy logs and remove them from the stack
proxy -h         Display help text
```

Proxies can also be loaded automatically from a `Map.conf` file and from the `$proxies` environment variable (JSON-serialized list).

---

### `log`

Corrupts system logs to erase evidence of activity.

```
log          Corrupt the log on the current context
log -a/all   Corrupt logs on all contexts in the stack
log -d       Download the log file from the current context
log -h       Display help text
```

---

### `tree`

Displays the filesystem in a color-coded tree view.

```
tree {path}    Display file tree from the given path (default: root)
tree -h        Display help text
```

**Color coding:**
- Green — credential files (passwd, Mail.txt, Bank.txt) with read permission
- Orange — credential files without read permission
- Blue — regular text files
- Varying shades for binaries, libraries, and executables

---

### `terminal`

Opens a native interactive terminal session on the current context.

```
terminal      Start a terminal session.  Uses launch(), and falls back to start_terminal if launch() fails.
terminal -h   Display help text
```

---

### `lockdown`

Secures the current system by removing all file permissions and deleting sensitive credential files. Detects home computers (via Xorg process) and preserves essential binaries accordingly.

```
lockdown      Apply lockdown to the current context
lockdown -h   Display help text
```

**Actions:**
- Sets all file permissions to `u-rwx, g-rwx, o-rwx`
- Deletes `Mail.txt`, `Bank.txt`, and `passwd` files
- On home computers, preserves execute permissions on Terminal, sudo, Mail, and similar essential programs

---

### `apt-get`

Manages package repositories and library installation via the in-game aptclient.

```
apt-get -i [lib.so] {path}    Install a library to the specified path
apt-get -l                    List all available packages across repos
apt-get -s                    Show all configured repositories
apt-get -a {ip}               Add a repository (or scan for a random hackshop)
apt-get -r [ip]               Remove a repository
apt-get -h                    Display help text
```

---

### `hackshop`

Discovers a random hackshop IP by scanning port 1542 on a random address.

```
hackshop      Print a random hackshop IP
hackshop -h   Display help text
```

---

### `download`

Downloads files from the current target to your home computer. Supports wildcard patterns.

```
download [path or filename]    Download matching file(s)
download -h                    Display help text
```

---

### `upload`

Uploads files from your home computer to the current target. Supports wildcard patterns.

```
upload [path or filename]    Upload matching file(s)
upload -h                    Display help text
```

---

### `sudo`

Escalates privileges on the current context.

```
sudo                           Brute-force root password using the stored password list
sudo [password]                Get root shell using the given password
sudo [username] [password]     Get a shell as the specified user
sudo -h                        Display help text
```

---

### `rtgen` (deprecated)

Generates additional rainbow table entries using a Markov chain password generator.

```
rtgen      Generate rainbow table entries
rtgen -h   Display help text
```

HEX also ships with a pre-loaded rainbow table (in `src/hex/passwords/`) covering common passwords at startup.  The `rtgen` command has been removed, since it is no longer needed.

---

### `htop`

Displays a real-time process and resource monitor (exit with Ctrl+C).

```
htop      Start process monitor
htop -h   Display help text
```

**Shows:** running processes, online users, CPU utilization bar, RAM utilization bar, process details (user, PID, CPU%, memory%, command).

---

### `restore`

Restores original binaries on NPC systems (e.g., shop default binaries).

```
restore -s/shop    Restore original NPC shop binaries
restore -h         Display help text
```

---

### `ssh`

Connects to a remote host via SSH, adds the resulting shell to the context stack, and swaps to it. Routes through the active proxy chain when one is configured.

```
ssh [user@password] [ip]          Connect via SSH on port 22
ssh [user@password] [ip] [port]   Connect via SSH on a custom port
ssh -h                            Display help text
```

**Examples:**
```
ssh root@toor 203.0.113.5
ssh admin@secret 10.0.0.1 2222
```

---

### `config`

Manages persistent configuration stored in an encrypted file at `/etc/hex.conf`. Settings survive restarts and are protected with AES-128 CBC encryption using a device-derived key. Supports proxy lists, rshell server credentials, and mail credentials.

```
config                                    Display a summary of all saved configuration
config proxy                              List all saved proxies
config proxy add [ip] [user] [pass] {port}   Add a proxy (port defaults to 22)
config proxy remove [#]                   Remove a proxy by its list number
config rshell                             Show the saved rshell server
config rshell set [ip] [user] [pass] {port}  Set the rshell server (port defaults to 1222)
config rshell remove                      Clear the saved rshell server
config mail                               Show the saved mail credentials
config mail set [address] [password]      Set mail credentials
config mail remove                        Clear the saved mail credentials
config -h                                 Display help text
```

---

### `ip`

Generates a random IP address, or scans a /24 network block to locate a specific user's neurobox network by sending probe emails.

```
ip                                Generate and print a random IP
ip [partial_ip] [username]        Scan a block for a user's neurobox network
ip -h                             Display help text
```

**IP completion:** Supply a partial IP with one unknown octet (e.g. `123.123.XXX.123` or omit the last octet entirely) and a username. HEX iterates through all 256 addresses in that block, checks each via `whois` for neurobox networks, and sends a probe email to `username@domain`. A successful send indicates the target network. Requires a mail account to be configured (via `config mail set`).

**Examples:**
```
ip
ip 203.0.113.X admin
ip 10.20.30 jsmith
```

---

### `userdel`

Deletes a user account from the current context.

```
userdel [username]    Delete the specified user
userdel -h            Display help text
```

---

### `clear`

Clears the screen and re-displays the HEX banner.

```
clear      Clear screen and display banner
clear -h   Display help text
```

---

### `quit`

Exits HEX.

```
quit      Exit the program
quit -h   Display help text
```

---

## Filesystem Commands

These commands operate on the current context's filesystem.

### `cat`

Prints the contents of a text file on the current context. Searches by filename across the filesystem; if multiple matches are found, presents a selection prompt. Refuses binary files and directories.

```
cat [file name or path]    Print the contents of a text file
cat -h                     Display help text
```

---

### `cd`

```
cd [folder]    Change to the specified directory
cd ..          Go up one level
cd /           Go to root
cd -h          Display help text
```

### `rm`

Removes files without leaving a log (uses a file-swap technique to avoid triggering the standard `rm` logging). Supports wildcards and recursive deletion.

```
rm [file/pattern]    Delete matching file(s)
rm -r [folder]       Recursively delete a folder and its contents
rm -h                Display help text
```

### `ps`

Lists running processes on the current context.

```
ps      Show running processes
ps -h   Display help text
```

### `kill`

Kills a running process on the current context.

```
kill [pid]    Kill the process with the given PID
kill -h       Display help text
```

### `reboot`

Reboots the current context system.

```
reboot      Reboot current system
reboot -h   Display help text
```

---

## Project Structure

```
hex-tool/
├── build/                        Compiled output (generated by Greybel)
├── src/
│   ├── hex.src                   Main entry point and command loop
│   └── hex/
│       ├── banner.src            ASCII art banner
│       ├── colors.src            Color definitions and formatting functions
│       ├── config.src             Encrypted config persistence (load/save/mask)
│       ├── constants.src         Version, global constants, config values
│       ├── default-binaries.src  Default hackshop binary definitions
│       ├── encryption.src        AES-128 CBC encryption/decryption module
│       ├── extensions.src        Extension methods on built-in types
│       ├── functions.src         Core utility and API functions (~1800 lines)
│       ├── json.src              JSON serialization/deserialization
│       ├── logs.src              Log corruption and file recovery
│       ├── mail.src              Email account management
│       ├── menu.src              Menu generation helpers
│       ├── meta.src              Metaxploit library linking and scanning
│       ├── nuke.src              Hard/soft/mail nuke operations
│       ├── prompt.src            Dynamic prompt generation
│       ├── proxies.src           Proxy chain setup and management
│       ├── rshell.src            Reverse shell server management
│       ├── classes/
│       │   ├── exploit.src       Exploit result object
│       │   └── result.src        Generic function result object
│       ├── fs/                   Filesystem commands
│       │   ├── cat.src
│       │   ├── cd.src
│       │   ├── kill.src
│       │   ├── ps.src
│       │   ├── reboot.src
│       │   └── rm.src
│       ├── passwords/            Pre-loaded rainbow table wordlists
│       │   └── 1-7.src           (comma-separated hash:plaintext entries)
│       ├── tools/                Exploitation tools
│       │   ├── decipher.src
│       │   ├── hack.src
│       │   ├── net.src
│       │   ├── nmap.src
│       │   ├── poison.src
│       │   ├── rshell.src
│       │   ├── scan.src
│       │   └── sniffer.src
│       └── utilities/            Helper utilities
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
│           ├── config.src
│           ├── restore.src
│           ├── rtgen.src
│           ├── ssh.src
│           ├── stack.src
│           ├── sudo.src
│           ├── swap.src
│           ├── terminal.src
│           ├── tree.src
│           ├── upload.src
│           └── userdel.src
└── reference/                    Reference scripts and examples
```

---

## Configuration

### `constants.src`

Edit `src/hex/constants.src` to configure HEX for your game mode:

```python
# Multiplayer (default)
g.const.insecureComp = ["/root/InsecureLibs/init.so", "1.0.0", "0x32E65B93", "indopositionx"]

# Nightly build servers
# g.const.insecureComp = ["/root/InsecureLibs/init.so", "1.0.2", "0x7D6EE022", "stconobjecttransf"]

# Singleplayer
# g.const.insecureComp = ["/root/InsecureLibs/net.so", "1.0.0", "0x18EBA092", "redit0"]
```

Uncomment the block matching your game mode before building.

### Environment Variables

HEX reads two optional environment variables during compilation.  Note that this functionality is only supported with greybel, and these variables must be configured in the greyble-vs settings (transpiler environment variables):

| Variable | Format | Description | file |
|---|---|---|---|
| `#envar proxies` | JSON list of `[ip, port, user, pass]` | Pre-configure proxy chains | In hex/proxies.src |
| `#envar rshell` | JSON object | Pre-configure the rshell server connection | In hex/rshell.src |

### Persistent Configuration

HEX stores persistent settings in `/etc/hex.conf` on your home computer, encrypted with AES-128 CBC using a key derived from the machine's hostname. This file is created automatically when you first save a setting via `config`. Stored settings include:

- **Proxy list** — proxies are loaded on startup and connected automatically
- **Rshell server** — the rshell server is initialized on startup if configured
- **Mail credentials** — mail is logged in on startup if configured

This replaces the need to configure environment variables for proxies and rshell in the Greybel transpiler settings, though those environment variables are still supported as a fallback.

### InsecureLibs Folder

The `poison` command uploads files from `/root/InsecureLibs/` on your home computer. Populate this folder with vulnerable `.so` files to use as poison payloads.

---

## Tips

- **Scan before hacking.** `scan` populates the vulnerability database; `hack` reads from it. Running `hack` on an unscanned target will scan first
- **Use the stack.** HEX can hold many simultaneous connections. Use `stack`, `swap #`, and `swap` (toggle) to move between them quickly.
- **Route through proxies.** Set proxies with `proxy {index}` or activate a chain (either hardcoded or from Map.conf) with `proxy -s` before attacking sensitive targets. Use `proxy -c` to corrupt proxy logs afterward.
- **Jump libraries first.** If you need to scan or exploit from inside a target, use `jump` to copy metaxploit to the target before proceeding.
- **Cover your tracks.** Run `log -a` to corrupt logs across all contexts after a session. The `rm` command also uses a log-safe swap technique to avoid leaving traces.
- **net from the router.** For network-wide operations (`net -n`, `net -l`, `net -z`, etc.) connect to the network's router first — `net` automatically detects router context and targets all LAN devices.

---

## Author

Created by **redit0**

---

> This tool is for educational and entertainment purposes within the Grey Hack game only.
