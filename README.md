# dstroman

dstroman is a CLI and TUI for navigating, transferring, and backing up files across Docker objects: containers, volumes, bind mounts, and the host itself, without hand copying container IDs or writing one off `docker cp` and `docker inspect` commands. Every action is driven through `gum` prompts, so the entire workflow is arrow keys and selections rather than memorized syntax.

## Design Philosophy

The object is the abstraction. Containers, volumes, bind mounts, and the host filesystem are all treated as instances of one concept: a thing with files that can be listed, selected, opened, transferred, or backed up. Every top level command (`navigate`, `transfer`, `create-backup`) starts by asking which kind of object the operator wants, then dispatches to the same handful of functions regardless of which one was picked.

The writable layer is resolved, not assumed. A running container does not expose its actual filesystem location directly, so selecting a container's files means finding where Docker actually put them on the host. The script tries this in order: `GraphDriver.Data.MergedDir` first (the standard overlay2 merged view), then `UpperDir` if that's empty, then a direct scan of `/var/lib/docker/overlay2` for a `merged` directory if neither field resolves, and finally a `rootfs/overlayfs/<container id>` path as a last resort. Only once one of these actually resolves does navigation into the container's files proceed.

The editor is pluggable, not built in. dstroman does not embed an editor. `open_editor` simply runs whatever command is written in `~/.config/dstroman/editor` against the selected path. By default that file just echoes the path back, and the operator is expected to replace it with `vim`, `nvim`, or whatever console editor they actually use, run as root so it can reach root owned container and volume paths.

Nothing proceeds on an empty selection. Every `gum choose` or `gum file` result is passed through `validate_empty` before being acted on, so backing out of a picker (or a picker returning nothing because there was nothing to pick) halts the operation immediately rather than running a transfer or backup against an empty path.

## How it works

### Bootstrap
`dstroman --init` detects the package manager (`/etc/yum` means yum, `/etc/apt` means apt), then checks for `wget`, `gum`, `which`, and Docker one at a time, prompting before installing anything that's missing. `gum` is fetched as a pinned `x86_64` tarball directly from its GitHub release and installed to `/usr/local/bin`; Docker is installed through the native package manager, added to a `docker` group, and enabled and started as a service. Every install step's output is redirected into `~/dstroman.log` so failures can be inspected without cluttering the terminal. Init also creates `~/.config/dstroman/backups` and a default `~/.config/dstroman/editor` file if either is missing.

### Discovering objects
- **Containers** come from `docker ps`, and a container's files are found through the writable layer resolution chain above.
- **Volumes** come from `docker volume ls`, resolved to their `Mountpoint` via `docker inspect`.
- **Bind mounts** come from inspecting every running container's `Mounts` and listing each one's `Source` path, since binds aren't part of any single container's writable layer and have to be surfaced separately.
- **Host** paths are just whatever directory the operator picks directly through `gum file`.

Each of these has a matching existence check (`is_exists_container`, `is_exists_volume`, `is_exists_bind`) that warns and exits early if there's nothing of that type to select from, rather than handing an empty picker to the operator.

### Backups
A backup is just `cp -r` from a resolved object or object file path into `~/.config/dstroman/backups/<name>`, where `<name>` is validated to be alphanumeric before the copy runs. Listing backups reads that same directory; removing one prompts for which backup to delete via `gum choose` and removes it.

### Transfers
A transfer asks for Copy or Move, resolves a source and a destination the same way navigation resolves objects (through `select_object_file`), and runs `cp -r` or `mv` between them as root.

## Usage

```
dstroman --init              Install dependencies and initialize configuration
dstroman --navigate          Browse and open a file from a container, volume, bind mount, or host path
dstroman --transfer          Copy or move a file between two objects
dstroman --list-backups      List all backups
dstroman --create-backup     Back up an object or a file within an object
dstroman --remove-backup     Delete a backup
```

## Dependencies and Supported Platforms
- **Platform**: GNU/Linux
- **Arch**: x86_64, amd64
- **Dependencies**: [gum](https://github.com/charmbracelet/gum), the Docker CLI

## Installation
```bash
sudo curl -fsSL https://raw.githubusercontent.com/myselfakashagarwal/dstroman/refs/heads/legacy/dstroman -o /usr/local/bin/dstroman && sudo chmod +x /usr/local/bin/dstroman
```

## Configuration
- **Editor**: edit `~/.config/dstroman/editor` and put in a command that accepts a file path as its first argument (for example `nvim` or `vim`). Make sure that command is available to root as well, since dstroman opens files with `sudo`.
- **Docker access**: the operator's user should be in the `docker` group so Docker commands run without a password prompt, and the Docker daemon needs to be running before use.
- **Backups**: stored at `~/.config/dstroman/backups/`.

## Gum Usage Notes
Enter selects (opening a directory or picking a file); arrow keys navigate. Bind mount paths are not directly browsable the way container and volume paths are, since they sit outside the container's writable layer, so binds are listed and selected separately rather than traversed. Esc aborts an action at any point.
