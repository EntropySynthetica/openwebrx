# Starting and Stopping OpenWebRX with SDRPlay

This guide covers the proper startup and shutdown procedures for OpenWebRX with SDRPlay device support and mobile touch interface improvements.

## Prerequisites

- OpenWebRX built from source with SDRPlay support
- SDRPlay API installed at `/opt/sdrplay_api/`
- Python virtual environment at `env/`
- Mobile touch improvements for better iPhone/iPad support (see MOBILE_TOUCH_IMPROVEMENTS.md)

## Starting OpenWebRX

### 1. Start the SDRPlay API Service

The SDRPlay API service must be running before OpenWebRX can communicate with SDRPlay devices.

**Check if the service is already running:**
```bash
systemctl status sdrplay.service
```

**Start the service if not running:**
```bash
sudo systemctl start sdrplay.service
```

**Verify the service is active:**
```bash
ps aux | grep sdrplay_apiService
```

You should see `/opt/sdrplay_api/sdrplay_apiService` running as root.

### 2. Activate the Python Virtual Environment

Navigate to the OpenWebRX directory and activate the virtual environment:

```bash
cd /home/erica/repos/openwebrx
source env/bin/activate
```

You should see `(env)` prefix in your terminal prompt indicating the virtual environment is active.

### 3. Start OpenWebRX

With the virtual environment activated, run OpenWebRX:

```bash
python openwebrx.py
```

OpenWebRX should now start and be accessible via web browser (typically at http://localhost:8073).

## Stopping OpenWebRX

### 1. Stop OpenWebRX

Press `Ctrl+C` in the terminal where OpenWebRX is running to gracefully stop the application.

### 2. Deactivate the Virtual Environment (Optional)

```bash
deactivate
```

### 3. Stop the SDRPlay API Service (Optional)

If you want to stop the SDRPlay service when not using the SDR:

```bash
sudo systemctl stop sdrplay.service
```

**Note:** The SDRPlay service is configured to restart automatically on failure, so it's generally safe to leave it running.

## Auto-start on Boot (Optional)

### Enable SDRPlay Service

To automatically start the SDRPlay service on system boot:

```bash
sudo systemctl enable sdrplay.service
```

### Create OpenWebRX systemd Service

To run OpenWebRX as a systemd service, create `/etc/systemd/system/openwebrx.service`:

```ini
[Unit]
Description=OpenWebRX SDR Web Server
After=network.target sdrplay.service
Requires=sdrplay.service

[Service]
Type=simple
User=erica
WorkingDirectory=/home/erica/repos/openwebrx
ExecStart=/home/erica/repos/openwebrx/env/bin/python /home/erica/repos/openwebrx/openwebrx.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Then enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable openwebrx.service
sudo systemctl start openwebrx.service
```

Check status:
```bash
sudo systemctl status openwebrx.service
```

View logs:
```bash
journalctl -u openwebrx.service -f
```

## Troubleshooting

### SDRPlay device not detected

1. Check if the SDRPlay service is running:
   ```bash
   systemctl status sdrplay.service
   ```

2. Restart the service:
   ```bash
   sudo systemctl restart sdrplay.service
   ```

### Python module import errors

Make sure the virtual environment is activated:
```bash
source /home/erica/repos/openwebrx/env/bin/activate
```

### Permission issues

The SDRPlay service runs as root. Make sure your user has the necessary permissions to access USB devices.

## Quick Start Commands

For quick reference, here's the typical startup sequence:

```bash
# Start SDRPlay service (if not already running)
sudo systemctl start sdrplay.service

# Navigate to OpenWebRX directory
cd /home/erica/repos/openwebrx

# Activate virtual environment
source env/bin/activate

# Start OpenWebRX
python openwebrx.py
```

To stop, simply press `Ctrl+C` in the OpenWebRX terminal.
