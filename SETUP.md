# Setting Up Attacca DMX & Ossia DMX on a Fresh Raspbian Image

## Common Steps

These steps are common to both configurations. Follow these steps, and then follow the steps specific to the configuration you'd like to set up.

1. Flash the latest image of Raspbian Lite from raspberrypi.org to your MicroSD card.

1. Add an empty file called `ssh` to the `/boot` directory of the card.

1. If you're setting up Ossia DMX, make these changes in `/boot` as well:

	1. Copy over a `wpa_supplicant` configuration.

	1. Remove `console=serial0,115200` from `cmdline.txt` (see <https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=146291#p978159>).

	1. Add `dtoverlay=spi1-1cs` to `config.txt`.

1. Eject the card and insert it into your Pi

1. If you're setting up Attacca DMX, plug in your USB sound dongle.

1. Connect power to the system.

1. Log in to your Pi over SSH as user `pi` with password `raspberry`.

1. Create the directory `~/.ssh`.

1. Inside `~/.ssh`, create the file `authorized_keys` and add your SSH key.

1. Run `sudo su` and repeat the last two steps.

1. Reboot and confirm that your SSH key works.

1. Uncomment or add this line in `/etc/ssh/sshd_config`:

		PermitRootLogin prohibit-password

1. Run `sudo raspi-config` and make these changes:

	1. Change your password.

	1. Set your hostname.

	1. Change your localization and timezone (if needed).

	1. Set video memory allocation to the minimum of 16 MB.

	1. If you're setting up Ossia DMX, enable SPI.

1. Reboot.

1. Run `sudo apt-get update && sudo apt-get upgrade`.

1. Run `sudo apt-get install ola`.

1. Disable unneeded plugins in OLA (e.g., at <http://sfx-player.local:9090>). You can do this by navigating to **New UI** (from the bottom of the page). From the **Plugins** tab, uncheck all plugins except the following:

	- **E1.31 (sACN)** or **ArtNet**, depending on your preferred DMX input
	- **GPIO** (for Ossia DMX only)
	- **SPI** (for Ossia DMX only)

	If you need any of the other plugins, of course, leave them enabled.

1. Proceed to the appropriate section below.


## Steps for Attacca DMX

1. Create the directories `/home/pi/music` and `/home/pi/playlists`.

1. Run `chgrp audio /home/pi/music /home/pi/playlists`.

1. Copy `asound.conf` to `/etc`.

1. Reboot.

1. Run `sudo apt-get install mpd mpc bc shairport-sync`. Leave out `shairport-sync` if you don't need the Pi to act as an Apple AirPlay audio receiver.

1. If you installed `shairport-sync`, you may want to copy `shairport-sync.conf` to `/etc` (overwriting the existing file if present). Feel free to customize this file.

1. Run `sudo systemctl stop mpd && sudo systemctl disable mpd`.

1. Copy these files to `/home/pi`:

		mpd@6600.conf
		mpd@6601.conf
		mpd@6602.conf
		olat_mpd.trigger
		olat_mpd.environment
		olat_mpd@6600.environment
		olat_mpd@6601.environment
		olat_mpd@6602.environment
		olat_sys.trigger
		olat_sys.environment
		olat_sys@1.environment
		olat_sys@2.environment

1. Copy `olat_*` control scripts and the binary `ws` to `/usr/local/bin`.

	Note: This binary of `ws` was compiled on Raspbian and is different from the one found at <https://github.com/yhat/ws/releases/download/v0.1/ws_linux_arm.zip>. See <https://github.com/yhat/ws> for source and other binaries.

1. Run `sudo chmod a+x /usr/local/bin/olat_* /usr/local/bin/ws`.

1. Copy these files to `/etc/systemd/system`:

		mpd@.service
		mpd@.socket
		mpd.target
		olat_mpd@.service
		olat_mpd.target
		olat_sys@.service
		olat_sys.target
		olat.target

1. Create these directories in `/etc/systemd/system`:

		olat_mpd.target.d
		olat_sys.target.d

1. Copy these files to `/etc/systemd/system/olat_mpd.target.d`:

		6600.conf
		6601.conf
		6602.conf

1. Copy these files to `/etc/systemd/system/olat_sys.target.d`:

		1.conf
		2.conf

1. At this point, you may add more instances of `olat_mpd@.service` or `olat_sys@.service` in the manner above. Just make sure there are matching `*.trigger`, `*.environment`, and (for `olat_mpd@.service`) `mpd@*.conf` files in `/home/pi`.

1. Run `sudo systemctl daemon-reload`. (Also do this whenever you've changed the `systemd` drop-ins mentioned in the last 3 steps.)

1. Set up your DMX universes in OLA (e.g., at <http://sfx-player.local:9090/>). This guide assumes you'll set up at least 2 universes. All the `olat_mpd@.service` daemons, as well as `olat_sys@1.service`, require Universe 1. `olat_sys@2.service` requires Universe 2. These requirements are configured in the various `.environment` files copied to `/home/pi`. Adjust them as needed.

1. Run `sudo systemctl enable olat.target`.

1. Reboot.


## Steps for Ossia DMX

1. Copy `wait.conf` to `/etc/systemd/system/dhcpcd.d`. It forces `dhcpcd` to wait up to 60 seconds for a DHCP lease, as `olad` will be useless if the Pi has no IP address. This is basically the **Wait for network at boot** feature enabled in `raspi-config`, but with a specific timeout. Alternatively, you could set a static IP address for the Pi.

1. Copy `ola-gpio.service` to `/etc/systemd/system`.

1. Run `sudo systemctl daemon-reload`.

1. Run `sudo systemctl stop olad`.

1. Run `sudo adduser olad gpio`.

1. Run `echo 'SUBSYSTEM=="spidev", MODE="0666"' | sudo tee -a /etc/udev/rules.d/99-spi.rules > /dev/null` (see <https://opendmx.net/index.php/OLA_Device_Specific_Configuration#SPI>).

1. Copy `ola-gpio.conf` to `/etc/ola`. In this file, you may adjust the DMX values to trigger a GPIO pin on/off. You may also change the `gpio_slot_offset` to change the start channel for these pins. Do not touch the `gpio_pins` line. It is set to GPIO connectors soldered to the header (23 and 24, if you followed the standard design for an Ossia DMX Pi). For details on this file, see <https://github.com/OpenLightingProject/ola/tree/master/plugins/gpio/README.md>.

1. Copy `ola-spi.conf` to `/etc/ola`. This is a sample configuration file. It sets up 3 segments of 2 APA102 pixels each on SPI0, as well as 2 segments of 2 APA102 pixels each on SPI1. For details on this file, see <https://github.com/OpenLightingProject/ola/tree/master/plugins/spi/README.md>.

1. Reboot.