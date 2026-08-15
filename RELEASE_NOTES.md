# RTSP Viewer — Ignition 8.1 Edition — v1.0.0 (Free)

First release of RTSP Viewer for the **Ignition 8.1** line. Same product as the 8.3 edition — the same
camera-wall component, the same configuration page, the same license keys — rebuilt against the 8.1
platform and shipped as its own `.modl`.

## Requirements
- **Ignition 8.1.5 or newer** — standard, **Maker Edition**, or unlicensed trial mode
- Cameras providing an **H.264** RTSP stream (switch H.265 to H.264 on the camera)
- A normal browser (Chrome/Edge) for viewing

## Install
1. Gateway → **Config → Modules → Install or Upgrade a Module…** → choose the `.modl`.
2. Accept the one-time certificate prompt (fingerprint is in the README).
3. The module starts immediately — **no Gateway restart needed** on 8.1.
4. Add cameras under **Config → Networking → RTSP Cameras**, then drop **RTSP Camera Grid** onto a
   Perspective view.

## What you get
- **HLS** by default — rides the Gateway's own web port, no extra ports to open
- **WebRTC** as an option — under a second of latency, on the same network
- Cameras configured on the Gateway; URLs and credentials never reach a browser
- UniFi Protect NVR integration for stream restart and camera reboot
- Free edition streams up to **6 cameras**

## Notes for this edition
- Installs on the whole 8.1 line, from **8.1.5** through current. For **Ignition 8.3+**, use the
  standard `RTSPViewer-Free-*.modl` instead — the two are separate downloads and a Gateway runs one
  or the other.
- The configuration page lives under **Config → Networking** here; on 8.3 it is under *Connections*.
- License keys are interchangeable between editions: a key issued for one works on the other.
