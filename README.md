
# Pacman Backup

Backup tool for off-the-grid updates via portable USB sticks or (mesh) LAN networks.


## Share Updates via USB Drive

In the below example, make sure that `pacman-usbstick` is writable.
Replace the path with the correct one that points to your mounted USB drive.

**Step 1**:

On the machine with internet connection, insert and mount the USB drive.

Use `pacman-backup archive` to copy the package cache to the backup folder.
Use `pacman-backup cleanup` to remain only the latest version of each package.

The `archive` action will also copy the necessary database files for `pacman -Sy`.

```bash
# Machine with internet connection

sudo pacman -Sy;
sudo pacman -Suw;

pacman-backup archive /run/media/$USER/pacman-usbstick;
pacman-backup cleanup /run/media/$USER/pacman-usbstick;
sync;

# Then, unmount the USB drive
```

**Step 2**:

On the machine without internet connection, insert and mount the USB drive.

Copy the necessary database files to pacman's `sync` folder (which is `$DBPath/sync` of `pacman.conf`).

Use `pacman-backup upgrade` to update from the backupfolder.
This will output the pacman command that you should verify manually before executing it.

```bash
# Machine without internet connection

sudo cp /run/media/$USER/pacman-usbstick/sync/*.db /var/lib/pacman/sync/;'
pacman-backup upgrade /run/media/$USER/pacman-usbstick;'
```


## Share Updates via LAN Connection

In the below example, the machine with internet connection has the IP `192.168.0.10`.
Replace the IP with the correct one that matches your setup. If in doubt, use `ip` or `ifconfig`.


**Step 1**:

On the machine with internet connection, connect the LAN cable (and internet connection).

Use `pacman-backup serve` to start a pacman server that can be used by other pacman clients.

```bash
# Machine with internet connection

sudo pacman -Sy;
sudo pacman -Suw;

pacman-backup serve;
```

**Step 2**:

On the machine without internet connection, connect the LAN cable and verify that the server
(the machine with internet connection) is reachable via its IP. If in doubt, use `ping`.

Use `pacman-backup download 192.168.0.10` to download the packages to pacman's package cache.

Use `pacman-backup upgrade` to update from the local pacman cache.
This will output the pacman command that you should verify manually before executing it.

```bash
# Machine without internet connection

sudo pacman-backup download 192.168.0.10;
pacman-backup upgrade;
```

## Share Updates via LAN Cache Proxy

`pacman-backup` can also emulate a local Cache Proxy for other pacman clients.
If `pacman-backup serve` is running on the machine with internet connection, it
can be used for `pacman` directly.

Note that if the packages don't exist, they will appear in the logs but aren't downloaded
directly; and that partial upgrades are not officially supported by Arch Linux.

In the below example, the machine with internet connection has the IP `192.168.0.10`.
Replace the IP with the correct one that matches your setup. If in doubt, use `ip` or `ifconfig`.

**Step 1**:

On the machine with internet connection, connect the LAN cable (and internet connection).

Use `pacman-backup serve` to start a pacman server that can be used by other pacman clients.


```bash
# Machine with internet connection

sudo pacman -Sy;
sudo pacman -Suw;

pacman-backup serve;
```

**Step 2**:

Modify the `/etc/pacman.d/mirrorlist` to have as a first entry the following line:

```conf
# Machine without internet connection
# Pacman Mirrorlist for local server

Server = http://192.168.0.10:15678
```

Use `pacman -Sy` and `pacman -Su` to update from the Cache Proxy.

```bash
# Machine without internet connection

sudo pacman -Sy;
sudo pacman -Su; # or use -Suw
```


## Advanced Usage

### archive

`archive` allows to backup everything to a specified folder. It copies the files from
`/var/cache/pacman/pkg` and `/var/lib/pacman/sync` into `$FOLDER/pkgs` and `$FOLDER/sync`.

```bash
# copy local packages to /target/folder/pkgs
# copy local database to /target/folder/sync

pacman-backup archive /target/folder;
```

### cleanup

`cleanup` allows to cleanup the package cache in a way that only the latest version of
each package is kept (for each architecture).

If no folder is specified, it will clean up `/var/cache/pacman/pkg`.
If a folder is specified, it will clean up `$FOLDER/pkgs`.

```bash
# cleanup /var/cache/pacman/pkg

sudo pacman-backup cleanup;
```

```bash
# cleanup /target/folder/pkgs

pacman-backup cleanup /target/folder;
```

### download

`download` allows to download packages from a `pacman-backup serve` based server.

If no folder is specified, it will download to `/var/cache/pacman/pkg`.
If a folder is specified, it will download to `$FOLDER/pkgs`.

If no server is specified, it generates a download list of package URLs that
you can use for your download manager of choice (e.g. uGet or jdownloader).

```bash
# download packages to /var/cache/pacman/pkg
# download database to /var/lib/pacman/sync

sudo pacman-backup download 1.3.3.7;
```

```bash
# download packages to /target/folder/pkgs
# download database to /target/folder/sync

pacman-backup download 1.3.3.7 /target/folder;
```

```bash
# generate HTTP/S URL list for packages that need downloading

pacman-backup download;
```

### serve

`serve` allows to start a `pacman` server that can be used as a local mirror.

If no folder is specified, it will serve from `/var/cache/pacman/pkg` and `/var/lib/pacman/sync`.
If a folder is specified, it will serve from `$FOLDER/pkgs` and `$FOLDER/sync`.

```bash
# serve packages from /var/cache/pacman/pkg
# serve database from /var/lib/pacman/sync

pacman-backup serve;
```

```bash
# serve packages from /source/folder/pkgs
# serve database from /source/folder/sync

pacman-backup serve /source/folder;
```

### upgrade

`upgrade` allows to generate an executable `pacman` command that uses the specified
cache folder as a package source by leveraging the `--cachedir` parameter.

If no folder is specified, it will upgrade from `/var/cache/pacman/pkg`.
If a folder is specified, it will upgrade from `$FOLDER/pkgs`.

```bash
# generate upgrade command via packages from /var/cache/pacman/pkg

pacman-backup upgrade;
```

```bash
# generate upgrade command via packages from /source/folder/pkgs

pacman-backup upgrade /source/folder;
```

`upgrade` also prints out a command for missing packages that need downloading.

In the scenario that the local database says that more packages need to be downloaded
to update everything, it will output an additional command that is prepared to let
`pacman` download the packages to the specified folder.

This is helpful in the scenario that the "offline machine" has more packages installed
than the "online machine" (so that more packages need to be downloaded to fully
upgrade the "offline machine").

Then you can simply copy/paste the generated command to a text file and execute it
next time you're online - in order to automatically prepare everything that's
necessary for the "offline machine".

