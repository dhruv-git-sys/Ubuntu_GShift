# 🚀 Enabling G-Mode on Dell G15 5530 (Linux)

G-Mode lets you crank the fans to max speed for better cooling — perfect for **intense workloads or gaming**.  

This guide explains how to:
- Install required modules  
- Write a toggle script  
- Add a desktop shortcut for one-click usage  

---

## 📑 Table of Contents
1. [Installing Required Libraries](#step-1-installing-required-libraries)  
2. [Choosing a Script Location](#step-2-choosing-a-script-location)  
3. [Creating the Script](#step-3-creating-the-script)  
4. [Adding a Menu Shortcut](#step-4-adding-a-menu-shortcut)  
5. [How to Use It](#how-to-use-it)  

---

## Step 1: Installing Required Libraries

To control G-Mode, we need the **acpi-call-dkms** module to interact with ACPI.

### Install the module
```bash
sudo apt install acpi-call-dkms
If acpi-call-dkms isn’t available in your distro, try acpi-call instead (check your package manager!).
```
### Load the module
``` bash
sudo modprobe acpi_call
Make it load at startup
echo "acpi_call" | sudo tee -a /etc/modules
``` 
Some systems use /etc/modules-load.d/ — adjust if needed.

## Verify it’s working
```
lsmod | grep acpi_call
```
If it’s empty, reboot or rerun:
```
sudo modprobe acpi_call
``` 
## Step 2: Choosing a Script Location
We’ll place the script in /usr/local/bin — it’s in your $PATH, so you can run it from anywhere without typing the full path.

## Step 3: Creating the Script

Create the script file
sudo nano /usr/local/bin/gmode
Add this code
```
#!/bin/bash

# Path to store the current state
STATE_FILE="/tmp/gmode_state"

# Check for root privileges
if [ "$(id -u)" -ne 0 ]; then
    echo "Error: This script requires root privileges (use sudo)."
    exit 1
fi

# Create state file if it doesn’t exist
if [ ! -f "$STATE_FILE" ]; then
    echo "0" > "$STATE_FILE"
fi

# Read current state
CURRENT_STATE=$(cat "$STATE_FILE")

# Toggle G-Mode
if [ "$CURRENT_STATE" -eq "0" ]; then
    echo "\_SB.AMWW.WMAX 0 0x15 {1, 0xab, 0x00, 0x00}" > /proc/acpi/call
    echo "\_SB.AMWW.WMAX 0 0x25 {1, 0x01, 0x00, 0x00}" > /proc/acpi/call
    echo "1" > "$STATE_FILE"
    echo "G-Mode enabled: Fans are at maximum speed."
else
    echo "\_SB.AMWW.WMAX 0 0x15 {1, 0xa0, 0x00, 0x00}" > /proc/acpi/call
    echo "\_SB.AMWW.WMAX 0 0x25 {1, 0x00, 0x00, 0x00}" > /proc/acpi/call
    echo "0" > "$STATE_FILE"
    echo "G-Mode disabled: Fans are in standard mode."
fi
```
Save and exit
Ctrl + O, then Enter
Ctrl + X to close Nano

Make it executable

```sudo chmod +x /usr/local/bin/gmode```
Now, typing:
```
sudo gmode
```
will toggle between max fan speed and standard mode.
## Step 4: Adding a Menu Shortcut
```
To toggle G-Mode without the terminal, create a .desktop file for your app menu.
```
### Create the file
```
nano ~/.local/share/applications/gmode.desktop
```
Insert this text
ini
```
[Desktop Entry]
Name=G-Mode Toggle
Comment=Toggle G-Mode to control fans
Exec=pkexec /usr/local/bin/gmode
Icon=/usr/share/icons/hicolor/48x48/apps/system.png
Terminal=false
Type=Application
Categories=Utility;
```
Install Polkit if not already available:

```
sudo apt install policykit-1
```
Swap the Icon path for any icon you like.

## Make it executable
```
chmod +x ~/.local/share/applications/gmode.desktop
```
### How to Use It
Terminal method
```
sudo gmode
```
### GUI method
Open your app menu

Find G-Mode Toggle

Click it (you’ll need to enter your password)
