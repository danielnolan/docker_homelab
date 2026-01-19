# OpenThread Border Router (OTBR) Setup

This Docker Compose configuration sets up an OpenThread Border Router for Thread devices with Home Assistant.

## Prerequisites

### Hardware
- Nabu Casa ZBT-2 (or compatible Thread radio)
- ZBT-2 must be flashed with OpenThread RCP firmware

### System Requirements
- Docker with host networking support
- IPv6 enabled on host system
- Home Assistant running in Docker container

### Flash ZBT-2 Firmware (if needed)

Visit the Open Home Foundation firmware tool to flash the Thread RCP firmware:
https://connectzbt2.home-assistant.io/

## Setup Steps

### 1. Verify Device Path

After connecting ZBT-2 to your server:

```bash
ls /dev/serial/by-id/usb-Nabu_Casa_ZBT-2_*
```

The compose.yaml is configured for device `usb-Nabu_Casa_ZBT-2_9C139EAC0ED8-if00`. Update if yours differs.

### 2. Enable IPv6 Forwarding

Download and run the host setup script from OpenThread:

```bash
wget https://raw.githubusercontent.com/openthread/ot-br-posix/refs/heads/main/etc/docker/border-router/setup-host
chmod +x setup-host
```

Edit the script to replace the network interface name with your actual interface (`enp87s0`), then run:

```bash
sudo ./setup-host
```

### 3. Create Data Directory

```bash
mkdir -p ~/docker/otbr
```

### 4. Start the Containers

```bash
cd ~/docker_compose/otbr
docker compose up -d
```

### 5. Check Logs

```bash
docker compose logs -f otbr
```

Look for successful OTBR initialization messages.

## Home Assistant Integration

### Add Thread Integration
1. Go to **Settings → Devices & Services**
2. Click **Add Integration**
3. Search for **Open Thread Border Router**
4. Enter URL: `http://127.0.0.1:8081`

### Set Preferred Border Router
1. Go to **Settings → Integrations → Thread → Settings**
2. Set this OTBR as preferred

## Mobile Device Setup (Thread Credentials)

To use Thread devices from your phone:
1. Open Home Assistant companion app
2. Navigate to **Settings → Integrations → Thread → Settings**
3. Select "Send credentials to phone"

**For Android:** Go to Companion App → Troubleshooting → Sync Thread credentials (run twice)

## Troubleshooting

### Check Thread Status
```bash
docker compose exec otbr ot-ctl state
```
Should return: `leader` or `router`

### View Thread Network Data
```bash
docker compose exec otbr ot-ctl dataset active
```

### Factory Reset OTBR
```bash
docker compose exec otbr ot-ctl factoryreset
```

### Verify Border Router Visibility
```bash
avahi-browse -rt _meshcop._udp
```

### Container won't start
- Check if the USB device path is correct
- Ensure the device is not being used by another process
- Verify the radio has RCP firmware

### Thread credentials won't sync to phone
- Restart Home Assistant container
- Access Home Assistant via local IP (not reverse proxy) initially
- Run sync twice on Android

## Configuration Files

- `compose.yaml` - Docker Compose configuration
- `.env` - Environment variables (Traefik hostname, timezone)

## Data Persistence

- OTBR data: `~/docker/otbr/`

## Access

- OTBR REST API: `http://localhost:8081`
- Via Traefik: `https://otbr.iot.hummingbirdholler.house`

## Resources

- [Community Thread - ZBT-2 Setup Guide](https://community.home-assistant.io/t/connect-zbt-2-thread-to-home-assistant-container/954156)
- [OpenThread Border Router Guide](https://openthread.io/guides/border-router)
- [Home Assistant Thread Integration](https://www.home-assistant.io/integrations/thread/)
