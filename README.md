# ASUS Ultimate Fan Speed

A simple systemd service to set the ASUS laptop fan control mode when the boot process is complete.

This service executes the following command:

- Wait for 2 seconds after the service starts.
- Found all `pwm1_enable` files in `/sys/devices/platform/asus-nb-wmi/hwmon/hwmon*/`.
- Writes the value `0` to each such file.
- Remains active status after command completes via `RemainAfterExit=yes`.

On many ASUS devices, the value `0` in `pwm1_enable` returns fan control to manual mode. This behavior depends on kernel support and the device used.

## Prerequisites

- Linux with `systemd`.
- Kernel containing the ASUS WMI driver (`asus-nb-wmi`).
- File `/sys/devices/platform/asus-nb-wmi/hwmon/hwmon*/pwm1_enable` is available.
- Access `root` or account with `sudo` permissions.

## Installation

Clone this repository

```bash
git clone https://github.com/sdqfrmnsyh/ultimate-fan-speed-asus.git
cd ultimate-fan-speed-asus
```

Copy the service unit to the local systemd directory:

```bash
sudo install -m 644 asus-fan-init.service /etc/systemd/system/asus-fan-init.service
```

Reload the systemd configuration, then enable the service to run every boot:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now asus-fan-init.service
```

`--now` runs the service as soon as it is enabled. If you only want to enable it for the next boot, use:

```bash
sudo systemctl enable asus-fan-init.service
```

## Verify

Check service status:

```bash
systemctl status asus-fan-init.service
```

Services that are successfully running will display the status `active (exited)`. Check if the fan control file is available and the values have changed:

```bash
for f in /sys/devices/platform/asus-nb-wmi/hwmon/hwmon*/pwm1_enable; do 
printf '%s: ' "$f" 
cat "$f"
done
```

The boot service log can be viewed by:

```bash
journalctl -u asus-fan-init.service
```

## Disable or delete

To stop a service from running on the next boot:

```bash
sudo systemctl disable --now asus-fan-init.service
```

To remove an already installed unit:

```bash
sudo rm /etc/systemd/system/asus-fan-init.service
sudo systemctl daemon-reload
```

## Troubleshooting

### Service is successful but the fan does not turn on

Make sure the control path is available:

```bash
ls /sys/devices/platform/asus-nb-wmi/hwmon/hwmon*/pwm1_enable
```

If there are no matching files, the ASUS WMI driver may not be active, the hwmon pathname is different, or the device does not provide such PWM control.

### Service failed on boot

See the full error message:

```bash
systemctl status asus-fan-init.service --no-pager
journalctl -u asus-fan-init.service -b --no-pager
```

### Hardware warning

Manual fan control can cause the temperature to increase if the curve or fan speed is not set correctly. Monitor the device temperature and ensure the fan configuration is appropriate for the ASUS model used.

## License

Lazy🧢