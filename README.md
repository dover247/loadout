# loadout

A single, dependency-free binary that **rebuilds a complete offensive-security
toolbox under `/tools`** after an OS re-image. It installs ~40 pentest tools —
from apt packages and pipx apps to Go release binaries and cloned repos with
their own virtualenvs — then wires them onto your `$PATH` and verifies the lot.

> Built for **authorized** penetration testing, red-team engagements, and
> CTF/HTB practice. Everything it installs is dual-use tooling; keep your use
> scoped to systems you are permitted to test.

## What it is

`loadout` is a concurrent provisioner: it runs in ordered **phases**
(system → installs → symlinks → finalize → verify), fans each phase's work out
under a bounded worker pool, downloads and extracts release assets natively
(no `curl | tar` plumbing), retries transient network failures, and writes a
unified log. One command takes a bare Linux box to a ready toolbox.

- **Target:** Debian/Ubuntu (apt), **linux/amd64**. Statically linked — no
  runtime dependencies to install first.
- **Install root:** `/tools` (override with `--tools-dir` or `$TOOLS_DIR`).
- **Idempotent:** re-running skips what's already present and fast-forwards git
  repos; safe to run repeatedly.

## Quick start

```bash
chmod +x loadout

./loadout            # provision everything into /tools
./loadout -n         # DRY RUN: print every action, change nothing
./loadout --list     # list the sections (phases) and exit
```

Runs **either as root or as a normal user**. As a normal user it calls `sudo`
only where a step needs it (creating `/tools`, installing to `/usr/local/bin`,
`/usr/bin`, `/usr/share`), so that account needs sudo rights. As **root** it runs
those steps directly — no `sudo` required or invoked (handy in root containers
where `sudo` isn't installed). When done, open a new shell (or `source ~/.bashrc`)
so the `/tools`-first `$PATH` takes effect.

Set `GITHUB_TOKEN` in your environment to lift the anonymous GitHub API rate
limit (60/hr) used when resolving the latest release binaries:

```bash
GITHUB_TOKEN=ghp_xxx ./loadout
```

## Options

| Flag | Default | Description |
|------|---------|-------------|
| `-n`, `--dry-run` | off | Print actions, change nothing. |
| `--list` | — | List section names (with phase) and exit. |
| `-c`, `--concurrency <n>` | #CPUs (2–8) | Max concurrent tasks. |
| `--minimal` | off | Skip the heavy ProjectDiscovery suite (`pd`). |
| `--only <a,b>` | — | Run only these comma-separated sections. |
| `--skip <a,b>` | — | Skip these comma-separated sections. |
| `--tools-dir <path>` | `/tools` (or `$TOOLS_DIR`) | Install root. |
| `--log <path>` | `./log.txt` | Unified log file (empty string disables). |
| `[sections...]` | all | Positional args: run only the named sections. |

**Sections** (a positional/`--only`/`--skip` name): `apt`, `toolchain`, `pipx`,
`pd`, `gem`, `repos`, `bin`, `jwt`, `symlinks`, `custom`, `toolscmd`, `path`,
`wordlists`, `verify`.

```bash
./loadout --minimal                 # everything except the ProjectDiscovery suite
./loadout --only repos              # just (re)clone repos and rebuild their venvs
./loadout --skip wordlists,pd       # skip the ~1GB SecLists clone and PD tools
./loadout repos bin                 # run only these two sections
./loadout --tools-dir ~/tools -n    # dry-run into a custom root
```

## What gets installed

Everything lands under `/tools` as a flat launcher dir (binaries, symlinks into
`~/.local/bin` & `~/.pdtm/go/bin`, and cloned repos). The `tools` command
(installed by the `toolscmd` section) prints the live inventory — each tool's
kind, real location, and whether `/tools` wins on `$PATH`.

### System packages (apt)
`git`, `curl`, `wget`, `unzip`, `build-essential`, `pkg-config`,
`python3` (+ `venv`/`pip`/`dev`/`full`), `pipx`, `golang-go`, `ruby` (+ `dev`),
plus build/runtime libs (`libssl`, `libffi`, `libkrb5`, `libpcap`, `libmagic`,
`exiftool`, `libxml2`/`libxslt`, `rustc`/`cargo`, …).

### AD / Kerberos / relay
| Tool | Source |
|------|--------|
| impacket (`secretsdump.py`, `getTGT.py`, …) | fortra/impacket |
| certipy | ly4k/Certipy (`certipy-ad`) |
| bloodyAD | CravateRouge/bloodyAD |
| krbrelayx (+ `printerbug`, `addspn`, `dnstool`) | dirkjanm/krbrelayx |
| PKINITtools | dirkjanm/PKINITtools |
| pywhisker | ShutdownRepo/pywhisker |
| bloodhound-python (CE) | dirkjanm/BloodHound.py `bloodhound-ce` |
| mimikatz | gentilkiwi/mimikatz (ParrotSec mirror) |
| kerbrute | ropnop/kerbrute |
| timeroast | SecuraBV/Timeroast |
| pre2k | garrettfoster13/pre2k |

### Creds / capture / shells
| Tool | Source |
|------|--------|
| nxc (NetExec) | Pennyw0rth/NetExec |
| hashcat | hashcat/hashcat (apt) |
| john | openwall/john (built from source) |
| responder | lgandx/Responder |
| pcredz | lgandx/PCredz |
| evil-winrm | Hackplayers/evil-winrm (gem) |
| manspider | blacklanternsecurity/MANSPIDER |
| o365spray | 0xZDH/o365spray |
| dehashapitool | hmaverickadams/DeHashed-API-Tool |

### Web / recon
| Tool | Source |
|------|--------|
| ffuf | ffuf/ffuf |
| sqlmap | sqlmapproject/sqlmap (apt) |
| jwt_tool | ticarpi/jwt_tool (own venv + wrapper) |
| gitleaks | gitleaks/gitleaks |
| metaextract | cereZ23/metaextract |
| username-anarchy | urbanadventurer/username-anarchy |

### ProjectDiscovery suite (via `pdtm`)
`nuclei`, `httpx`, `naabu`, `dnsx`, `subfinder`, `katana`, `urlfinder`,
`mapcidr`, `pdtm`. *(Skipped with `--minimal`.)*

### Network / access
| Tool | Source |
|------|--------|
| ligolo-ng (server + agent, incl. Windows agent) | nicocha30/ligolo-ng |
| nmap | nmap/nmap (apt) |
| xfreerdp3 | FreeRDP/FreeRDP (apt) |

### Wordlists (optional, ~1GB)
SecLists → `/usr/share/seclists` (+ extracts `rockyou.txt`). Skip with
`--skip wordlists`.

## How it works (phases)

| Phase | Sections | What happens |
|------:|----------|--------------|
| 0 | `apt`, `toolchain` | System packages, toolchains, per-user bin dirs; fetch gitleaks release. |
| 1 | `pipx`, `pd`, `gem`, `repos`, `bin`, `jwt` | pipx apps, ProjectDiscovery suite, gems, cloned repos + venvs/builds, release binaries, jwt_tool. |
| 2 | `symlinks` | Create `/tools/<name>` launchers (symlinks + venv/path-aware wrappers). |
| 3 | `custom`, `toolscmd`, `path`, `wordlists` | timeroast, the `tools` inventory command, the authoritative `/tools`-first `$PATH` normalizer in `~/.bashrc`, optional SecLists. |
| 4 | `verify` | Stat every expected tool; a missing required tool fails the run. |

The `path` section appends an **authoritative** PATH block to `~/.bashrc` that
prepends `/tools` and de-duplicates `$PATH`, so `/tools` launchers win over any
`~/go/bin`, `~/.pdtm/go/bin`, `~/.local/bin` or `/usr/bin` copies — and makes
login shells source `~/.bashrc` so it also applies over SSH.

## Building from source

This repo ships the prebuilt **linux/amd64** binary. To rebuild from the Go
source (not included here):

```bash
cd gosetup && go build -o loadout .
```
