# PetStalker_RaspberryPi_Camera

A Wi-Fi pet camera built on a Raspberry Pi Zero 2 W and a USB webcam. You can watch the live stream from your iPhone anywhere (at work, on cellular, etc.). Everything is free: no subscriptions and no cloud service.

- **Camera software:** motionEye (free, open source, no coding needed)
- **Remote access:** Tailscale (free Personal plan, no port forwarding)

---

## Hardware

| Item | Notes |
|------|-------|
| CanaKit Raspberry Pi Zero 2 W kit | Includes the Pi, 2.5A power supply, OTG cable, case, 64 GB microSD card |
| Logitech C270 webcam | Plugs in through the OTG cable |
| microSD card reader | Needed if your laptop has no microSD slot |
| Laptop (Windows) | Used only for setup |
| iPhone | Used to watch the stream |
| Home Wi-Fi | Must have a **2.4 GHz** network (the Zero 2 W can't use 5 GHz) |

Not needed: the GPIO header, the mini HDMI adapter, a monitor, or a keyboard.

> ⚠️ The camera has no night vision. Leave a small lamp on if you want to see your pets at night.

---

## Software (all free)

**On the laptop:** Raspberry Pi Imager, plus Terminal/PowerShell (SSH is built into Windows 10/11)
**On the Pi:** Raspberry Pi OS Lite (64-bit), motionEye, Tailscale
**On the iPhone:** Tailscale app, Safari

---

## Setup

### 1. Flash the microSD card
Reflash the card even though it came preloaded, so you get the latest OS with your Wi-Fi and SSH already set up.

1. Install **Raspberry Pi Imager** from raspberrypi.com and insert the card.
2. Device: **Raspberry Pi Zero 2 W**
3. OS: **Raspberry Pi OS (other) → Raspberry Pi OS Lite (64-bit)**
4. Storage: your microSD card
5. When asked to customize settings, choose **Edit settings** and set:
   - Hostname: `petcam`
   - Username and password (write these down)
   - Wi-Fi name and password (2.4 GHz network)
   - Services tab: **Enable SSH** (use password authentication)
6. Click **Write**.

> If Windows asks to format the card afterward, click **Cancel**. Formatting would erase what you just flashed.

### 2. Connect the hardware
- Put the card in the Pi and close the case.
- Webcam → OTG cable → the micro-USB port labeled **USB** (the one nearer the middle)
- Power supply → the port labeled **PWR IN** (the one at the edge)
- Wait 3–5 minutes for the first boot.

### 3. Log in from your laptop
Open PowerShell and run:
```
ssh yourusername@petcam.local
```
Type `yes` to accept the fingerprint, then enter your password.

Make sure the Pi sees the webcam:
```
ls /dev/video*
```
You should see `/dev/video0` (and possibly more). If you see nothing, check that the webcam is in the **USB** port, not PWR IN.

### 4. Update the Pi
```
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```
Wait a minute, then SSH back in.

### 5. Install motionEye
```
sudo apt install -y --no-install-recommends ca-certificates curl python3 python3-dev python3-pip libcurl4-openssl-dev gcc libssl-dev
sudo python3 -m pip install --break-system-packages --pre motioneye
sudo motioneye_init
```
This can take 10–20 minutes on the Zero 2 W, so be patient. motionEye now starts on its own every time the Pi boots.

### 6. Set up the camera in motionEye
1. On your laptop (same Wi-Fi), open `http://petcam.local:8765`.
2. Log in with username `admin` and **no password**.
3. Add the camera if it isn't detected automatically: **Add camera → Local V4L2 camera → the C270**.
4. Settings to change:
   - **General Settings → Admin password:** set a strong password
   - **Video Device → Resolution:** `640x480` (smoother on the Zero 2 W)
   - **Video Device → Frame rate:** `10–15`
   - Optional: turn on **Motion Detection** and **Movies** to record clips. Set **Preserve Movies** to about 1 week so the card doesn't fill up.
5. Click **Apply**.

### 7. Install Tailscale on the Pi
```
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```
It prints a login link. Open it on your laptop and sign in, then create a free account if you don't have one.

Next, open the Tailscale admin console at login.tailscale.com, find **petcam**, click **⋯ → Disable key expiry**. Without this, the Pi disconnects after about 6 months.

### 8. Set up the iPhone
1. Install the **Tailscale** app and sign in with the **same account**.
2. Turn Tailscale on.
3. In Safari, go to `http://petcam:8765` and log in.
4. Bookmark it, or use **Share → Add to Home Screen** to get an app-like icon.

That's it. As long as Tailscale is on, the stream works on any Wi-Fi or on cellular.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `petcam.local` not found | Wait longer after boot, check that the Wi-Fi is 2.4 GHz, or find the Pi's IP in your router's app |
| No camera / black image | Webcam must be in the **USB** port, not PWR IN. Run `ls /dev/video*` |
| Stream is choppy | Lower the resolution or frame rate in motionEye |
| Can't connect from work | Make sure Tailscale is on in the iPhone app and the Pi shows "Connected" in the admin console |
| Pi randomly reboots | Use the CanaKit 2.5A supply, not a phone charger |

---

## Optional: Flask instead of motionEye
If you'd rather write your own stream page in Python:
```
sudo apt install -y python3-flask python3-opencv
```
Don't install OpenCV with `pip` on the Zero 2 W. Run your Flask app on port 5000 and view it at `http://petcam:5000` through Tailscale. Use a systemd service to make it start on boot.
