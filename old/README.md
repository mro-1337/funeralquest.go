# Funeral Quest — BBS door + web server (Go)

Version 1.9.3. **New here? Start with `QUICK-INSTALL.md`.** `HELP.txt` lists every command and setting. See `CHANGES.md` for what's new; `fq version` shows the version you're running. The same guides are in `docs/` as web pages you can open in any browser.

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

The downloads are split by system: a Windows `.zip`, a Linux `.tar.gz` (unpack with `tar -xzf`), and a source `.zip`. For macOS and the BSDs, build from the source archive (`./build.sh`, needs Go 1.24+). Each one is complete: the programs for that system plus all the art, sounds, texts and source code. The examples below use `fq.exe`; on other systems use the matching name, e.g. `./fq-linux`. On Unix-like systems you may need `chmod +x fq-* fqedit-*` first. On macOS, the programs aren't signed by Apple, so the first time run `xattr -d com.apple.quarantine fq-macos-* fqedit-macos-*` (or right-click → Open).

```
assets/       pictures: assets/door (JPEG XL + PBM masks + PNG twins), assets/web (JPEG XL)
              sound: assets/sound (the original game's sounds and music as Ogg Vorbis)
gamedata/     the original game text files (risks, sales pitches, events, news...) - edit freely
web/public/   the browser page
data/         created on first run: world.json (the game), news/, logs, block lists, net/ (links), tls/
HELP.txt      plain-text help: every command and setting (opens in Notepad)
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

### E-mail addresses, forgotten passwords and notes for the sysop

* **Sign-up asks for an e-mail address** (web and telnet), only so a forgotten password can be sent back. Players who signed up before (and BBS players who set a web logon) are asked once each time they play until they give one; they can skip it. They can change it under **Stats → E-mail**. Door players who only play through the BBS are never asked.
* **Forgot your password?** (under the web log-on, or type `FORGOT` at the telnet logon prompt) lets a player leave the sysop a note: their logon name, how to reach them, and anything that proves the parlor is theirs.
* **Contact the sysop** (the link under the web log-on, or `CONTACT` at the telnet prompt) takes a name, an e-mail address and a message.
* Both are behind a **"prove you're among the living"** puzzle: letters and numbers in different colors carved on a gravestone (on telnet, big colored block letters). Each puzzle works once; wrong answers count toward a temporary ban, and one address can leave at most 4 notes an hour.
* **Every note is saved as a text file** in `data/operator`, named after when it arrived: `2026-11-27_21-35-01.operator.msg.txt`.
* **You're told about new notes:** the web admin pages show them on the Status tab and in the new **Mailbox** tab, and the window of a running `fq web`, `fq telnet` or `fq net` prints `*** New note for the operator ... Push R and Enter to read it.` Typing `r` and Enter shows the new notes. `fq admin notes` lists them all.
* **Answering a forgotten password:** passwords are stored scrambled, so nobody (the sysop included) can look one up. Set a new one, then e-mail it to the player. On the **Mailbox** tab: type the new password, **Set new password**, then **E-mail them** opens your mail program with the message written out. From the command line: `fq admin password <logon> <new password>`. A forgotten-password note shows which parlor the logon name belongs to and the e-mail address on file, so you can check the request is genuine before answering. (The person asking is never told whether the logon name exists.)

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

### Behind Caddy (or another proxy)

**Simple step-by-step guide: `CADDY.md`** (including what to do when your town shows as "outbound only").

Caddy can handle HTTPS for you, with a free certificate it renews by itself, and pass requests on to `fq web`. Run `fq web` on a local port (e.g. 8080), set **Behind my own proxy** to Yes (Settings → Server) so bans hit the real caller, and use one of these Caddyfiles:

**Its own (sub)domain** (`https://fq.example.com/`):
```
fq.example.com {
    reverse_proxy 127.0.0.1:8080
}
```

**A folder on your website** (`https://example.com/funeralquest/`):
```
example.com {
    handle_path /funeralquest/* {
        reverse_proxy 127.0.0.1:8080
    }
    # ... the rest of your site
}
```
Use `handle_path`, not `handle`: it removes `/funeralquest` before passing the request on. The game, the admin pages and the network link all work inside a folder.

Then set **This server's web address** to the address the world uses: `https://fq.example.com` or `https://example.com/funeralquest`. **Don't** use the local port (`:8080`), which only Caddy can reach. Other towns link to you with that same address.

**Or keep linking out of Caddy altogether, with a network port:**

1. Settings → Server → **Network port** = e.g. `8090`, then restart `fq web`. It opens a second port that only answers other Funeral Quest servers; the game isn't served there.
2. Forward that port on your router to the Funeral Quest machine.
3. Settings → Network → **Linked servers use the network port only** = Yes. It's also a checkbox on the Network tab and a switch in fqedit → Network links.
   * The address given to other towns becomes your host name on the network port, e.g. `http://tiestudio.ca:8090`. Just set **This server's web address** to your host name (any address on that host will do).
   * The web port stops answering other servers, so linking never goes through Caddy.
   * Towns already linked to you are told the new address and switch by themselves.

The certificate is optional on the network port. By default it uses plain HTTP, which is fine because messages between towns are signed and can't be faked or changed. Set **HTTPS on the network port** (Settings → Server) to use your certificate there too, which also keeps letters private on the way.

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

**Hiding the sysop pages a little.** Settings → Security → **Hide the Admin link** takes the Admin link off the web log-on page (the pages still work at `/sysop`), and **Puzzle with the admin password** adds a gravestone puzzle to the sysop log-on.

**Linked towns are never banned.** The addresses your linked towns use (where their signed messages come from, and what their addresses' host names point to) are never temporarily banned, and a ban already on one is lifted within a few minutes. So if a sysop mistypes their password on a friend's game, the two towns stay connected. The block lists still apply: blocking an address there is your own choice.

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

* **Real-time Cafe chat**: what's said in your Pale Horse Cafe shows up in theirs a split second later as `[Deathville] Alice: hi`, and the other way around. The cafe keeps a **chat wall**: walking in shows the last 30 lines of talk from the past two days. `/net` in the cafe lists linked towns and who's on.
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
| `fq web` | yes | the web port, or the **network port** if you set one |
| `fq telnet` | yes | the **network port** (`net_port` setting or `--net-port`), if you set one |
| only the door | run `fq net` alongside it (e.g. at startup) | the network port, if you set one (`fq net --port 8081`) |

Only one program per town does the network work at a time. If you run `fq web` and `fq telnet` together, one does it and the other stands by, taking over if the first one stops.

**Outbound only.** With no network port, and no web server reachable from outside, a town connects out only. It works behind a firewall or router with nothing forwarded. Leave **This server's web address** empty and link to the other towns by their address; nobody can link to you by address. An outbound-only town still gets everything by asking on each connection:

* real-time Cafe chat, both ways
* network rankings, letters both ways, settings votes, version and settings checks
* raids both ways: its players can raid other towns, and **other towns' players can raid it too**

**How raids get in.** Nobody can connect to an outbound-only town, so it fetches its visits instead. It keeps a waiting request open on every town it's linked to. When a player there breaks in, picks a lock, steals, fights or hires the Twins, that town queues the step. The outbound-only town picks it up at once, referees it with its own rules on its own player, and sends the result back. To players it feels the same as any other town; each step takes a fraction of a second.

While it's online and fetching, other towns treat it as fully linked. If its server is off, it's shown as **outbound only and not online right now**, and its players can't be visited until it's back. A town that normally can be reached but currently can't be is shown as **not reachable**.

**Night raids.** When a town can't be reached right now (an outbound-only town that's offline, or a town that's down), its players still show in **Rival parlors**, marked **night raid**. A night raid is planned in one go: pick steal, fight or arson (arson is paid up front). The plan is saved and handed to that town the next time it fetches. That town runs the whole raid on its own player with its own rules, and the result (spoils, letters to both players, the news) comes back on a later fetch. If the town doesn't fetch within 3 days, the raid is called off and anything paid is refunded.

**Two outbound-only towns.** They can never reach each other, so a town that can be reached passes their messages on. If Gravesend and Boneyard both connect out only and are both linked to Deathville, Deathville introduces them to each other (the Network tab shows the address as **through Deathville**), and from then on everything between them goes through Deathville: rankings and Rival parlors, chat, letters, news, settings votes and live raids. Messages stay signed by the town that wrote them, so the town in the middle can't read anything it couldn't see anyway, or fake or change anything. Live raids need both towns online at the time; otherwise the other town's players are offered for a night raid. A town that doesn't want to pass on other towns' messages can turn on **Don't pass on messages for other towns** (Settings → Network).

**Door-only BBSes (nothing else running).** Without `fq net`, `fq web` or `fq telnet` running, the door itself checks in with the linked towns: when a caller comes in (in the background, while the welcome screen shows) and when they leave ("Checking in with linked towns..."). That sends and fetches players, chat, letters, news and night raids. For a town that's busy only now and then, also add a **timed event** to your BBS that runs `fq net --once` every 10 to 15 minutes (Synchronet: SCFG → External Programs → Timed Events; Mystic: Event Editor; Linux: cron). It checks in once and exits. If `fq net`, `fq web` or `fq telnet` is running, these check-ins step aside, since that program already does it.

To be fully connectable without the web game, set **Network port** (`net_port`, Settings → Server) and forward it on your router. Then set **This server's web address** to `http://your.address:thatport`.

### How linked servers talk

All network traffic goes to these addresses on the web port (or the network port). Door nodes don't connect to other servers for chat or mail; they share the `data` folder with whichever program runs the network.

| Address | What for | When |
|---|---|---|
| `/fed/hello` | link requests | when a sysop links or accepts |
| `/fed/sync` | player lists, chat, letters, newspaper stories, game version and settings, settings votes | every 30 seconds, and within moments of new mail |
| `/fed/listen` | real-time chat: held open until someone in that town speaks (up to 25 s), then asked again at once | all the time |
| `/fed/raid` | one step of a raid | as the raider plays |

There's no central hub. Every pair of towns links directly, but you don't have to link to each one yourself: **introductions** fill in the network (below).

### Introductions: joining a whole network with one link

Linked towns tell each other which other towns they're linked to. If you link to town 5, and town 5 is linked to towns 2, 3 and 4:

1. Town 5 **introduces** 2, 3 and 4 to you, and introduces you to them.
2. Your town sends each of them a link request, and they send you one.
3. Towns that follow introductions accept each other by themselves. A few moments later, every town is linked directly to every other, and chat, rankings, letters, raids and votes work between all of them.

You only ever type in one address. The Network tab lists introduced towns that are still being linked, and `fq admin links` shows them too. Requests from introduced towns say who introduced them.

**Settings** (Settings → Network):
* **Don't follow introductions** (`net_intro_off`): link only to towns you add yourself. Introduced towns are listed for you to add by hand.
* **Don't introduce my links** (`net_share_links_off`): don't tell linked towns which other towns you're linked to.
* **Accept introduced towns automatically** (`net_accept_introduced`): accept a link request from any town that one of your linked towns is linked to, without asking. Two towns that both follow introductions link up anyway. This matters for towns that connect out only, since nobody can send *them* a request.

**Safety:**
* An introduction comes in a signed message from a town you already trust.
* Linking checks that the introduced town answers with exactly the ID it was introduced with, so a lying introducer can't point you at an impostor.
* Towns you **removed or blocked** are never brought back by an introduction. To link one again, add it by address yourself.
* Two outbound-only towns can't link to each other, since neither can be reached. Each still links with every reachable town.

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

### Sysops of linked towns: edit reports and messages

The people who run linked towns can keep an eye on each other and talk:

* **Player edits are reported.** Whenever a sysop changes a player by hand (on the web admin pages, in fqedit or with `fq admin`: their points, money, tags, password, or deleting them), every linked town's sysop gets a **Player edited** note in their **Mailbox** (web admin pages): when it happened, which player, what changed from what to what, which town did it and with what program. So nobody can quietly boost their own parlor in the shared rankings. Your own edits are kept in your Mailbox too, already read.
* **Write to another sysop.** The Mailbox tab has a **Write to another sysop** form: pick one linked town or all of them. Messages from other sysops arrive in your Mailbox with a **Reply** button.
* These notes travel with the regular sync, both ways, so towns that connect out only get them too; they're sent again until the other town has them. The console of a running fq says when one arrives ("Push R and Enter to read it").

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

Raids into a town that connects out only work the same way: it fetches the steps from the raider's town (see **Outbound only** above).

Turn raids off with **Turn off out-of-town raids** (`net_raids_off`). That stops raids in both directions for your town.

Turn off parts of it with `net_chat_off`, `net_mail_off` and `net_raids_off`. Network data is in `data\net\`. `identity.json` is your server's private key, so keep it with your backups and don't share it.

## Sysop commands

```
fq.exe admin players                      list parlors (shows who's online, bbs/web logons)
fq.exe admin config                       show settings
fq.exe admin config key=value ...         change settings
fq.exe admin maint                        run the nightly maintenance now
fq.exe admin backup | backups             zip the data folder now | list the backups
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

`data\world.json` is the whole game. The game backs up the `data` folder by itself every week (see below); you can also just copy the folder. Changes are protected by a lock file (`world.lock`), so any number of door nodes, the web server and the editor can run at once.

## Running on a small server

Funeral Quest is light: it typically needs 30–60 MB of memory and very little CPU, so a $5 VPS or a Raspberry Pi runs it fine. The game only rewrites its data file when something actually changed, which spares SD cards and slow disks. If your server is very small or has many linked towns, these settings (Settings → **Performance**) help further:

* **Sync with linked towns every (seconds):** 30 normally. 60–300 means fewer requests with many linked towns. Your own chat and letters still go out right away; other towns' news and players arrive a little later.
* **No live chat connections:** normally one connection per linked town stays open so their chat appears in the cafe at once. Turned on, chat from other towns arrives with each sync instead, and those connections (and their memory) go away.
* **Memory limit (MB):** for example 64 or 128 on a tiny VPS. fq tidies up its memory sooner to stay near it (a little more CPU, less RAM). Takes effect when fq starts.

Other settings that help: **AI players** (Settings → AI players: fewer or 0), **Don't pass on messages for other towns** (Network) if your town is a hub for many outbound-only towns, and keeping fewer weekly backups (Security).

## Backups and upgrades

Every week (setting "Backup every", Security tab, 7 days by default) the game zips the `data` folder (players, settings, news, network keys, ban lists) into `data/backups/fq-backup-<date>.zip` and keeps the newest 8 ("Backups to keep"). Before a new version changes your data for the first time, it saves `fq-before-upgrade-<version>-<date>.zip`; those, and backups made by hand, are never deleted.

* Back up now: web admin, News & maintenance tab, **Back up now** (the tab lists the backups too); `fq admin backup`; or fqedit, News and maintenance.
* List them: `fq admin backups`.
* Restore one: stop every Funeral Quest program, unzip the backup into the `data` folder (replacing what's there), and start again.

Upgrading never resets players or settings: new settings get their defaults and new player fields are filled in (from 1.8.0 on, every parlor upgraded from 1.7 or older gets the level its Bury Points earn, up to 40). To go back to an older version, restore the backup made before the upgrade.

### Updating from the web admin pages

From 1.9.0 on, the game can update itself from the project's [Releases page on GitHub](https://github.com/mro-1337/funeralquest.go/releases).

* Once a day, `fq web` (and `fq telnet`, `fq net`, `fq rlogin`) asks GitHub whether there's a newer release. When there is, the web admin **Status** tab says so, and the log says so too.
* The **Updates** tab shows the installed version, the newest release and what's new in it. **Install** does the rest:
  1. downloads the Windows or Linux package and checks it isn't damaged;
  2. backs up the data folder (`data/backups/fq-before-update-<version>-<date>.zip`, never deleted);
  3. keeps a copy of every file it replaces (in `data/update`), then puts the new version in place. The `data` folder is never touched. A `gamedata` text you've edited (in the Game texts tab or by hand) is left alone, and the new one is saved beside it as `.new` to compare;
  4. restarts the web server on the new version. Web players have to log on again. BBS door callers get the new version on their next call. Other running copies (`fq telnet`, `fq rlogin`) restart once nobody is online; `fq net` restarts right away.
* **Go back to the previous version** puts the old files back and restarts. The data folder isn't changed; if the newer version changed it and you want it back too, restore the before-update backup.
* Without the web server:
  * `fq -update` checks GitHub, downloads and installs the newest version in one go (no questions).
  * In the window of a running `fq net` (the usual shim for door-only BBSes), `fq telnet` or `fq web`, type **`update`** and press Enter: it installs the newest version and restarts itself on it. `check` only looks, `undo` goes back to the previous version, `help` lists the commands.
  * `fq update --check` says whether there's a newer version, `fq update` installs it after asking, and `fq update undo` goes back.
* Settings, **Updates** group: **Don't check for updates**; **Get updates from** (another GitHub project, `owner/name`); **My service manager restarts it**: if fq runs under NSSM, as a Windows service or under systemd, turn this on and fq exits with code 3 after an update instead of restarting itself, so the service manager starts the new version. (On Linux, restarting itself keeps the same process, so systemd is fine either way.)
* **Automatic updates (off unless you turn them on):** Settings → Updates → **Install updates automatically**. fq then looks for a new version every 6 hours, downloads it right away (even while people are playing), and installs it once nobody has been playing for a minute, then restarts on it. Only one running copy of fq does the installing; the others restart onto the new version once nobody is online. The data folder is backed up first and the Updates tab's **Go back** works as usual.
* **Permission errors when updating** (usually on Linux, when the game was installed as root but runs as the BBS user): see **`UPDATE-PERMISSIONS.md`** for the one-line fix.
* Ready-made updates are for the Windows and Linux programs. On macOS or BSD, build the new version from the source package.

**For whoever publishes releases:** the updater looks at the newest release on the Releases page (not files committed to the code). Each release needs a tag that's the version number with a `v` (`v1.9.1`), and the packages attached with their usual names: `FuneralQuest-1.9.1-Windows.zip` and `FuneralQuest-1.9.1-Linux.tar.gz` (and the source zip).

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

New in this port (the original left these unfinished):

* **The church** does something now. Karma goes down for bad deeds (stealing tags, hiring the Twins, digging up graves, in town or in a linked town) and back up by donating tags, or by a once-a-day confession for players below 50 karma. Players with good karma can ask for a daily blessing that adds luck.
* **The graveyard** (the cemetery on the town map; the original's graveyard button was "removed until I have a use for it"). Quests up to a daily limit (they don't cost turns): dig up a grave for treasure (or a curse, the groundskeeper, or a bony hand), answer the ghost's riddle in the mausoleum, hunt the ghoul, help the gravedigger for tags. Free: tend the graves once a day for karma, read headstones (one of them is a treasure hint), and see which parlors have filled the most plots.
* All of it is set under **Church & graveyard** in the settings, and linked towns compare these settings like the other game rules.
* **The hospital:** hit points carry from fight to fight (at 0 you can't raid), and the hospital patches you up for gold tags, sells morgue tips for extra customers, lets you volunteer for karma, and has visiting hours for rivals who lost a fight (flowers, or unplugging something). Settings are under **Hospital**.
* **Cash.** After operating costs and insurance, players keep a net profit of 6 to 15 percent of each funeral's bill (like real funeral homes; a sale close to the customer's limit adds 2 percent), and the bank sells tags for cash at prices the sysop sets.
* **Levels and dynasties:** players climb from level 1 to 60 (titles every 10 levels, gold tags each level, an extra daily turn every 10), with catch-up experience for players behind the town's average and no raiding anyone more than 15 levels below you. At 60 they can retire into the Hall of Legends and carry on as their heir ("Name Jr.", "III"...) with a family heirloom. Weekly challenges, rested turns (unused turns carry over up to 3 days) and optional seasons keep it going for people who don't play every day. Settings are under **Levels**.
* **Automatic backups:** the data folder is zipped into `data/backups` every week and before every upgrade (see Backups and upgrades).
* **Network news:** linked towns print each other's newspaper stories (announcements, crimes, fires, big sales, new parlors...), marked with the town's name. "Turn off network news" on the Network tab stops it.
* **AI players** (3 by default, up to 8): computer-run parlors that sell funerals, shop, raid and burn parlors, talk in the cafe and advertise in the paper, and can be raided back. They're deliberately weaker than real players: part-time work, no daily bonuses, no training past the average human player, one raid a day, and each player raided by AI players at most once a day. Settings are under **AI players**; set "AI players" to 0 for none.

## Credits

*Funeral Quest* © 2001 Robinson Technologies: Seth A. Robinson (programming, design), Akiko Robinson (art), Greg Smith (additional design). The code and Akiko Robinson's artwork are used under the **Proton SDK license** (`LICENSE-ProtonSDK.txt`). The sounds and music come from the original game's Flash files; the source release licenses them for use in Funeral Quest only, so don't reuse them in other projects. The browser JPEG XL decoder is @jsquash/jxl (libjxl, Apache-2.0).
