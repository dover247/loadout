# loadout

A single, static binary that rebuilds a full offensive-security toolbox under
`/tools` after an OS re-image — installs ~40 pentest tools, wires them onto your
`$PATH`, and verifies the result. One command takes a bare Linux box to a ready
toolbox.

> For **authorized** pentesting, red-team work, and CTF/HTB practice only.
> Target: Debian/Ubuntu, **linux/amd64**. Idempotent — safe to re-run.

## Run

```bash
chmod +x loadout
./loadout                      # install everything into /tools
./loadout -n                   # dry run: print actions, change nothing

./loadout list                 # catalog: every tool that CAN be installed
./loadout status               # inventory: what IS installed (kind, PATH health, location)
./loadout status -v            # same, with versions (-s shows only shadowed tools)
./loadout install nxc nuclei   # install only the named tool(s), with their deps

./loadout help                 # full help, commands, examples
./loadout --version            # version / build info
```

Runs as **root** or as a normal user with **sudo** rights. When it finishes,
open a new shell (or `source ~/.bashrc`) so `/tools` takes effect on `$PATH`.

`/tools` is kept as a flat **launcher surface** — only executables (binaries,
symlinks, venv-backed wrappers). Tool **source** checkouts and their virtualenvs
live separately under `~/.local/share/loadout/repos/` (override with
`--repos-dir` or `LOADOUT_REPOS`), so the directory on your `$PATH` stays clean.

**Pick specific tools:** `loadout list` shows everything available (filter with
`--kind pipx`, or `--names` for a bare list); `loadout install <name…>` installs
just those, pulling the base toolchain they need and skipping the rest. Names
accept aliases (`netexec`→`nxc`, `ligolo-ng`→`ligolo`, …).

Useful flags: `--minimal` (skip the ProjectDiscovery suite), `--only a,b` /
`--skip a,b` (pick sections), `--tools-dir <path>` (change install root),
`--repos-dir <path>` (where source repos are cloned; default
`~/.local/share/loadout/repos`), `-V`/`--verbose` (stream child-command output),
`--no-color` (plain output; also honors `NO_COLOR`), `--list` (list sections).
Set `GITHUB_TOKEN` to lift the GitHub API rate limit.

By default a run is quiet: it prints one status line per tool (installed /
failed) rather than the raw apt/pipx/go/git chatter. Failures still show a short
tail of the command's output so you know what broke, and the **full** output of
every command always goes to the log file under `$XDG_STATE_HOME/loadout/`
(default `~/.local/state/loadout/loadout.log`). Use `-V` to stream everything to
the console. Output is colored on a terminal and plain when piped.

## Tools installed

nmap · hashcat · john · sqlmap · ffuf · kerbrute · gitleaks · jwt_tool ·
testssl · wpscan · nxc (NetExec) · certipy · bloodyAD · impacket · krbrelayx ·
printerbug · PKINITtools · pywhisker · bloodhound-python (CE) · bloodbash ·
mimikatz · pre2k · timeroast · responder · pcredz · evil-winrm · manspider ·
o365spray · cloud_enum · metaextract · username-anarchy ·
ligolo-ng · xfreerdp3 · nuclei · httpx · naabu · dnsx · subfinder · katana ·
urlfinder · mapcidr · pdtm · SecLists (optional).

`loadout status` prints the live inventory at any time — each tool's kind, where
it resolves, and whether `/tools` wins on `$PATH` (`unified`) or is shadowed.
