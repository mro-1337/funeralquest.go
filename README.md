# Funeral Quest — BBS door + web server (Go)

Version 1.3.0. **New here? Start with `QUICK-INSTALL.md`.** See `CHANGES.md` for what's new; `fq version` shows the version you're running. The same guides are in `docs/` as web pages you can open in any browser.

**Contents:** The JPEG XL check · Sound · Install (Synchronet, Mystic, other BBSes, RLogin) · Web server · The sysop editor (fqedit) · Web admin pages · Security (HTTPS, block lists and bans) · Linking towns together (without the web server, versions and settings, settings votes, raids) · Sysop commands · Building from source · Credits

Seth A. Robinson's 2001 *Funeral Quest*, ported to **Go** as one program that runs as:

* **a BBS door** using **DOOR32.SYS**, for BBSes on Windows (32-bit, 64-bit, ARM), Linux (x86, ARM, Raspberry Pi), macOS and the BSDs. **SyncTERM** callers get Akiko Robinson's original artwork as **JPEG XL** pictures inside the ANSI screen.
* **a web server** with a browser version that looks like the original Flash client, also using JPEG XL.

Both modes share the same game data, so BBS callers and web players live in the same town. They share the rankings, the Pale Horse Cafe chat, letters, Yang's deliveries and parlor raids.

Towns on different servers can **link together**: type in the other server's address and the two towns share Cafe chat, network rankings and letters, and players can **break into, fight and burn out** parlors in the other town.

No runtimes, no installers, no npm, no database server. It's a single executable plus a folder of data files.

Every system has two programs: **fq** (the game: door, web server, telnet server and sysop commands) and **fqedit** (the full-screen sysop editor).

| System | Game | Editor |
|---|---|---|
| Windows 64-bit (most PCs) | `fq.exe` | `fqedit.exe` |
| Windows 32-bit (and 32-bit BBS software on 64-bit Windows) | `fq32.exe` | `fqedit32.exe` |
| Windows on ARM | `fq-arm64.exe` | `fqedit-arm64.exe` |
| Linux x86-64 | `fq-linux` | `fqedit-linux` |
| Linux 32-bit x86 | `fq-linux32` | `fqedit-linux32` |
| Linux ARM 64-bit (Raspberry Pi 4/5 with a 64-bit OS, ARM servers) | `fq-linux-arm64` | `fqedit-linux-arm64` |
| Linux ARM 32-bit (Raspberry Pi with a 32-bit OS) | `fq-linux-arm` | `fqedit-linux-arm` |
| macOS Apple Silicon (M1 and later) | `fq-macos-arm` | `fqedit-macos-arm` |
| macOS Intel | `fq-macos-intel` | `fqedit-macos-intel` |
| FreeBSD x86-64 / ARM64 | `fq-freebsd` / `fq-freebsd-arm64` | `fqedit-freebsd` / `fqedit-freebsd-arm64` |
| OpenBSD x86-64 | `fq-openbsd` | `fqedit-openbsd` |
| NetBSD x86-64 | `fq-netbsd` | `fqedit-netbsd` |

The downloads are split by system (Windows, Linux, macOS, BSD). Each one is complete: the programs for that system plus all the art, sounds, texts and source code. The examples below use `fq.exe`; on other systems use the matching name, e.g. `./fq-linux`. On Unix-like systems you may need `chmod +x fq-* fqedit-*` first. On macOS, the programs aren't signed by Apple, so the first time run `xattr -d com.apple.quarantine fq-macos-* fqedit-macos-*` (or right-click → Open).

```
assets/       pictures: assets/door (JPEG XL + PBM masks + PNG twins), assets/web (JPEG XL)
              sound: assets/sound (the original game's sounds and music as Ogg Vorbis)
gamedata/     the original game text files (risks, sales pitches, events, news...) - edit freely
web/public/   the browser page
data/         created on first run: world.json (the game), news/, logs, block lists, net/ (links), tls/
docs/         this manual, the quick install guide and the change list as web pages
extras/       RLogin setups for Synchronet, Mystic and SyncTERM
```

## The JPEG XL check

Every time a caller enters the door, it asks their terminal what it can do before drawing anything:

1. `CSI 6n` gives the screen size, and `CSI c` tells whether the terminal is SyncTERM (CTerm).
2. **`APC SyncTERM:Q;JXL`** asks whether the terminal can decode JPEG XL. SyncTERM answers `CSI =1;1-n` if it can and `CSI =1;0-n` if not.
3. If JPEG XL is supported, the door lists SyncTERM's picture cache (`APC SyncTERM:C;L`). It uploads each picture only once (`APC SyncTERM:C;S`), and only re-sends one if it changed. It then draws with `APC SyncTERM:C;DrawJXL`, layering customers over backgrounds with PBM masks the way the Flash client did.
4. If there's **no** JPEG XL, the door checks for Sixel (CTerm feature 4, or attribute 4 in the DA1 reply). If the terminal has Sixel, the same art is sent as Sixel. It looks the same but costs about 100 KB per picture instead of roughly 10 KB once.
5. If the terminal has neither, the game is played in text, with a caption in the picture box.

The result for each caller is written to `data\door.log`, for example:
`caller "Skelly" node 1: ... syncterm=true jpegxl=true sixel=false audio=true cell=8x16`.

## Sound

The original game's sounds and music are included as Ogg Vorbis files: the ambience and effects of every scene, the fight and sneak music, the town map tune, the newspaper and the intro. They play where the Flash client played them.

* **In the door,** the door asks `APC SyncTERM:Q;libsndfile`, then `libsndfileFormat;32;96` to check for Ogg Vorbis.
  * If the terminal answers yes, each sound is uploaded into its cache once and played with SyncTERM's audio commands: `A;Load`, `A;Copy`, `A;Queue` (looping for ambience and music) and `A;Flush` (fade out).
  * Terminals that don't answer get no sound.
  * Add `--nosound` to the door command line to turn sound off for everyone.
* **On the web,** there is a speaker button in the top bar, remembered per browser.

For JPEG XL, callers need **SyncTERM 1.4 or newer built with JPEG XL support**. The screen is the usual 80×25. Pictures fill the top-right 40×14 cells (308×224 pixels with SyncTERM's 8×16 font).

## Install

1. Unzip anywhere, for example `C:\doors\fq`. Keep the folders next to the exe.
2. Check that it runs: `fq.exe admin players` should print `Day 1`.
3. Optionally set it up:
   ```
   fq.exe admin config server_name="My BBS" turns_per_day=20
   ```

### Synchronet (Windows, Linux, Raspberry Pi, BSD, macOS)

In `scfg`, go to **External Programs → Online Programs (Doors) → (section) → Add** and set:

| Setting | Windows | Linux |
|---|---|---|
| Name / Internal Code | Funeral Quest / `FQ` | Funeral Quest / `FQ` |
| Start-up Directory | `C:\doors\fq` | `/sbbs/xtrn/fq` |
| Command Line | `fq32.exe door %f` | `./fq-linux door %f` |
| Multiple Concurrent Users | Yes | Yes |
| Native Executable/Script | **Yes** | **Yes** |
| I/O Method | **Socket** | **Socket** (or Standard; add `--stdio` to the command line) |
| BBS Drop File Type | **Door32.sys** | **Door32.sys** |
| Place Drop File In | Node Directory | Node Directory |

On a Raspberry Pi, FreeBSD or a Mac, use that system's program name instead of `fq-linux` (`./fq-linux-arm door %f`, `./fq-freebsd door %f`, `./fq-macos-arm door %f`). The settings are the same as for Linux.

`%f` is Synchronet's code for the path of the drop file it just wrote. In Socket mode the door talks straight to the caller's telnet socket, using the handle on line 2 of DOOR32.SYS.

### Mystic BBS (Windows 32/64-bit, Linux)

Add a menu command of type **D3** (run a DOOR32 door) whose data is:

```
C:\doors\fq\fq32.exe door C:\mystic\temp%N
```

Mystic writes `door32.sys` into each node's temp folder (`temp1`, `temp2`, …). The door accepts either the folder or the file itself, so pointing it at the node's temp folder is enough. `%N` is the node number (check the door command codes for your Mystic version, and use your own temp path if you moved it).

### Other 32-bit and Linux BBSes that write DOOR32.SYS

Any BBS that writes DOOR32.SYS and hands the door the socket handle works the same way:

```
fq32.exe door <full path of door32.sys, or the folder it is in>
```

* **WWIV 5:** `C:\doors\fq\fq32.exe door %E`. `%E` is WWIV's full path of `door32.sys`.
* **Maximus NG:** use the `door32_run` (`xtern_door32`) menu command, with the node temp directory as the argument.
* **EleBBS, Talisman, ENiGMA½, Mystic and others:** pass the DOOR32.SYS path the BBS gives you.

DOOR.SYS and DORINFO1.DEF are also read, for BBSes that run the door over standard input and output; add `--stdio` in that case. A player is identified by their BBS alias. If several BBSes share one game folder, give each one `--bbs-id NAME`.

### RLogin (BBS gateways and SyncTERM)

Instead of running the door, a BBS can send callers to Funeral Quest over **RLogin**, and players can also connect directly with SyncTERM's RLogin. Set it up under **Settings → RLogin** (web admin or fqedit):

| Setting | What it does |
|---|---|
| RLogin port (`rlogin_port`) | Port to listen on, e.g. 1513. 0 = off. `fq telnet` takes RLogin callers too; `fq rlogin` runs just RLogin. |
| Hosts allowed to connect (`rlogin_trusted`) | IPs or ranges, comma separated. `*` = anyone. If the BBS is on the same machine, `127.0.0.1` is safest. |
| BBS gateway secret (`rlogin_secret`) | Shared secret the BBS sends with the caller's name. 12+ characters, kept private. Empty = no gateway logins. |
| BBS ID for gateway callers (`rlogin_bbs_id`) | Gateway callers are the same players as door callers from the BBS with this ID (the door's `--bbs-id`, normally `bbs`), so a caller keeps their parlor whether they come in by door or RLogin. |

**How callers are identified.** RLogin has two user fields and no password field, and BBS programs disagree about which field holds the name and which holds the password. Synchronet sends the password first; SyncTERM has "RLogin" and "RLogin Reversed"; Mystic decides for itself. So Funeral Quest never relies on the order:

1. The caller's address must be in **Hosts allowed to connect** and must pass the block lists and bans.
2. **BBS gateway:** if exactly one field is the gateway secret, the other is the caller's name, and it's trusted because the BBS already checked the caller. A new name gets the "set up your parlor" screen.
3. **Their own password:** otherwise both orders are tried as Funeral Quest logon + password.
4. **Anything else** gets the normal log-on screen, where they can log on or open a parlor. A name on its own is never trusted.

Failed log-ons count toward the same temporary bans as telnet and web log-ons. Callers coming in through a BBS gateway all arrive from the BBS's address, so the per-address connection limits don't apply to them. Passwords and the secret never appear on screen or in the logs.

Setup files for Synchronet (an rlogin gate script), Mystic (an `IR` menu command) and SyncTERM are in `extras/rlogin/`, with a testing checklist.

### Test it without a BBS

```
fq.exe telnet --port 2323
```

Then connect SyncTERM to `localhost:2323`. Type `NEW` to make a character; the logon name and password also work on the web site.

## Web server

```
fq.exe web --port 8080
```

Then open `http://localhost:8080/`. Run the web server in the **same folder** as the door so both share `data\world.json`. To keep it running on Windows, use Task Scheduler ("At startup") or a service wrapper such as NSSM. For a public site, use HTTPS: either `--cert`/`--key` (see Security) or a proxy such as Caddy or IIS with `--trust-proxy`. The web server logs to `data\server.log` and `data\security.log`.

BBS players can also play in a browser: in the door, open **Stats → Web login** to pick a web logon name and password for the same parlor.

Browsers with native JPEG XL (Safari) show the art directly. For other browsers the page includes a small libjxl WebAssembly decoder, so the art is still JPEG XL.

## The sysop editor: fqedit

`fqedit.exe` is the editor program, like the old server's menus. Run it in the Funeral Quest folder, or use `fqedit.exe --home C:\doors\fq`. It's full screen in a console window (Windows 10 or later, or any Linux terminal). Use the arrow keys, Enter to change something, and Esc to go back.

* **Game settings**: every option with a description, grouped (Game, New players, Scoring, Odds, Tournament, Security, Network). Bad values are refused.
* **Players**: find, edit any value (money, BP, turns, tags, luck, hearse...), close or reopen, set a password, delete.
* **Game texts**: opens karma, ratings, news, events, risks, sales pitches, fortune cookies or the swear filter in Notepad (on Linux, `$EDITOR` or nano). When you close the editor, fqedit checks that the game can still read the file. If it can't, the old file is put back and your edit is kept as `name.txt.rejected`. The previous version is always kept as `name.txt.bak`.
* **News and maintenance**: post news, run maintenance, reset the game.
* **Security**: temporary bans (lift one, or ban an address), and the block lists.
* **Network links**: your fingerprint and address, auto-accept, link to another server, accept requests, sync, block or remove. It also shows version and settings differences with each town (with "Use their settings") and settings votes (propose yours, accept or decline theirs).
* **Web sysop password**, and **View logs**.

You don't need to stop anything to use it. The door, telnet and web servers pick up changed settings and texts within seconds.

## Web admin pages

The web server has a full admin section at **`http://yourserver:8080/sysop/`**. There's also an **Admin** link at the bottom of the game's log-on page.

**Admin log-on.** The first time, there's no admin password yet. The web server makes a one-time **setup code** and shows it only on the server:

* in the window where `fq web` is running (`ADMIN SETUP CODE: PT87-L365-N6P2`)
* in `data\server.log`
* in `data\sysop-setup-code.txt`

Open `/sysop/`, type the code and choose an admin password (10 characters or more). After that, you log on with the password. Only someone who can see the server's screen or files can do the first setup, so a stranger who finds your admin page first can't take it over. You can also set or change the password with fqedit or `fq.exe admin sysop-password`.

**Tabs:**

* **Status**: version, day, players, who's playing right now, bans, links, and warnings about linked towns (who needs to update, settings that differ, votes waiting for you)
* **Settings**: every setting, grouped:
  * Game, New players, Scoring, Odds, Tournament
  * Security
  * Server: web and telnet ports, listen address, HTTPS certificate and key, "behind my own proxy", sound
  * Network: this server's address, auto-accept, chat, mail and raids on or off, settings votes, raid limit
* **Players**: find, edit, close, set a password, delete
* **Game texts**: edit every text file in the browser, checked before saving
* **News & maintenance**: post news, run maintenance, reset the game
* **Security**: temporary bans (lift one, or ban an address), and the block lists
* **Network**: fingerprint, version, the auto-accept switch, linking, accepting, syncing, blocking and removing. It also shows each town's version and settings differences (with "Use theirs"), and settings votes (propose yours, accept or decline theirs).
* **Logs**: `server.log`, `door.log`, `security.log`
* **Password**: change the admin password, or turn the admin pages off

The **Server** settings take effect the next time `fq web`, `fq telnet` or the door starts. Switches on the command line (`--port`, `--cert`, `--key`, `--host`, `--trust-proxy`, `--nosound`) override them. To allow the admin pages only from your own network, set **Admin pages allowed from** (`sysop_allow`), e.g. `127.0.0.1, 192.168.0.0/16`. Everyone else gets "not found".

## Security

The web server was written to be safe on the open internet:

* Passwords (players and sysop) are stored as salted PBKDF2-SHA256 hashes (120,000 rounds). Log-ons take the same time whether the name exists or not.
* Sysop sessions use a random 256-bit token kept in the browser tab, not a cookie, so other web sites can't act as you (no CSRF). They're tied to your IP address and end after 30 idle minutes, after 8 hours, or when the sysop password changes. Every sysop change is written to `security.log`.
* A strict Content-Security-Policy, plus `X-Frame-Options`, `nosniff` and `no-referrer` headers. No directory listings, hidden files or `.bak` files are served. Only GET, HEAD and POST are accepted, and request sizes, header sizes and timeouts are limited (protection against slow-loris attacks).
* Rate limits on every request, log-ons and new accounts (3 per address per hour).
* HTTPS is built in: `fq.exe web --port 443 --cert fullchain.pem --key privkey.pem` (TLS 1.2+, with HSTS). If you run it behind your own proxy (Caddy, IIS, nginx) instead, add `--trust-proxy` so bans hit the real caller's address. The forwarded address is only believed when the proxy is on the same machine or LAN.

### HTTPS certificate

The web server can make its own **self-signed certificate**: Security tab → "Make a self-signed certificate", fqedit → Security → HTTPS certificate, or `fq.exe admin make-cert [host ...]`.

* It covers this server's web address host name, the computer's name, localhost and 127.0.0.1, or the names you give.
* It's saved in `data\tls\` and valid for 825 days.
* The web server is pointed at it, and **This server's web address** switches to `https://` by itself. Restart `fq web` to start using it.

**For players:** browsers warn that a self-signed certificate isn't from a known authority until each visitor accepts it. For a public site, a free Let's Encrypt certificate avoids that: use Caddy in front of `fq web`, or point **HTTPS certificate file** and **HTTPS key file** at certbot's files.

**For linked Funeral Quest servers:** a self-signed certificate is fine. They trust it the first time they connect and remember (pin) its fingerprint. If it changes later, they refuse it ("its HTTPS certificate has changed"), since that could be someone in between. If you really made a new one, the other sysops remove the link and link again. Messages between towns are signed either way; HTTPS adds privacy for letters on the way.

When a town changes its address (say from http to https, or to another port on the same host), it tells its linked towns, and they update. A town can't move itself to a different host name.

`fq.exe admin cert` shows the current certificate and its fingerprint. `fq.exe admin https-off` goes back to plain HTTP.

### Block lists and bans

These files are in the `data` folder. Each one is created with instructions the first time. Edit them in fqedit, on the sysop pages, or in Notepad; changes apply within seconds without a restart.

| File | What it does |
|---|---|
| `blocklist_ip.txt` | One IP or range per line: `203.0.113.7`, `198.51.100.0/24`, `2001:db8::/32` |
| `blocklist_hosts.txt` | Host names, with `*` wildcards: `*.badisp.example`. Matched against the caller's reverse DNS; plain names are also looked up to their IPs |
| `allowlist_ip.txt` | Never blocked or banned. Your own address and LAN belong here so you can't lock yourself out (127.0.0.1 is there already) |
| `tempbans.json` | Temporary bans, shared by the telnet server, the web server and every door node |

Blocked callers are refused by the telnet server, the web game, the sysop pages and the network link.

**Temporary bans** happen automatically. An address is banned for 30 minutes after **5 strikes in 10 minutes**, and each repeat ban doubles (1 h, 2 h, 4 h... up to 7 days). A strike is:

* a wrong telnet, web or sysop password, or going over the log-on rate limit
* a telnet log-on as an attack-bot name (`root`, `admin`, `support`, `ubnt`, `enable`, `shell`...). That's an instant ban
* more than 8 new telnet connections in a minute (instant ban), or more than 3 at once
* a forged or replayed network message, or flooding the web server

Telnet callers also have 3 minutes to log on before they're disconnected. You can change the limits under Settings → Security. See and lift bans in fqedit, on the sysop pages, or with:

```
fq.exe admin bans
fq.exe admin unban 203.0.113.7
fq.exe admin ban 203.0.113.7 1440        (minutes)
```

In door mode the BBS handles the connection, so use your BBS's own IP filters for door callers. The game's block lists cover the telnet and web servers.

## Linking towns together (Funeral Quest network)

Any number of Funeral Quest servers can link up. Linked towns share:

* **Real-time Cafe chat**: what's said in your Pale Horse Cafe shows up in theirs a split second later as `[Deathville] Alice: hi`, and the other way around. `/net` in the cafe lists linked towns and who's on.
* **Network rankings**: the rankings get an **All towns** button (and the web log-on page gets an "All linked towns" link).
* **Letters**: "Write a letter" also finds players in linked towns. Out-of-town letters have a working **Reply** button. Letters that can't be delivered come back to the sender.
* **Raids**: a visit to a rival parlor, and hiring the Twins at the desk, can also target a parlor in a linked town. See **Raids between towns** below.

To link:

1. Run something that carries the network: `fq.exe web`, `fq.exe telnet` or `fq.exe net` (see **Linking without the web server**).
2. Set **This server's web address** (`net_public_url`), e.g. `http://bbs.example.com:8080`, so the other server can reach you back. Leave it empty if your server can't be reached from outside; it then links **outbound only**.
3. In fqedit → Network links → **Link to another server**, or on the sysop pages → Network, type the other server's address and port (`theirbbs.example.com:8080`). You can also run `fq.exe admin link theirbbs.example.com:8080`.
4. The other sysop sees **wants to link** and accepts it (or links to you too). If they turned on **Accept link requests automatically** (`net_auto_accept`; it's a switch on the sysop Network tab and in fqedit), this happens by itself. From then on the towns stay in touch.

Each server can use any port; it's part of the address. Leave the port off only if it's 80 (`http://`) or 443 (`https://`).

### Linking without the web server

The web game isn't needed to be linked. Any of these programs runs the town's network link:

| You run | Network link | Other towns reach you on |
|---|---|---|
| `fq web` | yes | the web port |
| `fq telnet` | yes | the **network port** (`net_port` setting or `--net-port`), if you set one |
| only the door | run `fq net` alongside it (e.g. at startup) | the network port, if you set one (`fq net --port 8081`) |

Only one program per town does the network work at a time. If you run `fq web` and `fq telnet` together, one does it and the other stands by, taking over if the first one stops.

**Outbound only.** With no network port, and no web server reachable from outside, a town connects out only. It works behind a firewall or router with nothing forwarded. Leave **This server's web address** empty and link to the other towns by their address; nobody can link to you by address. An outbound-only town still gets everything by asking on each connection:

* real-time Cafe chat, both ways
* network rankings, letters both ways, settings votes, version and settings checks
* its players can raid other towns

The one thing it can't have is visitors: nobody can connect to it, so players from other towns can't raid it. Other towns list it as **outbound only** (the sysop pages, fqedit and `fq admin links` all say so), its players don't come up as raid targets, and `/net` in the Cafe marks it "connects out only". A town that normally can be reached but currently can't be is shown as **not reachable**.

To be fully connectable without the web game, set **Network port** (`net_port`, Settings → Server) and forward it on your router. Then set **This server's web address** to `http://your.address:thatport`.

### How linked servers talk

All network traffic goes to these addresses on the web port (or the network port). Door nodes don't connect to other servers for chat or mail; they share the `data` folder with whichever program runs the network.

| Address | What for | When |
|---|---|---|
| `/fed/hello` | link requests | when a sysop links or accepts |
| `/fed/sync` | player lists, chat, letters, game version and settings, settings votes | every 30 seconds, and within moments of new mail |
| `/fed/listen` | real-time chat: held open until someone in that town speaks (up to 25 s), then asked again at once | all the time |
| `/fed/raid` | one step of a raid | as the raider plays |

There's no central hub. Every pair of towns links directly, and messages aren't passed along, so link to every town you want to talk to.

### Versions and game settings

Every handshake carries the game version and the town's game rules: turns per day, starting stats, scoring, odds, daily limits, tournament mode and out-of-town raids per day. Town name, security and housekeeping settings stay private and aren't compared.

* **Versions**: if a linked town runs an older version, both sysops are told which town needs to update. A town from before 1.3.0 doesn't send its version and is reported as needing to update.
* **Settings**: each sysop sees which settings differ, with "here" and "there" values. Look on the sysop Status and Network tabs, in fqedit → Network links (towns marked NEEDS ATTENTION), with `fq admin links`, and in `server.log`. Copy the other town's values one at a time or all at once ("Use theirs", or `fq admin use-theirs Gravesend`), or agree with the other sysop on new ones.

### Settings votes

To get every linked town on the same rules, **propose** yours. Use the sysop pages → Network → "Propose this town's settings to all linked towns", fqedit → Network links → Settings votes, or `fq admin propose`.

1. Every town you're linked to gets the proposal. Its sysop is asked to **accept the settings from your town**, and sees a table of what would change for them.
2. Each sysop answers **Accept** or **Decline** (web, fqedit, or `fq admin votes` then `fq admin vote <id> yes`). They can change their answer until the vote ends.
3. When **more than half of all the towns in the vote** have accepted (the proposing town counts as a yes), the vote passes. Every town in it switches to the proposed settings, including towns that declined. A town whose sysop turned on **Don't apply settings votes** (`net_vote_apply_off`) stays as it is. With just two towns, the other town has to accept.
4. If enough towns decline that it can't pass, it fails. Votes also end after 7 days.

Each town signs its vote with its own key. A town only applies a passed vote after checking the signed votes itself, so the proposing town can't fake a result. Only the game settings listed above can be changed by a vote. A new proposal from the same town replaces its open one.

Each server has its own key. Its **fingerprint** (like `3DDF-B131-5126-B736`) is shown in the editor; compare fingerprints with the other sysop by email or phone if you want to be sure who you're linking to.

**What other servers can and can't do.** Each server owns its own players; a linked server can't edit them. Here's exactly what crosses the link:

* What you send is your players' **public** name, parlor, sex, BP and online status, your public Cafe lines, and letters your players wrote to theirs. That's all. Passwords, logons, money, tags, private messages and files never leave your server.
* What they send is only added to your server, after checks. Raids are the one thing that changes your players, and your own server runs those with the normal game rules (see Raids between towns). Their player list is stored separately and only shown in rankings. Their chat lines are added to your Cafe, and their letters are delivered to the named player's mailbox. All of it has control codes and escape sequences removed and is cut to a maximum size, so a hostile server can't paint over your callers' screens.
* Nothing they send can directly change, delete or read a player, a setting, a text file or anything else on your server. Even a raid is only a request: your server decides what happens.
* Every message is signed (Ed25519) and carries a time stamp, a one-time number and your server's ID. Forged, altered, replayed or misdirected messages are rejected, and the address that sent them gets strikes.
* Link requests need your OK unless you turn on **Accept requests automatically**. If a server you already know shows up with a different key, it's flagged **KEY CHANGED** and isn't trusted until you accept it.
* You can **Block** or **Remove** a server at any time. Removing it deletes its players and any letters waiting to go to it.

### Raids between towns

Players can do everything to a parlor in a linked town that they can do in their own:

* **Break in**: Rival parlors → Visit, then type part of the name. Players in linked towns come up as "Bobby of Bone Yard, in the town of Gravesend". It uses one of your parlor visits for the day, just like a local visit.
* **Pick the padlock**: each lockpick weakens the padlock by 5 to 10 points, and the owner gets a letter about it. This is the vandalism.
* **Steal tags**: up to 3 tags, unless the parlor has a safe.
* **Fight** the owner, with the same blows, soul smashes, hit points and first-swing rule as at home. The winner takes BP and up to 4 tags from the loser's pockets. A loser who was attacked goes under police protection, and the attacker's yell or mutter goes in both towns' newspapers.
* **Arson**: hire the Twins (6 gold tags) to burn a parlor in another town. A fire alarm usually saves it; otherwise 5% of its gravestones burn, and all its wood tags unless it has a safe. The gold is only spent if the Twins actually start a fire.

The same protections as at home apply: you can't raid someone who's playing right now, who's under police protection, or whom somebody else is already raiding.

**How it stays fair.** Each step goes to the defender's own server, which acts as referee:

1. The attacker's server sends the step (look, pick lock, steal, fight, swing, run, yell, arson) with the attacker's fighting stats, signed like all network messages.
2. The defender's server runs the normal game rules against its own player and changes only its own player. It answers with numbers (damage, tags taken, BP won), never with screen text.
3. The attacker's server writes the story from those numbers and changes only its own player.

Neither side has to trust the other:

* The defender's server checks that the attacker really is one of the other town's players. It won't accept more BP than that town last reported for them, or strength, hit points or burials more than twice what its own toughest player has.
* The attacker's server won't accept more than the normal maximum: 3 tags for a theft, 4 from a fight, and BP no more than 2% of what the defender's town reported for them.
* Each player can be raided or burned from out of town at most **3 times a day** (`net_raids_per_day`).
* The defender gets a letter naming the attacker and their town every time, and fights and fires make the paper.
* Steps arrive one at a time. If the other town can't be reached, the player is told "the road is closed" and nothing changes on either side.

Turn raids off with **Turn off out-of-town raids** (`net_raids_off`). That stops raids in both directions for your town.

Turn off parts of it with `net_chat_off`, `net_mail_off` and `net_raids_off`. Network data is in `data\net\`. `identity.json` is your server's private key, so keep it with your backups and don't share it.

## Sysop commands

```
fq.exe admin players                      list parlors (shows who's online, bbs/web logons)
fq.exe admin config                       show settings
fq.exe admin config key=value ...         change settings
fq.exe admin maint                        run the nightly maintenance now
fq.exe admin reset [--wipe]               new game (--wipe also deletes accounts)
fq.exe admin delete <name>                close a parlor
fq.exe admin news <text>                  put a notice on today's front page
fq.exe admin sysop-password [pw|off]      web sysop pages
fq.exe admin bans | ban <ip> [min] | unban <ip>
fq.exe admin make-cert [host ...] | cert | https-off
fq.exe rlogin [--port 1513]               run only the RLogin server
fq.exe admin id | links | link <addr:port> | accept <town> | unlink <town> | block-link <town>
fq.exe admin use-theirs <town> [setting ...]   copy a linked town's game settings
fq.exe admin propose                           start a settings vote among all linked towns
fq.exe admin votes | vote <id> yes|no          see and answer settings votes
fq.exe version                                 which version this is
```

A new game day starts at local midnight, on the first action after midnight (the web server also checks every minute).

`data\world.json` is the whole game. To back up, copy the `data` folder. Changes are protected by a lock file (`world.lock`), so any number of door nodes, the web server and the editor can run at once.

## Building from source

The source uses Go 1.24+ and only Go's standard library.

```
build.bat          (Windows)     or     ./build.sh     (Linux, macOS, BSD)
go test ./...      plays thousands of random moves; checks raids, battles, mail, chat, bans,
                   block lists, web security, and two linked towns chatting, writing letters
                   and raiding each other over HTTP (including a town that tries to cheat)
```

Both scripts build every system listed above. To build just one, for example a Raspberry Pi:
`CGO_ENABLED=0 GOOS=linux GOARCH=arm GOARM=7 go build -o fq-linux-arm .` and the same with `./cmd/fqedit`.

## What's ported

Everything from the original server and its scripts:

* customer generation and the four sales tactics, with mood and resistance
* psychology skills, risks, random events, arson, found tags
* Yang's, and the cafe with its slash commands and tag giving
* rival-parlor raids: padlocks, lockpicks, safes, stealing, fighting, police protection
* Spirit University, the bank, Di-Mart (hearses, security, supplies)
* the newspaper and old papers, the Book of Deeds, rankings, letters
* nightly maintenance with top-seller bonuses, and tournaments

## Credits

*Funeral Quest* © 2001 Robinson Technologies: Seth A. Robinson (programming, design), Akiko Robinson (art), Greg Smith (additional design). The code and Akiko Robinson's artwork are used under the **Proton SDK license** (`LICENSE-ProtonSDK.txt`). The sounds and music come from the original game's Flash files; the source release licenses them for use in Funeral Quest only, so don't reuse them in other projects. The browser JPEG XL decoder is @jsquash/jxl (libjxl, Apache-2.0).
