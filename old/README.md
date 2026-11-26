# Funeral Quest — BBS door + web server (Go)

Seth A. Robinson's 2001 *Funeral Quest*, ported to **Go** as one program that runs as:

* **a BBS door** using **DOOR32.SYS**, for Windows (32- or 64-bit) and Linux BBSes. **SyncTERM** callers get Akiko Robinson's original artwork as **JPEG XL** pictures inside the ANSI screen.
* **a web server** with a browser version that looks like the original Flash client, also using JPEG XL.

Both modes share the same game data, so BBS callers and web players live in the same town. They share the rankings, the Pale Horse Cafe chat, letters, Yang's deliveries and parlor raids.

No runtimes, no installers, no npm, no database server. It's a single executable plus a folder of data files.

```
fq.exe        Windows 64-bit
fq32.exe      Windows 32-bit (also runs on 64-bit Windows)
fq-linux      Linux x86-64
assets/       pictures: assets/door (JPEG XL + PBM masks + PNG twins), assets/web (JPEG XL)
gamedata/     the original game text files (risks, sales pitches, events, news...) - edit freely
web/public/   the browser page
data/         created on first run: world.json (the game), news/, door.log
```

## The JPEG XL check

Every time a caller enters the door, it asks their terminal what it can do before drawing anything:

1. `CSI 6n` gives the screen size, and `CSI c` tells whether the terminal is SyncTERM (CTerm).
2. **`APC SyncTERM:Q;JXL`** asks whether the terminal can decode JPEG XL. SyncTERM answers `CSI =1;1-n` if it can and `CSI =1;0-n` if not.
3. If JPEG XL is supported, the door lists SyncTERM's picture cache (`APC SyncTERM:C;L`). It uploads each picture only once (`APC SyncTERM:C;S`), and only re-sends one if it changed. It then draws with `APC SyncTERM:C;DrawJXL`, layering customers over backgrounds with PBM masks the way the Flash client did.
4. If there's **no** JPEG XL, the door checks for Sixel (CTerm feature 4, or attribute 4 in the DA1 reply). If the terminal has Sixel, the same art is sent as Sixel. It looks the same but costs about 100 KB per picture instead of roughly 10 KB once.
5. If the terminal has neither, the game is played in text, with a caption in the picture box.

The result for each caller is written to `data\door.log`, for example:
`caller "Skelly" node 1: ... syncterm=true jpegxl=true sixel=false cell=8x16`.

For JPEG XL, callers need **SyncTERM 1.4 or newer built with JPEG XL support**. The screen is the usual 80×25. Pictures fill the top-right 40×14 cells (308×224 pixels with SyncTERM's 8×16 font).

## Install

1. Unzip anywhere, for example `C:\doors\fq`. Keep the folders next to the exe.
2. Check that it runs: `fq.exe admin players` should print `Day 1`.
3. Optionally set it up:
   ```
   fq.exe admin config server_name="My BBS" turns_per_day=20
   ```

### Synchronet (Windows or Linux)

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

### Test it without a BBS

```
fq.exe telnet --port 2323
```

Then connect SyncTERM to `localhost:2323`. Type `NEW` to make a character; the logon name and password also work on the web site.

## Web server

```
fq.exe web --port 8080
```

Then open `http://localhost:8080/`. Run the web server in the **same folder** as the door so both share `data\world.json`. To keep it running on Windows, use Task Scheduler ("At startup") or a service wrapper such as NSSM. For a public site, put an HTTPS proxy such as Caddy or IIS in front of it.

BBS players can also play in a browser: in the door, open **Stats → Web login** to pick a web logon name and password for the same parlor.

Browsers with native JPEG XL (Safari) show the art directly. For other browsers the page includes a small libjxl WebAssembly decoder, so the art is still JPEG XL.

## Sysop commands

```
fq.exe admin players                      list parlors (shows who's online, bbs/web logons)
fq.exe admin config                       show settings
fq.exe admin config key=value ...         change settings
fq.exe admin maint                        run the nightly maintenance now
fq.exe admin reset [--wipe]               new game (--wipe also deletes accounts)
fq.exe admin delete <name>                close a parlor
fq.exe admin news <text>                  put a notice on today's front page
```

The settings match the original server's game options: `turns_per_day`, `visits_per_day`, `risk`, `normal_odds`, `luck_odds`, `script_odds`, `yangs_per_day`, `bonus_exp`, `bonus_luck`, `trade_bonus`, `tags_receivable_per_day`, `name_change_cost`, `inactive_days`, `tourn_active`, `tourn_days`, `max_players`, `server_name` and others. A new game day starts at local midnight, on the first action after midnight (the web server also checks every minute).

`data\world.json` is the whole game. To back up, copy the `data` folder. Changes are protected by a lock file (`world.lock`), so any number of door nodes and the web server can run at once.

## Building from source

The source uses Go 1.24+ and only Go's standard library.

```
build.bat          (Windows)     or     ./build.sh     (Linux/macOS)
go test ./...      plays thousands of random moves and checks raids, battles, mail and chat
```

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

*Funeral Quest* © 2001 Robinson Technologies: Seth A. Robinson (programming, design), Akiko Robinson (art), Greg Smith (additional design). The code and Akiko Robinson's artwork are used under the **Proton SDK license** (`LICENSE-ProtonSDK.txt`). The original sounds and music are not included. The browser JPEG XL decoder is @jsquash/jxl (libjxl, Apache-2.0).
