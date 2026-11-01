# Installing

## Quick start

1. Root the TV and install the Homebrew Channel ([rootmy.tv](https://rootmy.tv)).
2. Install the **CleanYourTV** manager app: from the [Homebrew Channel](#from-the-homebrew-channel), or from your computer with the
   install command and the TV's IP address (below).
3. On the TV, open **CleanYourTV** and choose **Set up now**. It takes about a minute; LG’s Home stays selected. You can switch to CleanYourTV as Home later in the manager app.

That is all. CleanYourTV and its service are set up while LG's Home remains in use. To make the launcher Home, enable **CleanYourTV as Home** in the manager app.

## Requirements

- An LG webOS TV rooted with the [Homebrew Channel](https://webosbrew.org).
- SSH access to the TV as `root`, which the Homebrew Channel can enable (Settings > SSH Server).
- Internet access on the TV (the install command downloads the package from GitHub).
- **webOS 5 or newer** (2020 models and later). webOS 4 and older are not supported: they have no systemd and an old Node.js,
  and the setup stops before it changes anything.
- Firmware from the [tested TVs](tested-tvs.md), or one where LG's Home app is of the flutter or web kind; see
  [What happens on unlisted firmware](#unlisted-firmware).

!!! tip "Give the TV a fixed address"
    Reserve a fixed IP address for the TV in your router so the install command and the network notification address never change.

## Install steps

### 1. Install the manager app

#### From the Homebrew Channel

Add CleanYourTV's repository to the Homebrew Channel once: open the Homebrew Channel, the gear at the top, **Add repository**, and
enter

```text
https://github.com/lasharor/cleanyourtv-WebOS/releases/latest/download/apps.json
```

**CleanYourTV** then shows in the channel's list: open it and choose **Install**. A new release shows there as an update. The
package is the same file as in [Releases](https://github.com/lasharor/cleanyourtv-WebOS/releases), checked against its SHA-256.

#### From your computer

With your TV's IP address:

```sh
ssh root@192.168.1.50 'curl -fsSL https://raw.githubusercontent.com/lasharor/cleanyourtv-WebOS/main/install.sh | sh'
```

The script downloads the latest release package, checks its SHA-256 and installs it. Nothing else on the TV changes in this
step. The package can also be downloaded from [Releases](https://github.com/lasharor/cleanyourtv-WebOS/releases) and installed
with the Homebrew Channel or `ares-install`.

### 2. Set up, on the TV

Open **CleanYourTV** from the app list, choose the options and **Set up now**.

| Option | Default |
|---|---|
| Debloat LG's ads and telemetry | off |
| Take over the Input key | off |

The setup:

1. saves the original state in `/media/developer/cleanyourtv/backup`;
2. installs the launcher and the service;
3. adds boot scripts to `/var/lib/webosbrew/init.d/`;
4. keeps LG's Home selected and starts the service. To replace LG's Home later, enable **CleanYourTV as Home** in the manager app;
5. checks that the service is running. If CleanYourTV is selected as Home, it also checks that the launcher came up; a failed setup removes the boot scripts and restarts with LG's Home.

Its log is `/media/developer/cytv-setup.log`, kept across the restart, with the service log next to it in
`cytv-setup.log.service`.

??? note "The same setup over SSH"
    ```sh
    sh /media/developer/apps/usr/palm/services/com.cleanyourtv.manager.service/bootstrap.sh install
    ```
    Options: `--no-debloat`, `--no-input-key`, `--no-start`, `--no-verify`, `--force`.

## Unlisted firmware

The setup matches the TV's firmware against [`compat.json`](https://github.com/lasharor/cleanyourtv-WebOS/blob/main/compat.json)
(it fetches the newest copy from the repository and falls back to the one in the package). A listed firmware is set up directly.
For an unlisted firmware the setup continues when LG's Home app is the flutter or web kind, and otherwise stops and offers
**Set up anyway**. On a TV whose Home is LG's native program (webOS 5 to 23) the launcher becomes Home through a stand-in, see
[what differs between webOS versions](tested-tvs.md#what-differs-between-webos-versions). To add a firmware to the list, open an issue with the output of **Share log** ([Troubleshooting](troubleshooting.md)).

## Installing a dev build

To try a change that is not released yet, install the dev build: see [Updating and uninstalling](managing.md#dev-builds).

## Troubleshooting

- **The Home button opens LG's full-screen Home, not CleanYourTV.** Some TVs have a Home Style setting. Set TV Settings >
  General > Home Settings > Home Style to Basic.
- **`FAILED_IPKG_INSTALL`.** Remove the `com.cleanyourtv.manager` entry marked `half-installed` from
  `/media/developer/apps/var/lib/opkg/status` and install again.
- **Black screen after switching or rebooting.** Wait a minute; if it stays black, restart the TV.

More in [Troubleshooting](troubleshooting.md).
