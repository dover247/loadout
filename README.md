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

./loadout list                 # list every tool (name, kind, source, description)
./loadout install nxc nuclei   # install only the named tool(s), with their deps

./loadout help                 # full help, commands, examples
./loadout --version            # version / build info
```

Runs as **root** or as a normal user with **sudo** rights. When it finishes,
open a new shell (or `source ~/.bashrc`) so `/tools` takes effect on `$PATH`.

**Pick specific tools:** `loadout list` shows everything available (filter with
`--kind pipx`, or `--names` for a bare list); `loadout install <name…>` installs
just those, pulling the base toolchain they need and skipping the rest. Names
accept aliases (`netexec`→`nxc`, `ligolo-ng`→`ligolo`, …).

Useful flags: `--minimal` (skip the ProjectDiscovery suite), `--only a,b` /
`--skip a,b` (pick sections), `--tools-dir <path>` (change install root),
`--no-color` (plain output; also honors `NO_COLOR`), `--list` (list sections).
Set `GITHUB_TOKEN` to lift the GitHub API rate limit. Output is colored on a
terminal and plain when piped; the run log is written under
`$XDG_STATE_HOME/loadout/` (default `~/.local/state/loadout/loadout.log`).

## Tools installed

nmap · hashcat · john · sqlmap · ffuf · kerbrute · gitleaks · jwt_tool ·
testssl · wpscan · nxc (NetExec) · certipy · bloodyAD · impacket · krbrelayx ·
printerbug · PKINITtools · pywhisker · bloodhound-python (CE) · bloodbash ·
mimikatz · pre2k · timeroast · responder · pcredz · evil-winrm · manspider ·
o365spray · cloud_enum · metaextract · username-anarchy ·
ligolo-ng · xfreerdp3 · nuclei · httpx · naabu · dnsx · subfinder · katana ·
urlfinder · mapcidr · pdtm · SecLists (optional).

The `tools` command (installed by loadout) prints the live inventory at any time.
