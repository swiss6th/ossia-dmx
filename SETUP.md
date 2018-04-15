# Setting Up Ossia DMX on a Fresh Raspbian Image

1. Flash the latest image of Raspbian Lite from raspberrypi.org to your MicroSD card.

1. Add an empty file called `ssh` to the `/boot` directory of the card.

1. Copy over a `wpa_supplicant` configuration.

1. Remove `console=serial0,115200` from `cmdline.txt` (see <https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=146291#p978159>).

1. Add `dtoverlay=spi1-1cs` to `config.txt`.

1. Eject the card and insert it into your Pi

1. Connect power to the system.

1. Log in to your Pi over SSH as user `pi` with password `raspberry`.

1. Create the directory `~/.ssh`.

1. Inside `~/.ssh`, create the file `authorized_keys` and add your SSH key.

1. Run `sudo su`, `cd ~`, and repeat the last two steps.

1. Reboot and confirm that your SSH key works.

1. Uncomment or add this line in `/etc/ssh/sshd_config`:

		PermitRootLogin prohibit-password

1. Run `sudo raspi-config` and make these changes:

	1. Change your password.

	1. Set your hostname.

	1. Change your localization and timezone (if needed).

	1. Set video memory allocation to the minimum of 16 MB.

	1. Enable SPI.

1. Reboot.

1. Run `sudo apt-get update && sudo apt-get upgrade`.

1. Run `sudo apt-get update && sudo apt-get install ola`.

1. Disable unneeded plugins in OLA (e.g., at <http://prop.local:9090>). You can do this by navigating to **New UI** (from the bottom of the page). From the **Plugins** tab, uncheck all plugins except the following:

	- **E1.31 (sACN)** or **ArtNet**, depending on your preferred DMX input
	- **GPIO**
	- **SPI**

	If you need any of the other plugins, of course, leave them enabled.

1. Copy `wait.conf` to `/etc/systemd/system/dhcpcd.service.d`. It forces `dhcpcd` to wait up to 60 seconds for a DHCP lease, as `olad` will be useless if the Pi has no IP address. This is basically the **Wait for network at boot** feature enabled in `raspi-config`, but with a specific timeout. Alternatively, you could set a static IP address for the Pi.

1. Copy `ola-gpio.service` to `/etc/systemd/system`.

1. Run `sudo systemctl daemon-reload`.

1. Run `sudo adduser olad gpio`.

1. Run `sudo systemctl enable ola-gpio`.

1. Run `echo 'SUBSYSTEM=="spidev", MODE="0666"' | sudo tee -a /etc/udev/rules.d/99-spi.rules > /dev/null` (see <https://opendmx.net/index.php/OLA_Device_Specific_Configuration#SPI>).

1. Run `sudo systemctl stop olad`.

1. Copy `ola-gpio.conf` to `/etc/ola`. In this file, you may adjust the DMX values to trigger a GPIO pin on/off. You may also change the `gpio_slot_offset` to change the start channel for these pins. Do not touch the `gpio_pins` line. It is set to GPIO connectors soldered to the header (23 and 24, if you followed the standard design for an Ossia DMX Pi). For details on this file, see <https://github.com/OpenLightingProject/ola/tree/master/plugins/gpio/README.md>.

1. Copy `ola-spi.conf` to `/etc/ola`. This is a sample configuration file. It sets up 3 segments of 2 APA102 pixels each on SPI0, as well as 2 segments of 2 APA102 pixels each on SPI1. For details on this file, see <https://github.com/OpenLightingProject/ola/tree/master/plugins/spi/README.md>.

1. Reboot.