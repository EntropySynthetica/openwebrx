# Building OpenWebRX Plus with SDRPlay RSP1B Support

This guide documents the steps needed to build and install OpenWebRX Plus with full SDRPlay RSP1B support, running from source to enable custom modifications.

## Why Build from Source?

This setup runs OpenWebRX directly from the repository instead of installing from the PPA for these reasons:

1. **Mobile/Touch Improvements**: The stock OpenWebRX Plus doesn't work well on mobile devices. This setup allows custom modifications for proper mobile support.
2. **Immediate Code Changes**: Running from source means code changes take effect immediately without reinstalling.
3. **Customization**: Maintain custom features like enhanced touch controls, mobile UI improvements, and state persistence.

If you need the stock version without modifications, use the [luarvique PPA](https://luarvique.github.io/ppa/) instead.

## Migrating from PPA Installation

If you previously installed OpenWebRX from the PPA, you may want to stop/disable that version first:

```bash
# Stop the PPA-installed service
sudo systemctl stop openwebrx
sudo systemctl disable openwebrx

# Optionally remove the PPA version (if desired)
# sudo apt-get remove openwebrx

# Note: Keep the connector packages (soapy-connector, etc.) as they're still needed
```

The PPA-installed connectors and dependencies can remain - they're installed system-wide and work fine with the source-based installation.

## Quick Reference

**Starting OpenWebRX manually:**
```bash
cd ~/repos/openwebrx
source env/bin/activate
python openwebrx.py
# Access at http://localhost:8073
```

**Managing the systemd service:**
```bash
sudo systemctl start openwebrx-erica    # Start
sudo systemctl stop openwebrx-erica     # Stop
sudo systemctl restart openwebrx-erica  # Restart
sudo systemctl status openwebrx-erica   # Check status
sudo journalctl -u openwebrx-erica -f   # View logs
```

**After making code changes:**
```bash
# Code changes are immediate when running from repo
# Just restart the service:
sudo systemctl restart openwebrx-erica
```

---

## System Information
- OS: Ubuntu/Debian-based Linux
- SDR Hardware: SDRPlay RSP1B
- OpenWebRX Version: 1.2.106 (OpenWebRX Plus with custom mobile improvements)
- Installation Type: Source-based with virtual environment
- Purpose: Enable mobile device support and custom modifications

**Note:** This guide is for building from a custom fork. If you don't need mobile improvements or custom modifications, consider using the simpler [PPA installation](https://luarvique.github.io/ppa/).

## Prerequisites

### 1. Install Build Tools and Dependencies

```bash
sudo apt-get update
sudo apt-get install -y \
    git \
    cmake \
    build-essential \
    libsoapysdr-dev \
    soapysdr-tools \
    libsamplerate0-dev \
    libfftw3-dev \
    pkg-config \
    python3 \
    python3-setuptools \
    python3-dev \
    debhelper \
    dh-python
```

## Installation Steps

**Note:** The order of building csdr, pycsdr, owrx_connector, and js8py can be flexible as they don't strictly depend on each other. However, they all must be completed before installing OpenWebRX.

### 1. Install SDRPlay API v3

The SDRPlay API is required for the RSP1B to function. Download and install from SDRPlay's website:

```bash
# Download SDRPlay API v3.x from https://www.sdrplay.com/downloads/
# Example (version may vary):
wget https://www.sdrplay.com/software/SDRplay_RSP_API-Linux-3.15.2.run

# Make executable and install
chmod +x SDRplay_RSP_API-Linux-3.15.2.run
sudo ./SDRplay_RSP_API-Linux-3.15.2.run

# Verify installation
ldconfig -p | grep sdrplay
```

**Expected output:** Should show `libsdrplay_api.so` libraries in `/usr/local/lib/`

### 2. Install SoapySDR SDRPlay3 Module

SoapySDR provides the bridge between OpenWebRX and the SDRPlay hardware:

```bash
sudo apt-get install -y soapysdr-module-sdrplay3
```

Verify the installation:
```bash
SoapySDRUtil --find
```

You should see your RSP1B device listed.

### 3. Clone OpenWebRX Plus Repository and Set Up Virtual Environment

Clone your OpenWebRX Plus repository (with your customizations) and create a Python virtual environment:

```bash
cd ~/repos
git clone git@github.com:EntropySynthetica/openwebrx.git
cd openwebrx

# Create a Python virtual environment (recommended to avoid affecting system Python)
python3 -m venv env

# Activate the virtual environment
source env/bin/activate

# Upgrade pip in the venv
pip install --upgrade pip setuptools wheel
```

**Important:** Keep this virtual environment activated for all subsequent build steps. You'll need to activate it whenever you work on OpenWebRX:

```bash
cd ~/repos/openwebrx
source env/bin/activate
```

### 4. Build owrx_connector (SDR Connectors)

The owrx_connector package provides the interface between OpenWebRX and various SDR hardware:

```bash
cd ~/repos
git clone https://github.com/jketterl/owrx_connector.git
cd owrx_connector
mkdir build && cd build
cmake ..
make -j$(nproc)
sudo make install
```

Verify installation:
```bash
ls -l /usr/local/bin/soapy_connector
```

### 5. Build csdr (Signal Processing Library)

csdr is a core dependency for OpenWebRX's signal processing:

```bash
cd ~/repos
git clone https://github.com/luarvique/csdr.git
cd csdr
mkdir build && cd build
cmake ..
make -j$(nproc)
sudo make install
sudo ldconfig
```

Verify installation:
```bash
csdr --version
```

**Expected:** csdr version 0.18.37 or newer

### 6. Build pycsdr (Python Bindings)

pycsdr provides Python bindings for csdr. Install system-wide (required for system access):

```bash
cd ~/repos
git clone https://github.com/luarvique/pycsdr.git
cd pycsdr
python3 setup.py build
sudo python3 setup.py install

# Also install in the venv for development
cd ~/repos/openwebrx
source env/bin/activate
cd ~/repos/pycsdr
pip install .
```

### 7. Build js8py (JS8Call Support)

js8py provides JS8Call decoding capabilities. Install into the virtual environment:

```bash
cd ~/repos
git clone https://github.com/jketterl/js8py.git

# Make sure venv is activated
cd ~/repos/openwebrx
source env/bin/activate

# Install js8py into the venv
cd ~/repos/js8py
pip install .
```

### 8. Install OpenWebRX Plus in the Virtual Environment

With all dependencies built, install OpenWebRX Plus into the virtual environment:

```bash
cd ~/repos/openwebrx

# Make sure venv is activated
source env/bin/activate

# Install OpenWebRX in editable/development mode
# This allows you to edit the code without reinstalling
pip install -e .
```

Verify installation:
```bash
cd ~/repos/openwebrx
source env/bin/activate
which openwebrx  # Should show ~/repos/openwebrx/env/bin/openwebrx
openwebrx --version  # Should show 1.2.106
```

**Note:** When running from the repository with the venv activated, Python uses the source code directly from the `owrx/` directory. This means you can edit the code and see changes immediately without reinstalling.

### 9. Install Optional Decoders and Tools (Recommended)

OpenWebRX Plus supports many additional decoders. Install the ones you need:

```bash
# Digital voice modes
sudo apt-get install -y python3-digiham direwolf

# WSPR, FT8, and other weak signal modes
sudo apt-get install -y wsjtx

# JS8Call
sudo apt-get install -y js8call

# APRS symbols
sudo apt-get install -y aprs-symbols

# Aviation (ADS-B, ACARS, VDL2, HFDL)
sudo apt-get install -y dump1090-fa dump978-fa dumpvdl2 dumphfdl acarsdec

# Additional useful packages
sudo apt-get install -y \
    rtl-433 \
    multimon-ng \
    imagemagick \
    codec2 \
    redsea \
    libhamlib-utils \
    lame
```

**Note:** Some of these packages may need additional repositories or may not be available on all systems.

### 10. Configure OpenWebRX

OpenWebRX configuration is located in `/etc/openwebrx/` when installed, but when running from the repository, you can also edit the config file in your repo:

**System configuration:**
```bash
sudo nano /etc/openwebrx/config_webrx.py
```

**Or use the repo config (for development):**
```bash
nano ~/repos/openwebrx/config_webrx.py
```

Add your SDRPlay device configuration. Example:

```python
sdrs = {
    "sdrplay": {
        "name": "SDRPlay RSP1B",
        "type": "soapy",
        "device": "driver=sdrplay",
        "profiles": {
            "default": {
                "start_freq": 7100000,
                "start_mod": "nfm",
                "center_freq": 7100000,
                "samp_rate": 2400000,
                "rf_gain": 30,
            }
        }
    }
}
```

### 11. Test the Installation

Start OpenWebRX manually from the repository to test:

```bash
cd ~/repos/openwebrx
source env/bin/activate  # Activate the venv
python openwebrx.py

# Or use the installed command:
openwebrx
```

Access the web interface at: `http://localhost:8073`

You should see your SDRPlay RSP1B listed and functional.

**Note:** Both `python openwebrx.py` and `openwebrx` command work when the venv is activated. The `openwebrx.py` script is a simple wrapper that calls the main entry point.

### 12. Set Up Systemd Service (Optional)

A custom systemd service file `openwebrx-erica.service` is included that runs OpenWebRX from the repository:

```bash
# Copy the custom service file
sudo cp ~/repos/openwebrx/openwebrx-erica.service /etc/systemd/system/

# Reload systemd
sudo systemctl daemon-reload

# Enable and start the service
sudo systemctl enable openwebrx-erica
sudo systemctl start openwebrx-erica

# Check status
sudo systemctl status openwebrx-erica

# View logs
sudo journalctl -u openwebrx-erica -f
```

**Service Configuration:**

The `openwebrx-erica.service` file runs OpenWebRX directly from the repository:

```ini
[Unit]
Description=OpenWebRX WebSDR receiver
After=network.target

[Service]
Type=simple
User=erica
Group=erica
WorkingDirectory=/home/erica/repos/openwebrx
ExecStart=/usr/bin/python3 /home/erica/repos/openwebrx/openwebrx.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

This runs the application with system Python but from the repository directory, allowing it to use the source code directly.

**To stop the service:**
```bash
sudo systemctl stop openwebrx-erica
```

**To restart after code changes:**
```bash
sudo systemctl restart openwebrx-erica
```

## Troubleshooting

### Conflicts with PPA Installation

If you previously installed OpenWebRX from the PPA, you may encounter conflicts:

**Port already in use (8073):**
```bash
# Check if the PPA version is still running
sudo systemctl status openwebrx

# Stop and disable it
sudo systemctl stop openwebrx
sudo systemctl disable openwebrx

# Or check what's using the port
sudo lsof -i :8073
```

**Configuration conflicts:**
```bash
# The PPA version uses /etc/openwebrx/config_webrx.py
# Your custom version can use its own config in the repo
# Make sure to edit the correct config file for your setup
```

**Module import conflicts:**
```bash
# If you get import errors, make sure you're using the venv
cd ~/repos/openwebrx
source env/bin/activate
python openwebrx.py
```

### Rebuilding After Code Changes

Since OpenWebRX is running from the repository, most Python code changes are immediately effective:

```bash
cd ~/repos/openwebrx

# Edit your code
nano owrx/some_file.py

# If running manually, just restart it
# Ctrl+C to stop, then:
source env/bin/activate
python openwebrx.py

# If running as a service, restart it:
sudo systemctl restart openwebrx-erica

# Watch the logs to verify:
sudo journalctl -u openwebrx-erica -f
```

**For changes to installed dependencies (csdr, pycsdr, js8py, owrx_connector):**

```bash
# Rebuild the C/C++ components as needed:
cd ~/repos/csdr/build
make -j$(nproc)
sudo make install

# Rebuild owrx_connector if needed:
cd ~/repos/owrx_connector/build  
make -j$(nproc)
sudo make install

# Then restart OpenWebRX:
sudo systemctl restart openwebrx-erica
```

### Device Not Found

Check if SoapySDR can see the device:
```bash
SoapySDRUtil --find="driver=sdrplay"
```

### Permission Issues

Add your user to the plugdev group:
```bash
sudo usermod -a -G plugdev $USER
```
Log out and back in for changes to take effect.

### Check Service Logs

```bash
sudo journalctl -u openwebrx -f
```

### Verify All Components

```bash
# Check csdr
csdr --version
which csdr  # Should show /usr/local/bin/csdr

# Check connectors  
ls -l /usr/local/bin/*connector
# Should show soapy_connector, rtl_tcp_connector, etc.

# Check SoapySDR modules
SoapySDRUtil --info
SoapySDRUtil --find="driver=sdrplay"

# Check Python modules (with venv activated)
cd ~/repos/openwebrx
source env/bin/activate
python -c "import csdr; import pycsdr; import js8py; print('All modules available')"

# Check OpenWebRX installation
which openwebrx  # Should show ~/repos/openwebrx/env/bin/openwebrx
openwebrx --version  # Should show 1.2.106

# Check service status (if running)
systemctl status openwebrx-erica
```

## Package Versions (Reference)

After successful installation, you should have:

- **csdr**: 0.18.37
- **pycsdr**: 0.18.37
- **owrx-connector**: 0.6.5
- **soapy-connector**: 0.6.5
- **openwebrx**: 1.2.106
- **soapysdr-module-sdrplay3**: 0.8.11
- **libsdrplay_api**: 3.15

## Key Repositories

- **Your OpenWebRX Plus Fork**: https://github.com/EntropySynthetica/openwebrx (with custom modifications)
- **Upstream OpenWebRX Plus**: https://github.com/luarvique/openwebrx
- **csdr**: https://github.com/luarvique/csdr
- **pycsdr**: https://github.com/luarvique/pycsdr
- **owrx_connector**: https://github.com/jketterl/owrx_connector
- **js8py**: https://github.com/jketterl/js8py
- **PPA** (optional, for other components): https://luarvique.github.io/ppa/

## Notes

- **This build runs from source for mobile device support** - Stock OpenWebRX Plus from the PPA doesn't work well on mobile devices, so this fork includes custom mobile/touch improvements
- **Migration from PPA**: If you previously used the PPA version, you can keep the connector packages but disable the old service
- **Virtual environment is used** to isolate Python dependencies and avoid system Python conflicts
- **Runs directly from repository** using `python openwebrx.py`, allowing immediate code changes without reinstallation
- **Custom modifications preserved**: Running from your fork preserves mobile UI improvements and other customizations
- The luarvique fork (OpenWebRX Plus) is the upstream source with many enhancements over the original OpenWebRX
- SDRPlay API must be installed separately as it's proprietary software
- The SoapySDR module for SDRPlay3 provides the interface between SoapySDR and the SDRPlay API
- C/C++ components (csdr, owrx_connector) are installed system-wide to `/usr/local/bin/`
- Python packages (js8py, OpenWebRX) are installed in the virtual environment
- pycsdr is installed both system-wide and in venv for compatibility
- The `openwebrx-erica.service` systemd service runs OpenWebRX from the repository directory
- PPA-installed connectors (soapy-connector, rtl-connector, etc.) can coexist with this setup

## Additional Features in OpenWebRX Plus

OpenWebRX Plus (base luarvique fork) includes many additional decoders and features not in the original OpenWebRX:
- AIS, SSTV, FAX, FLEX, POCSAG, HFDL, VDL2, ADSB, ACARS
- CW, RTTY, SITOR-B decoders
- Background decoding with image browser
- Built-in chat between users
- Scanner functionality
- Enhanced maps with aircraft positions, HAM repeaters, broadcast stations

**However, the PPA version has limited mobile/touch support**, which is why this custom fork was created. See the "Custom Modifications" section above for mobile improvements added in this fork.

## Custom Modifications in This Fork

This custom fork (EntropySynthetica/openwebrx) was created to address mobile device compatibility issues with the stock OpenWebRX Plus. Key enhancements include:

### Mobile/Touch Support Improvements
The primary motivation for this fork was that **stock OpenWebRX Plus doesn't work well on mobile devices**. These modifications fix that:

- **Enhanced Mobile/Touch Support**: Major improvements to touch controls for iOS and Android devices
- **Mobile Controls Window**: Dedicated mobile interface with show/hide functionality for easier interaction on small screens
- **Touch Event Handling**: Proper handling of touch events for waterfall interaction, tuning, and frequency selection
- **State Persistence**: Band and waterfall settings are saved between sessions, preserving user preferences
- **Responsive UI Adjustments**: Interface optimizations for smaller screens and portrait/landscape orientations

### Development History
These modifications were developed and tested on February 4, 2026, and have been successfully running on mobile devices since then. The mobile improvements are documented in detail in:
- [MOBILE_TOUCH_IMPROVEMENTS.md](MOBILE_TOUCH_IMPROVEMENTS.md)  
- [MOBILE_TOUCH_FIX_SUMMARY.md](MOBILE_TOUCH_FIX_SUMMARY.md)

See the git commit history for detailed changes:
```bash
cd ~/repos/openwebrx
git log --oneline --grep="mobile\|touch\|iOS" -i
```

### Why This Setup is Necessary
Running from source is essential because:
1. The PPA version lacks mobile optimizations
2. Mobile UI needs iterative testing and refinement
3. Custom JavaScript and CSS changes require immediate testing
4. The systemd service allows running the custom version as a production service

**If you don't need mobile improvements**, consider using the standard PPA installation instead.

## Build Verification

These instructions were verified against an actual build completed on **February 4, 2026**. The build sequence was:

1. **12:31** - Created Python virtual environment in ~/repos/openwebrx/env
2. **12:33** - soapysdr-module-sdrplay3 installed via apt
3. **12:34** - libsamplerate0-dev installed via apt  
4. **12:37** - owrx_connector built and installed to /usr/local/bin/
5. **12:40-41** - csdr built and installed to /usr/local/bin/
6. **12:40** - pycsdr built with setup.py and installed system-wide
7. **12:37** - js8py built and installed into venv
8. **14:19** - openwebrx installed into venv with pip install
9. **20:10** - openwebrx-erica.service started and has been running successfully

**Key Setup Details:**
- Virtual environment: `~/repos/openwebrx/env/`
- C binaries (csdr, connectors): `/usr/local/bin/`
- Python packages: Virtual environment and system-wide (pycsdr)
- Running method: From repository with `python openwebrx.py`
- Service: `openwebrx-erica.service` runs from repo directory
- Configuration: `~/repos/openwebrx/config_webrx.py` and `/etc/openwebrx/`

All components were successfully built and tested with an SDRPlay RSP1B device.
