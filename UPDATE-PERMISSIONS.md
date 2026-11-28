# Fixing permissions for updates

An update (the web **Updates** tab, `fq -update`, or typing `update` in the `fq net` window) replaces the game's files in its folder. So it must run as a user who can write to that folder.

**The usual problem:** the game was unpacked or installed as **root** (with `sudo`), but it runs as the BBS's user (`sbbs`, `mystic`, `bbs`, `pi`...). That user can run the game but can't replace its files, so the update stops with an error like *permission denied* or *files couldn't be replaced*.

Your players and settings are not harmed. An update never changes the `data` folder, and if it stops early, **Go back** on the Updates tab (or `fq update undo`) puts the old version back.

## Linux: the fix (do it once)

### 1. Find out which user runs the game

Use whichever fits:

* **It runs as a service** (systemd): look for the `User=` line:
  ```
  systemctl cat funeralquest | grep User=
  ```
  If there's no `User=` line, it runs as root.
* **It's running now:**
  ```
  ps -o user= -C fq-linux
  ```
  (On a Raspberry Pi or 32-bit system, the program may be `fq-linux-arm`, `fq-linux-arm64` or `fq-linux32`.)
* **Your BBS starts the door:** it's the user your BBS runs as. For Synchronet that's often `sbbs`, for Mystic `mystic`, on a Raspberry Pi often `pi`.

### 2. Give that user the game folder

Replace `sbbs` with your user, and `/opt/funeralquest` with your game folder:

```
sudo chown -R sbbs:sbbs /opt/funeralquest
```

That's it. From now on, every way of updating works.

### 3. Check it

This lists anything in the game folder that the user doesn't own:

```
find /opt/funeralquest ! -user sbbs
```

If it prints nothing, updates will work.

## Linux: updating as root instead

If you'd rather keep the folder owned by root, run the update as root:

```
cd /opt/funeralquest
sudo ./fq-linux -update
```

Then give the files back to the BBS user, or the game can't save its data any more:

```
sudo chown -R sbbs:sbbs /opt/funeralquest
```

**Don't skip the second command.** Files the update wrote as root, including the data backup, would belong to root, and the game running as `sbbs` couldn't write them.

Then restart the game (or the `fq net` / `fq web` / `fq telnet` service) so it runs the new version.

## Windows

Run the update as the Windows user that installed Funeral Quest. If the game is in `C:\Program Files`, either:

* move the game folder somewhere your user owns (for example `C:\BBS\FuneralQuest`), or
* give your user **Full control** of the folder: right-click the folder → **Properties** → **Security** → **Edit** → pick your user → tick **Full control** → **OK**.

If `fq` runs as a Windows service, the service's account (shown in **Services** → the service → **Log On**) needs Full control of the folder too.

## Still stuck?

| Message | What to do |
|---|---|
| `permission denied` or `files couldn't be replaced` | The folder isn't writable by the user running the update. Do steps 1 and 2 above. |
| `couldn't back up the data folder first` | The `data` folder isn't writable by that user either. Do steps 1 and 2 above. |
| `text file busy` (Linux) | Another program has the file open. Stop `fq net`, `fq web` and `fq telnet`, update with `fq -update`, then start them again. |
| The game can't save after an update as root | You missed the second command under "Updating as root instead": run the `chown` now. |
| Something went wrong halfway | Use **Go back** on the Updates tab, or `fq update undo`. The data folder is never changed by an update. |
