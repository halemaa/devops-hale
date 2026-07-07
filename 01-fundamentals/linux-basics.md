\# Linux Basics for DevOps



\## What it is

Linux is the OS that runs the majority of servers, containers, and cloud

infrastructure. As a DevOps engineer, you live in a Linux shell.



\## The filesystem

```mermaid

flowchart TD

&#x20;   Root\[/] --> etc\[/etc - configs]

&#x20;   Root --> var\[/var - logs, variable data]

&#x20;   Root --> home\[/home - user files]

&#x20;   Root --> usr\[/usr - installed programs]

&#x20;   Root --> tmp\[/tmp - temporary]

&#x20;   Root --> opt\[/opt - optional software]

&#x20;   var --> log\[/var/log]

```



\## Commands you use every day

\- \*\*Navigation:\*\* `pwd`, `cd`, `ls -la`, `tree`

\- \*\*Files:\*\* `cat`, `less`, `head`, `tail -f`, `grep`, `find`

\- \*\*Edit:\*\* `nano`, `vim`

\- \*\*Permissions:\*\* `chmod`, `chown`, `sudo`

\- \*\*Processes:\*\* `ps aux`, `top`, `htop`, `kill`, `systemctl`

\- \*\*Network:\*\* `ip a`, `ss -tulpn`, `curl`, `ping`

\- \*\*Disk:\*\* `df -h`, `du -sh \*`, `lsblk`, `mount`



\## Permissions in one paragraph

Every file has an owner, a group, and "others." Each gets read (4), write (2),

execute (1). `chmod 755 file` = owner all, group and others read+execute.

`755` for scripts, `644` for regular files, `600` for secrets.



\## systemd — the service manager

\- `systemctl status nginx` — check a service

\- `systemctl start|stop|restart nginx` — control it

\- `systemctl enable nginx` — start on boot

\- `journalctl -u nginx -f` — tail logs



\## Common gotchas

\- Case-sensitive filenames — `File.txt` and `file.txt` are different.

\- `sudo` runs as root. Understand a command before prefixing it.

\- `rm -rf /` will destroy the system. Never run destructive commands blindly.

\- Line endings differ between Windows (CRLF) and Linux (LF) — matters in scripts.



\## What I want to try

Set up an Ubuntu VM (VirtualBox or WSL2), harden it with basic security

(SSH keys only, firewall, fail2ban), and document the process in a new repo.



\## Sources

\- The Linux Command Line — William Shotts (free PDF)

\- Linux Journey (linuxjourney.com)

