# Funeral Quest 1.3.0: Quick Install

This gets a town running in about ten minutes. The full guide is `README.md` (or `docs/Funeral-Quest-Manual.html`).

## 1. Unzip

Download the zip for your system and unzip it somewhere permanent:

| System | Zip | Game program | Editor |
|---|---|---|---|
| Windows | `FuneralQuest-1.3.0-Windows.zip` | `fq.exe` (64-bit), `fq32.exe` (32-bit), `fq-arm64.exe` | `fqedit.exe`, `fqedit32.exe`, `fqedit-arm64.exe` |
| Linux / Raspberry Pi | `FuneralQuest-1.3.0-Linux.zip` | `fq-linux`, `fq-linux32`, `fq-linux-arm64`, `fq-linux-arm` | `fqedit-linux...` |
| macOS | `FuneralQuest-1.3.0-macOS.zip` | `fq-macos-arm` (M1 and later), `fq-macos-intel` | `fqedit-macos-...` |
| FreeBSD / OpenBSD / NetBSD | `FuneralQuest-1.3.0-BSD.zip` | `fq-freebsd`, `fq-freebsd-arm64`, `fq-openbsd`, `fq-netbsd` | `fqedit-...` |

For example, unzip to `C:\doors\fq` or `/opt/fq`. Keep the folders (`assets`, `gamedata`, `web`) next to the programs. The `data` folder is created on the first run and holds the whole game.

**Linux / macOS / BSD:** make the programs runnable: `chmod +x fq-* fqedit-*`
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
2. Set **This server's web address** (Settings → Network), e.g. `http://bbs.example.com:8080`, and forward that port on your router. If you can't open a port, leave it empty: your town links **outbound only**. That still works, except players from other towns can't raid yours.
3. Network tab → **Link to another server**: type their address and port. Their sysop accepts the request, or turns on auto-accept.

Linked towns share real-time Cafe chat, rankings, letters and raids. They also compare game versions and settings and tell you about any differences. Settings votes can put every town on the same rules.

## Upgrading from an older version

1. Stop `fq web` / `fq telnet` / `fq net` (and make sure nobody is in the door).
2. **Back up the `data` folder.** It holds the whole game.
3. Unzip the new version over the old folder. Your `data` folder is kept, and any edited texts in `gamedata` are overwritten. If you changed texts, copy your own `gamedata` back, or redo the changes in the editor.
4. Start the programs again. New settings get sensible defaults.

## Help

* `fq.exe` with no options lists every command.
* `README.md` / `docs/Funeral-Quest-Manual.html` is the full manual.
* `CHANGES.md` lists what's new in each version.
* Logs are in `data\`: `server.log`, `door.log`, `security.log`.
