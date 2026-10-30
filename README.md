# loadout

A single static binary that rebuilds a full offensive-security toolbox under
`/tools` after an OS re-image — it installs ~40 pentest tools, wires them onto
your `$PATH`, and verifies the result. One command takes a bare Linux box to a
ready toolbox.

> [!IMPORTANT]
> For **authorized** penetration testing, red-team engagements, and CTF/HTB
> practice only. Use it only against systems you are permitted to test.

**Platform:** Debian / Ubuntu, `linux/amd64` · **Idempotent:** safe to re-run.

## Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Quick start](#quick-start)
- [Commands](#commands)
- [Flags](#flags)
- [Installing specific tools](#installing-specific-tools)
- [How it works](#how-it-works)
- [Output and logging](#output-and-logging)
- [Environment variables](#environment-variables)
- [Tools installed](#tools-installed)
- [Troubleshooting](#troubleshooting)

## Requirements

- Debian or Ubuntu on `linux/amd64`.
- Run as **root**, or as a normal user with **sudo** rights (loadout detects
  which and skips `sudo` when already root).
- Outbound HTTPS (apt, PyPI, GitHub, Go/Ruby package registries).

## Installation

loadout ships as one prebuilt binary — nothing to compile.

```bash
curl -fsSL https://github.com/dover247/loadout/raw/refs/heads/main/loadout -o loadout
chmod +x loadout
./loadout --version
```

Optionally put it on your `$PATH`:

```bash
sudo install -m 0755 loadout /usr/local/bin/loadout
```

## Quick start

```bash
./loadout            # install the whole toolbox into /tools
./loadout -n         # dry run: print every action, change nothing
```

When it finishes, open a new shell (or `source ~/.bashrc`) so `/tools` takes
effect on your `$PATH`.

## Commands

| Command                   | Description                                                                 |
| ------------------------- | --------------------------------------------------------------------------- |
| `loadout`                 | Provision the whole toolbox (the default when no command is given).         |
| `loadout install <tool…>` | Install only the named tools, plus the base toolchain they depend on.       |
| `loadout list`            | Print the catalog — every tool that **can** be installed.                   |
| `loadout status`          | Live inventory — what **is** installed, its kind, `$PATH` health, location. |
| `loadout help [command]`  | Full help, or help for one command.                                         |
| `loadout version`         | Version and build info.                                                     |

`loadout status` is an alias for `loadout list --status`. Add `-v` for versions,
`-s` to show only tools shadowed on `$PATH`.

## Flags

**Build commands** (`loadout` / `provision` and `install`; flags and
tool/section names may appear in any order):

| Flag                    | Description                                                                 |
| ----------------------- | --------------------------------------------------------------------------- |
| `-n`, `--dry-run`       | Print actions, change nothing.                                              |
| `-c`, `--concurrency N` | Max parallel tasks (default: #CPUs, clamped 2–8).                           |
| `--tools-dir DIR`       | Install root (env `TOOLS_DIR`, default `/tools`).                           |
| `--repos-dir DIR`       | Where tool source repos are cloned (env `LOADOUT_REPOS`, default `~/.local/share/loadout/repos`). |
| `--log FILE`            | Log file (`""` disables).                                                   |
| `--no-color`            | Plain output (also honors `NO_COLOR`).                                      |
| `-V`, `--verbose`       | Stream child-command output to the console.                                 |

**Provision only** (full build):

| Flag           | Description                                   |
| -------------- | --------------------------------------------- |
| `--minimal`    | Skip the heavy ProjectDiscovery suite (`pd`). |
| `--only a,b`   | Run only these sections.                      |
| `--skip a,b`   | Skip these sections.                          |
| `--list`       | List section names and exit.                  |

**`list` / `status`:**

| Flag                | Description                                                      |
| ------------------- | ---------------------------------------------------------------- |
| `--kind KIND`       | Show only one kind (see `install` list above).                   |
| `--names`           | Print tool names only, one per line (scriptable).                |
| `--sections`        | List sections instead of tools.                                  |
| `-v`, `--versions`  | With `status`: also probe each tool's version (slower).          |
| `-s`, `--shadowed`  | With `status`: show only tools shadowed on `$PATH`.              |
| `--tools-dir DIR`   | Toolbox root to inspect (env `TOOLS_DIR`, default `/tools`).      |
| `--no-color`        | Plain output (also honors `NO_COLOR`).                           |

## Installing specific tools

```bash
loadout list                     # everything available
loadout list --kind pipx         # filter by kind (apt, pipx, gem, uv, pd, bin, repo, custom, wordlist)
loadout list --names | fzf       # bare names, one per line (scriptable)

loadout install nxc nuclei ffuf  # install just these, with their dependencies
loadout install -n certipy       # preview a targeted install
```

`install` pulls in the base toolchain each tool needs and skips the rest. Names
accept aliases — `netexec`→`nxc`, `ligolo-ng`→`ligolo`, `printerbug`→`krbrelayx`,
and more.

## How it works

loadout runs in ordered **phases**; sections within a phase run concurrently:

```
system/toolchain → installs (apt · pipx · pd · gem · uv · repos · bin)
                 → symlinks → finalize (custom · PATH · wordlists)
                 → verify → cleanup
```

- **`/tools` is a flat launcher surface** — only executables (release binaries,
  symlinks, and venv-backed wrapper scripts). Nothing else lives there, so the
  directory on your `$PATH` stays clean.
- **Tool source and virtualenvs live elsewhere** — cloned under
  `~/.local/share/loadout/repos/<tool>` (override with `--repos-dir` /
  `LOADOUT_REPOS`). `/tools` exposes them only through wrappers.
- **`/tools` is made authoritative on `$PATH`** — loadout appends a normalizer
  to `~/.bashrc` that prepends `/tools` and de-duplicates, so its copies win over
  `~/go/bin`, `~/.pdtm/go/bin`, `~/.local/bin`, and `/usr/bin`.
- **`verify` fails the run** (non-zero exit) if a required tool is missing, so a
  broken provision is caught rather than reported as success.

## Output and logging

A run is **quiet by default**: one status line per tool (installed / failed)
rather than raw apt/pip/go/git chatter. A failed command still prints a short
tail of its output so you can see what broke, and the **full** output of every
command always goes to the log file:

```
$XDG_STATE_HOME/loadout/loadout.log      # default: ~/.local/state/loadout/loadout.log
```

Use `-V` / `--verbose` to stream everything to the console. Output is colored on
a terminal and plain when piped or when `NO_COLOR` is set.

## Environment variables

| Variable         | Effect                                                             |
| ---------------- | ------------------------------------------------------------------ |
| `TOOLS_DIR`      | Install root (overridden by `--tools-dir`; default `/tools`).      |
| `LOADOUT_REPOS`  | Source-repo checkout dir (overridden by `--repos-dir`).            |
| `GITHUB_TOKEN`   | Lifts the GitHub API rate limit for release-binary downloads.      |
| `XDG_STATE_HOME` | Base for the log directory.                                        |
| `NO_COLOR`       | Disables colored output.                                           |

## Tools installed

| Category | Tools |
| -------- | ----- |
| **Recon / web** | nmap · nuclei · httpx · naabu · dnsx · subfinder · katana · urlfinder · mapcidr · pdtm · ffuf · sqlmap · testssl · wpscan · gitleaks · jwt_tool |
| **AD / Kerberos / relay** | nxc (NetExec) · certipy · bloodyAD · impacket · krbrelayx (+ printerbug, addspn, dnstool) · PKINITtools · pywhisker · bloodhound-python (CE) · pre2k · kerbrute · timeroast · mimikatz |
| **Credentials / capture / shells** | hashcat (source, v8+) · john · responder · pcredz · evil-winrm · manspider |
| **Cloud / OSINT** | o365spray · cloud_enum · metaextract · username-anarchy |
| **Network / access** | ligolo-ng · xfreerdp3 |
| **Analysis / wordlists** | bloodbash · SecLists *(optional)* |

Run `loadout list` for the authoritative catalog with sources and descriptions,
or `loadout status` for the live inventory of what is installed.

## Troubleshooting

- **A tool runs an old version / `loadout status` shows `shadow`.** Another copy
  earlier on `$PATH` is winning (e.g. an apt package). Open a new shell so the
  `/tools`-first PATH fix applies, or remove the duplicate. `loadout status -s`
  lists only shadowed entries.
- **`apt` warns "Release file is not valid yet".** The machine's clock is behind
  real time — common on freshly imaged VMs. Sync it (`sudo timedatectl set-ntp
  true`) and re-run.
- **GitHub release downloads rate-limited.** Set `GITHUB_TOKEN` to raise the API
  limit.
- **See exactly what a step did.** Re-run with `-V`, or read the full log at
  `~/.local/state/loadout/loadout.log`.
