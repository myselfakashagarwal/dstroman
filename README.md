
![Screenshot 2025-04-29 at 2 20 30 AM](https://github.com/user-attachments/assets/0d550e7f-f619-486f-af26-f9727af3c2f5)


### A CLI + TUI for managing Docker objects with the file system. Perform day-to-day tasks and troubleshooting with ease no copy-pasting IDs or running command sprees, just arrows and coffee to get the work done.  
<br>

# Key Features 
- File transfer between host, container, volume, and bind mounts
- Open any file/directory from host, container, volume, or bind mounts with simple arrow navigation, in your desired console-based editor 
- Take backups of host, container, volume, bind mounts, and their files with ease
<br>

# Dependencies & Supported Platforms
- **Platforms:** GNU/Linux 
- **Arch:** x86_64, amd64
- **Dependencies:** [charmbracelet/gum](https://github.com/charmbracelet/gum), [Docker CLI](https://docs.docker.com/engine/reference/commandline/cli/)
<br>

# DEMO
https://github.com/user-attachments/assets/79beaeb2-f58d-4c91-a133-4b7c373a42e2

<br> 

# Installation 
Copy and paste the command below after installing the dependencies(gum, docker cli) (The command requires curl)
```bash
sudo curl -fsSL https://raw.githubusercontent.com/myselfakashagarwal/dstroman/refs/heads/legacy/dstroman -o /usr/local/bin/dstroman && sudo chmod +x /usr/local/bin/dstroman 
```
<br><br>

# Usage
## Configure the editor for dstroman
- By default, dstroman does not use any console-based text editor; it simply echoes the path that the `open_editor()` function gets.  
- To configure it with your desired editor, edit the file `~/.config/dstroman/editor` and input the command that accepts a file path as the first argument.  
- Make sure the editor command is available for the root user as well.

Example:
- If you want to open files in **neovim** (assuming neovim is installed), the entry in `~/.config/dstroman/editor` would be `neovim`.
- If you want to open files in **vim**, the entry would be `vim`.

## Docker Requirements
- You must have access to Docker without requiring a password. You can achieve this by creating a group named `docker` and adding your user to it.
- The Docker daemon must be running before use.

## Gum Usage Notes
- Press **Enter** to select (if the current highlight is a directory, it will open the directory; if it's a file, it will select the file).
- Use **arrow keys** to navigate (note: bind mount paths are not navigable because they are not part of the container's writable layer — binds are listed separately for navigation).
- Press **Esc** whenever you need to abort an action.

## Other 
- Backups can be found at ~/.config/dstroman/backups/
- Editor config can be found at ~/.config/dstroman/editor

## Options 

for first initialization
```bash
dstroman --init
```

For navigateing objects and their files 
```bash
dstroman --navigate
```

For moving copying file inbetween objects 
```bash
dstroman --transfer
```

To list backups
```bash
dstroman --list-backups
```

For creating backup 
```bash
dstroman --create-backup
```

For removing backup
```bash
dstroman --remove-backup
```

<hr> 

![hackctl](https://github.com/user-attachments/assets/c7347fcf-6d29-4b99-a3c4-749841234ed1)




