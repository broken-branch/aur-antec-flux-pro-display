# antec-flux-pro-display for Arch Linux

Arch package for [antec-flux-pro-display](https://github.com/Reikooters/antec-flux-pro-display),
which shows CPU and GPU temperatures on the side-panel display of the Antec Flux Pro
case. Arch Linux and Arch-based distributions only: Arch, Omarchy, CachyOS, EndeavourOS,
Manjaro, etc.

## Install

From the AUR, with an AUR helper or by hand:

```sh
git clone https://aur.archlinux.org/antec-flux-pro-display.git
cd antec-flux-pro-display
makepkg -si
```

The same files are at <https://github.com/broken-branch/aur-antec-flux-pro-display>.
If your Rust comes from rustup.rs rather than the `rust` package, use `makepkg -d -f`
and install the result with `sudo pacman -U antec-flux-pro-display-[0-9]*.pkg.tar.zst`
(the pattern skips the debug package makepkg also builds).

The service starts whenever the display is present, so there is nothing to enable,
and installing the package normally starts it on the spot. If `systemctl status
antec-flux-pro-display` says it is inactive, run:

```sh
sudo udevadm trigger --action=add --attr-match=idVendor=2022 --attr-match=idProduct=0522
```

## Configure

`/etc/antec-flux-pro-display/config.conf` names the two sensors. The defaults suit an
AMD CPU and GPU; the comments cover Intel and the case where two devices are both
called `amdgpu`. `antec-flux-pro-display --list-sensors` shows what is available.
Restart the service after editing:

```sh
sudo systemctl restart antec-flux-pro-display
```

## If only one temperature shows

The case has a mode button, the rightmost one on the top I/O strip, labelled "Temp".
It cycles CPU & GPU, CPU, GPU, off.

## Troubleshooting

- `systemctl status antec-flux-pro-display` and `journalctl -u antec-flux-pro-display -b`.
  At start the journal names the chip and label chosen for each sensor; after that it
  only logs changes.
- A configuration error exits with status 2 and the unit stays failed until you
  restart it after fixing the file.
- `lsusb -d 2022:0522` (package `usbutils`) shows whether the display is connected
  at all.
- "connected but cannot be opened" means the USB node does not carry the daemon's
  group yet; run the `udevadm trigger` line above.

## Remove

```sh
sudo systemctl stop antec-flux-pro-display
sudo pacman -Rs antec-flux-pro-display
```

`-Rs` keeps an edited config as `config.conf.pacsave`; `-Rns` deletes it. The
`antec-flux-pro-display` system user stays, as pacman never removes users; `sudo
userdel antec-flux-pro-display` if you want it gone. The display's USB node keeps that
group until the display is replugged. The panel blanks a few seconds after the last
frame.

## For AI agents

What is not obvious from the files:

- The display is a HID device with only an OUT endpoint, so it never gets a hidraw
  node and is driven over libusb. Do not look for a kernel driver.
- The service has no `[Install]` section on purpose: the udev rule starts and stops it
  with the device. `systemctl enable` refuses; `mask` turns it off.
- The unit runs as a static user from `sysusers.d`, because the udev rule assigns the
  USB node to that user's group whenever the display appears, service running or not.
  The rule must stay numbered below 99 so `99-systemd.rules` sees its tag.
- The unit does not set `PrivateNetwork=`: libusb learns of hotplug through a netlink
  socket and its device list goes stale without one.
- The `000?-*.patch` series is upstream's code with fixes intended for upstream; none
  has been submitted yet. Rebase on new releases; drop each once merged.
- To verify a change: the service is `active`, the journal shows two "using" lines
  and nothing after, `systemd-analyze security` still says "safe", and unplugging
  the display stops the service while replugging starts it.

## License

Packaging files: [0BSD](https://spdx.org/licenses/0BSD.html). The daemon is GPL-3.0-only.
