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
./loadout            # install everything into /tools
./loadout -n         # dry run: print actions, change nothing
./loadout --list     # list sections and exit
```

Runs as **root** or as a normal user with **sudo** rights. When it finishes,
open a new shell (or `source ~/.bashrc`) so `/tools` takes effect on `$PATH`.

Useful flags: `--minimal` (skip the ProjectDiscovery suite), `--only a,b` /
`--skip a,b` (pick sections), `--tools-dir <path>` (change install root).
Set `GITHUB_TOKEN` to lift the GitHub API rate limit for release downloads.

## Tools installed

nmap · hashcat · john · sqlmap · ffuf · kerbrute · gitleaks · jwt_tool ·
nxc (NetExec) · certipy · bloodyAD · impacket · krbrelayx · printerbug ·
PKINITtools · pywhisker · bloodhound-python (CE) · mimikatz · pre2k · timeroast ·
responder · pcredz · evil-winrm · manspider · o365spray · dehashapitool ·
metaextract · username-anarchy · ligolo-ng · xfreerdp3 · nuclei · httpx · naabu ·
dnsx · subfinder · katana · urlfinder · mapcidr · pdtm · SecLists (optional).

The `tools` command (installed by loadout) prints the live inventory at any time.
