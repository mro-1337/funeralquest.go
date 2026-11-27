# Funeral Quest 1.8.0: Quick Install

This gets a town running in about ten minutes. The full guide is `README.md` (or `docs/Funeral-Quest-Manual.html`).

## 1. Unzip

Download the zip for your system and unzip it somewhere permanent. It unpacks into a `FuneralQuest-1.8.0` folder; you can rename that, for example to `C:\doors\fq`.

| System | Download | Game program | Editor |
|---|---|---|---|
| Windows | `FuneralQuest-1.8.0-Windows.zip` | `fq.exe` (64-bit), `fq32.exe` (32-bit), `fq-arm64.exe` | `fqedit.exe`, `fqedit32.exe`, `fqedit-arm64.exe` |
| Linux / Raspberry Pi | `FuneralQuest-1.8.0-Linux.tar.gz` | `fq-linux`, `fq-linux32`, `fq-linux-arm64`, `fq-linux-arm` | `fqedit-linux...` |
| macOS | build from `FuneralQuest-1.8.0-Source.zip` (`./build.sh`, needs Go 1.24+) | `fq-macos-arm` (M1 and later), `fq-macos-intel` | `fqedit-macos-...` |
| FreeBSD / OpenBSD / NetBSD | build from `FuneralQuest-1.8.0-Source.zip` (`./build.sh`, needs Go 1.24+) | `fq-freebsd`, `fq-freebsd-arm64`, `fq-openbsd`, `fq-netbsd` | `fqedit-...` |

For example, unzip to `C:\doors\fq` or `/opt/fq`. Keep the folders (`assets`, `gamedata`, `web`) next to the programs. The `data` folder is created on the first run and holds the whole game.

**Linux:** unpack with `tar -xzf FuneralQuest-1.8.0-Linux.tar.gz` (the programs are already runnable).
**macOS / BSD (built from source):** make the programs runnable: `chmod +x fq-* fqedit-*`
**macOS:** the programs aren't signed by Apple, so the first time run: `xattr -d com.apple.quarantine fq-macos-* fqedit-macos-*`

The examples below use `fq.exe`. Use your system's name instead (e.g. `./fq-linux`).

Check that it works:

```
fq.exe version
fq.exe admin players          (prints "Day 1")
```

## 2. Pick how people will play

You can use any mix of these. They all share the same town.

### A. Web game (browser)

```
fq.exe web --port 8080
```

Open `http://localhost:8080/`. To keep it running, use Task Scheduler ("At startup") or NSSM on Windows, or systemd on Linux.

### B. BBS door (Synchronet, Mystic, WWIV, any BBS that writes DOOR32.SYS)

Command line for the BBS:

```
C:\doors\fq\fq32.exe door %f          (Synchronet: %f = path of door32.sys)
C:\doors\fq\fq32.exe door C:\mystic\temp%N   (Mystic)
/sbbs/xtrn/fq/fq-linux door %f        (Linux)
```

**Synchronet (SCFG):** Native = Yes, I/O = Socket, drop file = Door32.sys, multiple users = Yes.
Use `fq32.exe` with 32-bit BBS software and `fq.exe` with 64-bit. The README has the details for each BBS.

SyncTERM callers get the original artwork (JPEG XL) and sound automatically. Other terminals get Sixel pictures or text.

### C. Telnet server (no BBS needed)

```
fq.exe telnet --port 2323
```

### D. RLogin (BBS gateway, or SyncTERM's RLogin)

In the editor or web admin, under **Settings → RLogin**, set the RLogin port (e.g. 1513) and a long BBS gateway secret. Then run `fq.exe telnet` (it takes RLogin callers too) or `fq.exe rlogin`. Ready-made setups for Synchronet, Mystic and SyncTERM are in `extras/rlogin/`.

## 3. Set the admin password

**In the browser:** start `fq.exe web`, then open `http://localhost:8080/sysop/` (or click **Admin** on the game's front page). The first time, it asks for a **setup code**. You'll find the code:

* in the `fq web` window (`ADMIN SETUP CODE: XXXX-XXXX-XXXX`)
* in `data\server.log`
* in `data\sysop-setup-code.txt`

Enter it and choose an admin password (10+ characters).

**Or from the command line:** `fq.exe admin sysop-password`

The admin pages have every setting: game rules, players, game texts, news, security, network, logs. There's also the full-screen editor, `fqedit.exe`, which needs no web server.

## 4. Set up your town

In the web admin's **Settings** tab, or in `fqedit`:

* **Town name**, turns per day, and the other game rules
* **Security**: the never-block list (`data\allowlist_ip.txt`) already has 127.0.0.1. Add your own IP address too so you can't lock yourself out.
* **HTTPS** (optional): Security tab → *Make a self-signed certificate*, then restart `fq web`. For a public site without browser warnings, use a Let's Encrypt certificate instead (see the README).

## 5. Link with other towns (optional)

1. Make sure something carries the network: `fq.exe web`, `fq.exe telnet`, or, for door-only BBSes, `fq.exe net`.
2. Set **This server's web address** (Settings → Network), e.g. `http://bbs.example.com:8080`, and forward that port on your router. If you can't open a port, leave it empty: your town links **outbound only**. Everything still works, raids included: your town fetches whatever other towns send it.
3. Network tab → **Link to another server**: type their address and port. Their sysop accepts the request, or turns on auto-accept. **You only need one link to join a whole network**: the town you link to introduces the others, and your town links to them by itself.

Linked towns share real-time Cafe chat, rankings, letters and raids. They also compare game versions and settings and tell you about any differences. Settings votes can put every town on the same rules.

## Upgrading from an older version

1. Stop `fq web` / `fq telnet` / `fq net` (and make sure nobody is in the door).
2. **Back up the `data` folder.** It holds the whole game. (From 1.8.0 on, the game also saves its own backup in `data/backups` before changing anything, but a copy of your own never hurts.)
3. Unzip the new version somewhere, then copy everything inside its `FuneralQuest-1.8.0` folder into your existing Funeral Quest folder, replacing the old files. Your `data` folder isn't in the zip, so it's kept. The `gamedata` text files are replaced: if you edited texts, copy your own `gamedata` back, or redo the changes in the editor.
4. Start the programs again. New settings get sensible defaults, and your players and settings are kept. Upgrading from 1.7 or older to 1.8: every parlor gets the level its Bury Points earn (up to 40), and a backup of your old data is saved as `data/backups/fq-before-upgrade-1.8.0-<date>.zip`.

## Help

* `fq.exe` with no options lists every command.
* `README.md` / `docs/Funeral-Quest-Manual.html` is the full manual.
* `CHANGES.md` lists what's new in each version.
* Logs are in `data\`: `server.log`, `door.log`, `security.log`.
