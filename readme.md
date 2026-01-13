
### install and run


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
**5. Error: GDBus.Error:org.freedesktop.DBus.Error.Spawn.ExecFailed: Failed to execute program org.freedesktop.PackageKit: Permission denied**

This error is actually a **good sign**. It means QGroundControl is running, but it tried to ask the system to auto-install some video codecs and was told "No" (because you aren't root).

### Scenario A: The QGC Window is Open

**If you see the QGroundControl window on your browser screen:**
**IGNORE THE ERROR.**

* It is just a warning. QGC works perfectly fine without this permission. It just means it can't automatically update its own video drivers (which we already installed manually).

### Scenario B: The Screen is Still Black (App didn't launch)

If you get this error and the app **crashes** or never appears, you are missing one specific sound library (`libpulse-mainloop-glib0`) that QGC needs to draw the window.

**Run this fix inside your Docker terminal:**

```bash
docker exec -it -u root qgc_web apt-get install -y libpulse-mainloop-glib0

```

**Then try launching QGC again:**

```bash
docker exec -it -u abc -e DISPLAY=:1 -e QT_QUICK_BACKEND=software -e XDG_RUNTIME_DIR=/tmp/runtime-abc qgc_web /config/Desktop/QGC/squashfs-root/AppRun

```

*(Note: I assumed you are using the default user `abc`. If you successfully switched to `danyi` in the previous steps, change `-u abc` to `-u danyi`).*


