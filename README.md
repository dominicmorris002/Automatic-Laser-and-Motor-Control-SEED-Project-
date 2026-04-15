# Automatic Laser and Motor Control (SEED Project)

A Raspberry Pi 4 based system that controls two stepper motors via a TMC5072 motor driver over SPI. A live camera feed is displayed in a GUI window — clicking anywhere on the image automatically moves the motors to the corresponding X/Y position on the physical stage. Designed for use with DNA chip imaging and precision positioning.

---

## Hardware Required

- Raspberry Pi 4 (running Raspberry Pi OS / Debian)
- TMC5072 dual stepper motor driver board
- 2x stepper motors
- Raspberry Pi Camera Module (or USB webcam)
- Ethernet cable (for connecting Pi to laptop)
- 5V 3A USB-C power adapter for the Pi
- Windows laptop with XLaunch / VcXsrv installed

---

## Wiring — Raspberry Pi 4 to TMC5072

The TMC5072 communicates over SPI. Connect it to the Pi's SPI0 pins:

| TMC5072 Pin | Raspberry Pi 4 Pin | GPIO |
|-------------|-------------------|------|
| SDI (MOSI)  | Pin 19            | GPIO10 |
| SDO (MISO)  | Pin 21            | GPIO9  |
| SCK (CLK)   | Pin 23            | GPIO11 |
| CSN (CS)    | Pin 24            | GPIO8  |
| GND         | Pin 6 (or any GND)| GND  |
| VCC         | External 5V supply | —   |

> ⚠️ The TMC5072 motor supply voltage (VM) must come from an external power supply — do NOT power it from the Pi's 5V pin. Motor current draw will damage the Pi.

Connect Motor 1 to the TMC5072's Motor 1 output and Motor 2 to Motor 2. Refer to your TMC5072 board's silkscreen for exact motor coil pinout (A1, A2, B1, B2).

---
## Connect To Raspberry PI Via RealVNC Viewer

<img width="888" height="630" alt="image" src="https://github.com/user-attachments/assets/770c62ac-e85f-4839-87f7-99a7a00d521a" />

<img width="1045" height="1009" alt="image" src="https://github.com/user-attachments/assets/76150070-bbea-428d-a0b1-24e7a47957f0" />


## One-Time Pi Setup (do this once after flashing a new OS)

### 1. Enable SPI on the Pi

SSH into the Pi and run:

```bash
sudo raspi-config
```

Navigate to: **Interface Options → SPI → Yes → Finish**

Then reboot:

```bash
sudo reboot
```

### 2. Share Internet from your Windows Laptop to the Pi

The Pi connects to your laptop via ethernet. To give it internet access:

1. On your Windows laptop press `Win + R`, type `ncpa.cpl`, hit Enter
2. Right-click your **WiFi** connection → Properties → Sharing tab
3. Check **"Allow other network users to connect through this computer's Internet connection"**
4. In the dropdown select your **Ethernet** adapter → OK

### 3. Install Dependencies on the Pi

SSH in and run:

```bash
pip install opencv-python pillow --break-system-packages
pip install pillow --upgrade --break-system-packages
sudo apt install python3-pil.imagetk -y
```

### 4. Copy the Project Files to the Pi

From your Windows laptop (in a new terminal):

```bash
scp -r "C:\Users\lab\Downloads\BaseControll\SEED project" pi@raspberrypi.local:~/
```

Password: `raspberry`

---

## Running the Program (Every Session)

### On your Windows laptop:

1. Launch **XLaunch** (VcXsrv) — click Next through all defaults
2. Open **Command Prompt** and run `ipconfig` — note your IPv4 address (e.g. `10.69.236.246`)

### In your SSH terminal on the Pi:

```bash
ssh pi@raspberrypi.local
password: pi
export DISPLAY=10.69.236.246:0.0    # replace with your laptop's IP
cd "SEED project"
python autocontrol.py
```

The GUI window will appear on your Windows screen showing the DNA chip image. Click anywhere on the image to move the motors to that position.

> 💡 **Note:** Your laptop's IP address may change each session on a school/office network. Always run `ipconfig` on your laptop first and update the `export DISPLAY` line accordingly.

---

## One-Command Startup (after initial setup)

Create a file called `start_seed.bat` on your Windows Desktop with this content:

```bat
@echo off
for /f "tokens=2 delims=:" %%a in ('ipconfig ^| findstr "IPv4" ^| findstr "10.69"') do set IP=%%a
set IP=%IP: =%
ssh pi@raspberrypi.local "echo %IP% > ~/display_ip.txt && bash ~/start.sh"
```

And create `~/start.sh` on the Pi:

```bash
#!/bin/bash
IP=$(cat ~/display_ip.txt)
export DISPLAY=$IP:0.0
cd "/home/pi/SEED project"
python autocontrol.py
```

Make it executable:

```bash
chmod +x ~/start.sh
```

Then each session just double-click `start_seed.bat` on your Desktop!

---

## Key Files

| File | Purpose |
|------|---------|
| `autocontrol.py` | Main program — GUI, camera feed, click-to-move logic |
| `tmc5072.py` | Driver library for the TMC5072 motor controller |
| `tmc5072.ini` | Motor configuration (speed, acceleration, microstepping) |
| `DNA_Chip.jpeg` | Reference image displayed in the GUI |
| `xy_coordinate.txt` | Log of all clicked coordinates |

---

## Motor Configuration

Configured in `tmc5072.ini`. Key values:

- `vmax = 400000` — maximum motor velocity
- `amax = 2000` — acceleration
- `full_step = 51200` — microsteps per full revolution
- `micro_step = 142` — microsteps per 1 degree of movement

Adjust these in `tmc5072.ini` to tune motor speed and precision for your specific motors.

---

## Troubleshooting

**"Could not resolve hostname raspberrypi.local"** — Run `ssh pi@raspberrypi.local` again; the first attempt sometimes fails on cold boot. Make sure the ethernet cable is plugged into both the Pi and your laptop.

**GUI window doesn't appear** — Make sure XLaunch is running on your Windows laptop and that the `export DISPLAY` IP matches your laptop's current IP (`ipconfig` to check).

**SPI / FileNotFoundError** — SPI is not enabled. Run `sudo raspi-config` → Interface Options → SPI → Yes, then reboot.

**Motor doesn't move but GUI works** — Check your wiring to the TMC5072 (especially CSN/CS pin) and confirm `spi_device = 0` and `spi_cs = 0` in `autocontrol.py`.
