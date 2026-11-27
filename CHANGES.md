# Changes

Releases are numbered major.minor.patch. `fq version` (or `fqedit --version`) shows which one you have, and linked towns compare versions automatically.

## 1.8.2 (2026-11-27)

* **Every picture checked against what's happening, and seven new ones drawn.** Each picture the game shows (sales, risks, random events, letters, the graveyard, the church, raids, fires) was compared with the text it goes with. Where nothing fit, new art was made in the game's own style, mostly from the game's own drawings:
  * **Keying a rival's hearse** now shows that hearse with "LOSER" scratched down the side (it used to show a dog dragging a hand out of a grave).
  * **The dust storm** event shows your parlor lost in blowing dust, tumbleweed and all (it showed the parlor on a clear day).
  * **The graveyard** got its own cast: the **ghost of Old Mortimer** in the mausoleum, a **ghoul** with glowing eyes to hunt, **Digby the gravedigger** (who is also the groundskeeper who catches grave robbers), an **open grave** with shovel and lantern for digging, and a **sea captain's treasure chest** for the big find. The bony hand shows the skeleton, a curse shows the ghost, headstones are read by night, and tending the graves shows fresh roses.
  * **Raising the dead gone wrong** ("You raised something all right!") shows the ghoul.
  * **Fire letters** show the fire when the fire did damage; the smoke detector picture is kept for when the detector saved the day.
  * Random events with no picture of their own got fitting ones: the software upgrade and the computer game at your desk, forgotten taxes with money flying off, "being watched" with the prowler, charging kids to see a body with a money bag.
  * Retiring shows the parlor your heir is taking over.
* If you've edited `gamedata/risks.txt` or `gamedata/events.txt`, copy your own files back after upgrading and, if you want the new pictures, change: the "key hearse" `success_url` to `flash/hearse/hearse4_keyed.swf`, the "cast spell" `fail_url` to `flash/people/ghoul.swf`, and the duststorm event's `set_pic` to `flash/places/duststorm.swf`.

## 1.8.1 (2026-11-27)

* **Every screen fits the window, with nothing to scroll.** The web game no longer makes you scroll to reach the buttons or to read a message. The picture sizes itself to the window, and when a message is long the picture gets smaller to make room for it (and on short screens the side panel packs itself tighter), so the whole message and the buttons are always in view. The text box is only as big as it needs to be, with the buttons right underneath. The only things that still scroll are the ones that can't fit any other way, inside their own box: the cafe's chat log, a very long newspaper page and long ranking lists. Checked on every screen at 1280x720, 1366x768, 1120x873 and 1920x1080.
  * **Stats** is a compact two-column sheet instead of one long list.
  * **Challenges** opens as its own full-width page (and a full page in the door) instead of squeezing into the text box.
  * Rankings, Rival parlors' list, the challenges page and the newspaper hide the empty text box under them, so the newspaper gets the whole screen; the side panel and title bar are tighter.
  * The **log-on page** fits too: smaller artwork and a two-column "New mortician" form.
  * Phones and very small windows still scroll normally.
* **Fixed:** letters with no picture of their own (the top-seller and top-parlor letters, letters from other players and from linked towns, returned mail) showed a broken picture. They now show the desk, and the top-seller and top-parlor letters show a prize picture.
* **Fixed:** on the town map, the name that pops up when you point at a building was cut off for buildings at the top edge (Spirit University, the Graveyard). Names now show under buildings along the top, and line up with buildings at the left and right edges, so they always stay on the map.

## 1.8.0 (2026-11-26)

**Upgrading is safe:** the first time 1.8.0 starts, it zips your whole `data` folder into `data/backups/fq-before-upgrade-1.8.0-<date>.zip`, then adds the new fields to your players. Nothing is removed or reset: names, logons, passwords, Bury Points, tags, cash, stats, rankings, letters and settings all stay as they were. New settings get their defaults. (Going back to 1.7 later? Restore that backup: an older version would quietly drop levels and challenges when it saves.)

* **Levels 1 to 60.** Players earn experience from every Bury Point they earn and from weekly challenges, and climb from level 1 to 60. A new title every 10 levels, shown in rankings, the stats screen and the side panel: Gravedigger, Undertaker (10), Embalmer (20), Mortician (30), Funeral Director (40), Master Mortician (50), Living Legend (60).
  * Each level pays gold tags (more at higher levels) and a full heal. Every 10 levels the player has time for one more customer a day. Reaching a new title makes the paper (and linked towns' papers).
  * **Leveling pace** (setting, 100 = normal): playing a few times a week reaches level 60 in roughly two to three months. 50 is twice as fast, 200 twice as slow.
  * **Catching up:** players below the town's average level earn 10% more experience per level behind, up to double, so latecomers and returning players close the gap.
  * **Fair fights:** nobody can break into, fight or burn the parlor of a player more than 15 levels below them ("Raid level gap"; 0 turns it off). Raiding up is always fine. This also applies between linked towns on 1.8.0, and to AI players.
  * **Existing players** start at the level their Bury Points earn, up to level 40, so veterans get a head start but still have to earn the top titles.
* **Retire and start a dynasty.** At level 60 a Retire button appears at the desk. Retiring puts the parlor in the **Hall of Legends** (the graveyard's "Legends" button, and under the rankings on the log-on page), announces it in the paper, and the player carries on as their heir: "Earl Hatch Jr.", then "III", "IV" and so on, at level 1 in the same parlor with the same logon, plus a family heirloom they pick:
  * The family appointment book: +2 turns every day (up to 3 times)
  * Grandpa's psychology notes: start one psychology level higher (up to 2)
  * The family hearse: each heir keeps the hearse
  * The old padlock and smoke detector: each heir starts with both
  * The family ledger: +2% profit on every sale (up to 3 times)
  * Lucky rabbit's foot: luck never drops below 30

  Heirlooms add up from generation to generation. Players who'd rather not retire can keep going as a Living Legend.
* **Weekly challenges.** Every Monday each player gets three challenges, picked from: bury customers, make sales close to the customer's limit, answer the ghost's riddle, beat the ghoul, dig up graves, tend the graves, win fights, steal tags, donate at church, talk in the cafe, buy tags at the bank. Each one pays experience and a gold tag, and all three pay a bonus (experience, 2 gold tags and 5 luck). See them, and your level progress, under **Challenges** at the desk. "No weekly challenges" turns them off.
* **Rested turns.** Turns you don't use carry over, and every day you miss adds a day's turns, up to 3 days' worth ("Rested days"; 0 = the original rule, unused turns are lost). Players who can't play every day still get a full session.
* **Optional seasons** ("Season length", off by default). Every so many days the town's top three by Bury Points earned that season get bonus experience and a place in the Book of Deeds, the winner gets a season trophy, and the season race starts over. City Hall gets a Season standings button. Seasons never reset levels, Bury Points or dynasties.
* **Automatic backups.** Every week (setting "Backup every", Security tab) the data folder is zipped into `data/backups`. The last 8 are kept ("Backups to keep"); backups made before an upgrade or by hand are never deleted. Back up any time with **Back up now** (web admin, News & maintenance tab, which also lists the backups), `fq admin backup`, or fqedit's News and maintenance menu. `fq admin backups` lists them.
* The admin player list shows levels, and the player editor can set a player's experience (which sets the level).
* Graveyard buttons got shorter names so they all fit on a door caller's screen.
* **Long lists scroll.** Rankings, network rankings, Rival parlors' list of parlors to visit, and the rankings on the log-on page now scroll inside their own box with a visible scroll bar, while the title and column headings stay in place. The box grows to fit the window. In the admin pages, the Players, Bans and Network tables do the same. (Door callers already get long lists a page at a time with "more".)

## 1.7.0 (2026-11-26)

* **Network news.** Newspaper stories are now printed in every linked town's Funeral Times, marked with the town they came from (e.g. `[Gravesend] Bobby makes a perfect sale...`). That covers announcements (players' and AI players'), crimes, fires, perfect sales, new parlors, name changes, the graveyard robber, City Hall notices and tournament winners. Each town's random daily flavor stories stay at home, and a fight with a visitor from another town is reported once, by the visitor's town.
  * It works with towns that can only connect out: stories go both ways in every sync, whichever town makes it.
  * Each story is printed once, even when two towns sync at the same moment, and stories aren't passed on second-hand (every linked town gets them from the source).
  * New setting "Turn off network news" (Network tab) keeps your paper to yourself and stops printing other towns' stories.
* **Rival parlors shows every linked town.** "Look" in Rival parlors now lists the parlors in every linked town you can raid right now, together with your own town's, biggest first, with the town each one is in. (Visiting someone in another town by name already worked; now you can see who's there.) Towns that can't be reached at the moment, and players under police protection in your town, are left off.
* **A bigger, sharper town map that fits the window.** The map picture lost the black border and frame baked into it (the page draws its own frame now), so the town itself is wider. It's redrawn at 1.4 times the size, sharpened, with slightly richer color. It never grows taller than the browser window, the big empty text box under it is now one line ("Where to?"), and the buttons sit right below, so the whole map and every button fit at 100% zoom. The click spots were moved to match.
* Browsers now pick up new pictures and page updates right away after an upgrade, instead of showing cached old ones for up to a day.
* **Fixed:** color codes showed as text (like `` `w ``) in the side panel of the church and graveyard in the web game. The panel now always shows plain text.

## 1.6.0 (2026-11-26)

* **AI players.** Computer-run parlors live in town (3 by default, up to 8; setting "AI players"). Each town makes up its own: random names (Mabel Toombs, Otis Bonesteel, Ima Plotz...) running randomly named parlors (Dead & Breakfast, Six Feet Under Inc, Graves & Sons...), each with one of eight personalities for what they say in the cafe and the paper. They do what players do: sell funerals, buy tags, padlocks, smoke detectors, safes and lockpicks, train at Spirit University, break into parlors, steal tags, fight, now and then hire the Twins, talk in the Pale Horse Cafe, and put announcements in the Funeral Times. Players can raid them back, and an AI player remembers who broke in last and goes after them.
  * **They're kept weaker than real players:** they work only part of a day ("AI skill", 70% of a day's turns), sell like a player with no psychology training, never get the daily top-seller or top-parlor bonuses, never train past the town's average human player, raid at most once a day, only raid parlors at least half their size, and each player can be raided by AI players at most once a day. Beaten AI players stay down for the rest of the day.
  * "AI players don't attack" and "AI players stay quiet" turn off their raids and their talking. Lowering "AI players" closes the extra AI parlors (raising it again reopens them with their old scores).
  * They're never "online", can't be logged into, and show as "(AI player)" in the admin player lists. They move every 8 to 30 minutes while the game is running (web, telnet, or a door caller playing).
* **Cash is worth something.** The original gave every parlor starting money that nothing used.
  * **Net profit from each sale:** after operating costs and insurance, a funeral home keeps about 6 to 15 percent of the bill (the real-world range), so each sale pays a random profit in that range as cash, plus 2 percent for a sale close to the customer's limit. The sale screen shows the profit and its percent. Settings: "Lowest profit per sale" and "Highest profit per sale".
  * **The bank sells tags for cash:** wood $500, silver $1,000, gold $2,000 by default ("Tag prices at the bank"; 0 stops sales of that kind).
  * Cash shows on the stats screen, beside the Bury Points in the web game, and on the door's side panel.

* **The church does something.** In the original, Father Smyth only said hi. Now:
  * **Karma counts.** Stealing tags, hiring the Twins (here or in a linked town) and digging up graves cost karma ("Karma lost per bad deed", 2 by default).
  * **Donate** wood, silver or gold tags for 1, 2 or 4 karma, up to a daily limit (8 by default).
  * **Confession**, once a day for players below 50 karma: costs wood tags (2), gives back karma (10, never above 50).
  * **Blessing**, once a day for players with good karma (70+): adds luck (3).
* **The graveyard is open.** Click the cemetery on the town map (or press G in the door). The original's graveyard button was "removed until I have a use for it"; here are the uses:
  * **Dig up a grave**: wood, silver or gold tags, or rarely a sea captain's chest with gold and Bury Points. Or nothing, the groundskeeper (pay him off with a tag or lose luck, and maybe make the paper), a bony hand (hit points), or a curse for players with wicked karma. Luck helps. Digging costs karma.
  * **Explore the mausoleum**: the ghost of Old Mortimer asks a riddle. Right answer: Bury Points and luck. Wrong: luck goes down.
  * **Hunt the ghoul**: strength and luck decide it. Win Bury Points, karma and sometimes a gold tag; lose and you lose hit points (never down to 0).
  * **Help the gravedigger**: a few wood tags, sometimes a silver one.
  * These four cost a turn each, up to "Graveyard quests per day" (4 by default; 0 closes the graveyard).
  * Free: **tend the graves** once a day (+1 karma, sometimes a dropped tag), **read headstones** (one in eight is an "X marks the spot" hint for your next dig), and **biggest plots**, the parlors that have filled the most graves.
* New settings group **Church & graveyard** in the web admin, fqedit and `fq admin`. These settings are shared rules, so linked towns compare them and can vote on them. Towns updated from 1.5.0 get the defaults.
* The "Casket points" and "Plot points" settings now say they have no effect. They belong to the original's first, simple sale; the sale the original later switched to (and this port uses) scores by price and the trade bonus.
* Towns still on 1.5.0 are told to update, since 1.6.0 has new shared settings.

## 1.5.0 (2026-11-26)

Windows and Linux builds, and a source archive. (macOS and BSD: build from the source with ./build.sh.)

* **Towns that can't be reached can be raided.** An outbound-only town now fetches raid steps from the towns it links to, keeping a waiting request open, the same way it gets chat. When someone breaks in, picks a lock, steals, fights or sets a fire there, the step reaches it within a fraction of a second. It referees the step with its own rules on its own player and sends the result back. Outbound-only towns are now full members of the network while they're online; they're only marked "outbound only and not online right now" when their server is off.
* **A wrong web address no longer breaks a link.** Every town now fetches its raid steps (not only towns with no address), so a town whose address doesn't work from somewhere still gets chat, letters, rankings and raids there. The Network tab says which address isn't working and what to check.
* **A separate network port with the web server.** `fq web` now opens the **Network port** setting too (before, only `fq telnet`/`fq net` did). It's a port just for other Funeral Quest servers, so towns whose web game sits behind Caddy or another proxy can link without routing the network through it.
  * **"Linked servers use the network port only"** (a setting, a checkbox on the Network tab, and a switch in fqedit). The address given to other towns switches to the network port by itself, the web port stops answering other servers, and linked towns are told the new address.
  * **"HTTPS on the network port"**: the certificate is optional there. The default is plain HTTP (messages are signed either way); turn it on to use the town's certificate.
* **The web game and admin pages work in a folder** behind a proxy, e.g. `https://example.com/funeralquest/` with Caddy's `handle_path`. Linking to a town at such an address (with or without the trailing slash) works for syncing, real-time chat and raids. The README has Caddyfile examples.
* **Introductions only pass on addresses that work.** A town only introduces towns it has actually reached in the last 30 minutes, so a mistyped or unreachable address isn't spread around the network.

## 1.4.0 (2026-11-26)

Windows and Linux builds. (macOS and BSD: build from the included source with ./build.sh, or use 1.3.0.)

* **Introductions: the network fills itself in.** Linked towns tell each other which other towns they're linked to. When you link to one town in a network, your town sends link requests to the rest. Towns that follow introductions (the normal setting) accept each other by themselves, so every town ends up linked to every other with nobody clicking anything.
  * Linking checks that each introduced town answers with exactly the ID it was introduced with, so a lying introducer can't send you to an impostor.
  * A town you removed or blocked is never brought back by an introduction.
  * New settings: "Don't follow introductions", "Don't introduce my links", and "Accept introduced towns automatically" (for towns that connect out only).
  * Link requests show which linked town introduced them. Introductions waiting to be followed are listed on the Network tab and in `fq admin links`.

The 1.3.1 fixes below were never released on their own; they're part of 1.4.0.

## 1.3.1 (folded into 1.4.0)

* **Fixed: linking to a town that uses HTTPS** (for example with a self-signed certificate) failed with "bad answer from the other server" unless you typed `https://` in front of its address. Without `http://` or `https://`, the game now tries both.
* **Fixed: a town showed as "not reachable / outbound only"** even though your server was reaching it fine, when that town hadn't set its own web address. A town you can reach is now always shown as reachable, and its players can be raided. (Its sysop should still set **This server's web address** so the towns can reach each other in both directions.)
* The admin Status tab warns when this server is linked but its web address isn't set.
* The "Request came from ..." note no longer stays on towns that were accepted (automatically or by hand).
* **Sturdier network answers.** Every answer between towns now has an exact length (no chunked encoding, which some proxies, firewalls and antivirus web filters break). An answer whose connection drops right after it's complete is accepted when its signature checks out.
* **`fq admin test-link <address>`** checks the way to another town step by step (name lookup, port, Funeral Quest ID, sending a message) and says what's wrong.
* Linking: clearer messages when linking fails. Before linking, the game checks that the address really is a Funeral Quest server. If it isn't, the message shows what answered instead and how to check the address. An answer cut off by a proxy, firewall or antivirus web filter is reported as such, instead of "bad answer from the other server".

## 1.3.0 (2026-11-26)

* **Real-time network chat.** Every linked server keeps a "listen" request open on each of its links, so a line said in one town's Cafe shows up in the others in well under a second (about 0.1 s in testing). It works even when only one side can reach the other. Chat inside a town is real time too: door callers see new lines within a quarter second, and the web page gets them the moment they're said.
* **Version check.** Linked towns exchange their game versions and tell both sysops who's behind and needs to update ("Gravesend runs an older version (1.2.0, you have 1.3.0). Gravesend's sysop needs to update.").
* **Game settings check.** Linked towns exchange their game rules (turns, starting stats, scoring, odds and limits) in every signed handshake. Each sysop sees which settings differ, with "here" and "there" values, and can copy the other town's values one at a time or all at once. You'll find it on the web sysop pages (Status and Network tabs), in fqedit, with `fq admin links`, and in the server log.
* **Settings votes.** A sysop can propose their town's game settings to every linked town. Each other sysop is asked to accept the settings from that town. When more than half of all the towns in the vote agree (the proposing town counts as a yes), every town switches to them. Votes are signed by each town, and every town checks the signatures itself before applying a result. A town can opt out with "Don't apply settings votes". Votes end after 7 days.
* **Auto-accept for links** has its own switch on the web sysop Network tab (it was already a setting, and in fqedit).
* `fq version`, `fq admin propose`, `fq admin votes`, `fq admin vote`, `fq admin use-theirs`.
* **Web admin, complete.**
  * Every setting is on the web admin Settings tab, including the new **Server** group: ports, listen address, HTTPS certificate and key, proxy, and sound. These used to be command-line switches only; the switches still work and override the settings.
  * The game's log-on page has an **Admin** link.
  * First-time admin setup works in the browser with a one-time setup code that only appears on the server.
  * The Status tab shows who's playing.
  * The admin pages can be turned off from the web.
* **Linking without the web server.**
  * `fq telnet` now runs the network link too, and can open a network-only port (`net_port`) for other towns. For BBSes that only run the door, the new `fq net` runs just the network.
  * Towns with no open port at all can link **outbound only**. They get chat, rankings, letters, votes and version and settings checks by connecting out. Other towns list them as "outbound only", and their players can't be raided from other towns.
  * Only one program per town does the network work at a time; the others stand by.
* **RLogin.** Built on the rlogin kit's methods.
  * A BBS can send callers in over RLogin with a gateway secret: Synchronet `rlogin_gate`, Mystic `IR`, or any BBS with outbound RLogin. Players can also connect straight from SyncTERM with their own logon and password.
  * The swapped name/password fields (Synchronet, SyncTERM "RLogin"/"RLogin Reversed", Mystic) are handled by value, so either order works.
  * Only trusted hosts may connect, a name alone is never trusted, and failed log-ons count toward the game's bans.
  * Gateway callers are the same players as door callers from that BBS.
  * It runs in `fq telnet`, or on its own with `fq rlogin`. Setup files are in `extras/rlogin/`.
* **Self-signed HTTPS certificate.** One click on the web admin Security tab, in fqedit, or `fq admin make-cert`. The web address switches to https by itself, and linked towns are told.
* **HTTPS between towns.** Linked servers accept a self-signed certificate the first time and then pin it, refusing a different one later.
* Web game: one request at a time per player session (two fast clicks could step on each other before).

## 1.2.0 (2026-11-25)

* **Raids between linked towns.** Players can break into parlors in other towns, pick their padlocks, steal tags, fight the owner and hire the Twins to burn them out. The defender's own server referees with the normal rules, so linked servers still never change each other's players directly. Both ends cap what the other claims, each player can be raided from out of town at most 3 times a day, and sysops can turn it off (`net_raids_off`).
* **Builds for every system.**
  * Windows: 64-bit, 32-bit and ARM.
  * Linux: x86-64, 32-bit, ARM64 and Raspberry Pi.
  * macOS: Apple Silicon and Intel.
  * FreeBSD (x86-64 and ARM64), OpenBSD and NetBSD.

  fqedit now runs on macOS and the BSDs too. The downloads are split by system.

## 1.1.0 (2026-11-25)

* **fqedit**, a separate full-screen sysop editor (Windows 32/64-bit and Linux). It edits settings, players, game texts, news, maintenance, security, network links, the web sysop password and the logs.
* **Web sysop pages** at `/sysop/`: the same tools in a browser, behind a sysop password.
* **Security**: block lists for IP addresses and host names (`data\blocklist_ip.txt`, `data\blocklist_hosts.txt`) and a never-block list. Automatic temporary bans for telnet attack bots, failed log-ons and connection floods, getting longer each time. The web server is hardened (CSP and security headers, rate limits, timeouts, no directory listings, constant-time log-on, built-in HTTPS, CSRF-proof sysop sessions).
* **Funeral Quest network**: link your town to other servers by address and port. Linked towns share Cafe chat, network rankings and letters. Messages are signed with Ed25519, and linked servers can't change anything on yours.
* Game texts and settings changes take effect in running servers within seconds, without a restart.
* Fixed: "thrown by a house" now says "thrown by a horse".
* Fixed: the picture could stay blank on some screens (seen on a teenage boy customer). The door now checks that the terminal has received each picture before drawing it and re-sends any that didn't arrive. It also redraws the picture after a full-page screen.

## 1.0.0 (2026-11-24)

* First Go release: BBS door (DOOR32.SYS, JPEG XL pictures for SyncTERM with a Sixel fallback, SyncTERM sound), web version with the original Flash look, stand-alone telnet server, Windows 32/64-bit and Linux.
