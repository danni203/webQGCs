Since you are using **WiFi (UDP)**, this error is a "False Alarm." QGroundControl thinks you *might* want to plug in a USB radio, so it checks for permissions it doesn't have. Because you are using WiFi, you don't need those permissions.

The reason `usermod danyi` failed is that "danyi" is only your **Web Login** name. The internal Linux user is still named **`abc`**.

Here are the two ways to stop the error.

### Option 1: Disable the Check in QGC (Recommended)

Since you are using WiFi, simply tell QGC to stop looking for USB devices.

1. **Launch QGC** (click OK on the error to dismiss it for now).
2. Click the **"Q" Icon** (Top Left)  **Application Settings**.
3. Scroll down to the **AutoConnect** section.
4. **UNCHECK** the following boxes:
* [ ] Pixhawk
* [ ] SiK Radio
* [ ] PX4 Flow
* [ ] LibrePilot
* [ ] RTK GPS


5. **KEEP CHECKED:**
* [x] UDP
* [x] NMEA GPS Device (Optional, but usually safe)



**Result:** QGC will stop scanning your USB ports on startup, and the error will disappear.

---

### Option 2: The "Nuclear" Fix (Docker Flag)

If you can't get to the settings or want the error gone forever at the system level, use the `--privileged` flag. This gives the container full access to your hardware, so the check passes.

**1. Delete the old container:**

```bash
docker rm -f qgc_web

```

**2. Run with `--privileged`:**
*(I also fixed the user variable to `abc` so the drivers install correctly).*

```bash
docker run -d \
  --name qgc_web \
  --privileged \
  --shm-size=1gb \
  -e PUID=1000 \
  -e PGID=1000 \
  -e CUSTOM_USER=danyi \
  -e PASSWORD=ilikecoffe \
  -p 3000:3000 \
  -p 3001:3001 \
  -v "$(pwd)/qgc_data:/config" \
  lscr.io/linuxserver/webtop:ubuntu-xfce

```

**3. Re-install Drivers (One last time):**

```bash
docker exec -it -u root qgc_web bash -c "apt-get update && \
apt-get install -y libpulse-dev libx11-xcb1 libasound2t64 libnss3 libxcomposite1 libxcursor1 libxi6 libxtst6 libxcb-cursor0 libxcb-xinerama0 libxkbcommon-x11-0 libpipewire-0.3-0 gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl"

```

**4. Launch:**

```bash
docker exec -it -u abc -e DISPLAY=:1 -e QT_QUICK_BACKEND=software -e XDG_RUNTIME_DIR=/tmp/runtime-abc qgc_web /config/Desktop/QGC/squashfs-root/AppRun

```
