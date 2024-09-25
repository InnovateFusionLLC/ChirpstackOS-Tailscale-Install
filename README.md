# ChirpstackOS-Tailscale-Install

# Tailscale Installation on ChirpstackOS (OpenWRT Variant)

This guide walks you through the process of installing and configuring Tailscale on **ChirpstackOS**, an OpenWRT variant. These steps ensure that Tailscale runs automatically on boot and is properly configured for the platform. 

Refer to the [Chirpstack OS Gatway](https://github.com/chirpstack/chirpstack-gateway-os) for more information on the Gatway OS. 
Refer to the [Tailscale](https://tailscale.com/) website for more information on Tailscale.
You can ind the pgks for Tailscal [HERE](https://pkgs.tailscale.com/stable/?v=latest#static).

## Prerequisites

- A device running **ChirpstackOS**.
- SSH access to the device.
- Basic knowledge of terminal commands.

## Installation Steps

### 1. Update ans upgrade Package Lists

First, ensure that your package lists are up to date:

```bash
opkg update
```

```bash
opkg list-upgradable | cut -f 1 -d ' ' | xargs -r opkg upgrade
```

### 2. Download and Extract the Tailscale Binary

Since Tailscale isn’t available directly from the OpenWRT package manager, we need to manually download and extract it.

1. Navigate to the `/tmp` directory:

   ```bash
   cd /tmp
   ```

2. Download the Tailscale binary (update version "..tailscal_x.xx.x_arm" as needed):

   ```bash
   wget https://pkgs.tailscale.com/stable/tailscale_1.74.1_arm.tgz
   ```

3. Extract the file:

   ```bash
   tar xzf tailscale_1.74.1_arm64.tgz
   ```

4. Move the binaries to a system-wide directory:

   ```bash
   cd tailscale_1.74.1_arm
   cp tailscale tailscaled /usr/sbin/
   ```

### 3. Create the Tailscale Init Script

This script will allow Tailscale to be managed as a service and start automatically at boot.

1. Create the init script:

   ```bash
   nano /etc/init.d/tailscaled
   ```

2. Add the following content:

   ```bash
    #!/bin/sh /etc/rc.common

   START=99
   STOP=10

   start() {
       echo "Starting Tailscale..." >> /tmp/tailscaled.log
       /usr/sbin/tailscaled --tun=userspace-networking --statedir=/etc/tailscale &
   }

   stop() {
       echo "Stopping Tailscale..."
       killall tailscaled
   }

   ```

3. Make the script executable:

   ```bash
   chmod +x /etc/init.d/tailscaled
   ```

### 4. Enable the Tailscale Service

Enable the Tailscale service to ensure it starts at boot:

```bash
/etc/init.d/tailscaled enable
```

### 5. Start Tailscale for the First Time

Manually start Tailscale and log in to authenticate your device:

```bash
/etc/init.d/tailscaled start
```

After the service starts, initiate the login process:

```bash
tailscale up --netfilter-mode=off
```

You will be provided with a URL. Open this URL in your browser and log in with your Tailscale account.

### 6. Verify the Tailscale Service

Check the status of Tailscale to ensure it’s connected to your network:

```bash
tailscale status
```

### 7. Verify Automatic Startup After Reboot

To ensure Tailscale starts automatically on boot, perform a reboot and check if Tailscale is running:

```bash
reboot
```

After the reboot, SSH back into the device and check Tailscale’s status:

```bash
ps | grep tailscaled
tailscale status
```

### 8. Monitor Logs for Issues

If you encounter any issues, monitor the Tailscale log created in `/tmp/tailscaled.log`:

```bash
cat /tmp/tailscaled.log
```

## Optional: Handling DNS Warnings

ChirpstackOS/OpenWRT might not fully support Tailscale’s DNS management. If you see DNS-related health warnings, you can safely ignore them unless they impact functionality.

---

## License

This project is licensed under the MIT License

