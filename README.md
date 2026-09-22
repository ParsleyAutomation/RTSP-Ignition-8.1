# RTSP Viewer for Ignition 8.1

> **This is the Ignition 8.1 build.** Running **Ignition 8.3 or newer**? Use
> **[ParsleyAutomation/RTSP-Ignition-8.3](https://github.com/ParsleyAutomation/RTSP-Ignition-8.3)** instead — same
> product, built for that platform line. A Gateway runs one or the other: this 8.1 build *will* install on
> an 8.3 Gateway but logs an error and its configuration page does not work there, so use the 8.3 build
> instead. Licenses and Perspective views work on either.

View live IP-camera **RTSP** streams natively in **Ignition Perspective**. A gateway-managed relay
repackages each camera's existing stream for the browser — **without re-encoding it** — and plays it
in an **RTSP Camera Grid** component. No browser plugins, and no transcoding, so image quality is
untouched and a screen full of cameras costs the Gateway very little.

One download, for Ignition 8.1. It runs **free with up to 3 cameras** forever, is **unlimited on Maker
Edition**, and lifts the limit on any Gateway that activates an **RTSP Viewer Pro** licence — no
keys to paste, no reinstall. See [Editions](#editions).

**Two delivery modes.** **HLS** (the default) rides the Gateway's own web port — no extra ports, works
anywhere the Gateway is reachable, a few seconds behind live. **WebRTC** is optional and gets you
**under a second** of latency, but sends video direct over a UDP port you must open and only works on
the **same network**. See [Low-latency mode](#low-latency-mode-webrtc).

![Live camera wall in a Perspective session](images/camera-wall.png)

*A camera wall running live in a browser Perspective session.*

---

## Download

Get the latest `.modl` from the **[Releases](https://github.com/ParsleyAutomation/RTSP-Ignition-8.1/releases)** page.

> This build is **self-signed**. On install, the Gateway shows a one-time certificate prompt —
> review the fingerprint below and accept it. Nothing else is affected.

**Signing certificate — SHA-256 fingerprint** (verify before trusting):
```
40:37:69:D9:BB:72:E6:A9:FC:09:92:67:81:B1:93:24:56:2C:AB:8A:7D:E3:50:51:93:CC:67:D5:B7:8F:A6:C0
Subject: CN=Parsley Automation, O=Parsley Automation, L=Fresno, S=CA, C=US
```

The Gateway's trust dialog shows the shorter **SHA-1 thumbprint** instead:
```
933a62da510691a8a758c601229b4afa6ed66cd5
```

---

## Requirements
- **Ignition 8.1.5+** (the 8.1 line) — standard, **Maker Edition**, or unlicensed trial mode. *(Edge is not supported —
  IA requires Edge modules to be whitelisted.)*
- Cameras providing an **H.264** or **H.265 (HEVC)** RTSP stream. The Gateway repackages either without
  re-encoding; what differs is the viewer. H.264 plays everywhere. H.265 plays where the browser can
  decode it — Safari, and Chrome/Edge on machines with HEVC support — and a tile that cannot decode it
  says so in as many words rather than sitting on "Connecting…". H.265 also cannot be carried over
  WebRTC by most browsers, so those tiles use HLS automatically. If your viewers are mixed or unknown,
  **H.264 remains the safe choice.**
- View camera feeds in a normal **browser** (Chrome/Edge) — this is the supported configuration.
  **Perspective Workstation** can also display them, but does not play H.264 out of the box; see
  [Perspective Workstation](#perspective-workstation) below.

## Install (Gateway)
1. Gateway web UI → **Config → Modules** → **Install or Upgrade a Module…**
2. Choose the downloaded `.modl` → **Install** → accept the **certificate** and the **license
   agreement** when prompted. Until both are accepted the Gateway keeps the module quarantined and
   it will not start.
3. That's it — on 8.1 the module starts immediately and shows **Running** under Config → Modules.
   No Gateway restart is required.

### Installing in Docker

The official `inductiveautomation/ignition:8.1.*` image works, with two things to know:

- Put the `.modl` in `/usr/local/bin/ignition/user-lib/modules/` (bind-mount or `docker cp`) before the
  container's first start.
- Set **`GATEWAY_MODULES_ENABLED=all`** so the Gateway finishes commissioning with a third-party module
  present.
- The 8.1 images have no variable for accepting a module certificate, so trust it once in the Gateway
  web UI after the container is up.

## Add cameras (Gateway)
1. **Config → Networking → RTSP Cameras** → **+ Create new Camera**.
2. Enter a **Name**, the **RTSP URL** (with any credentials), optional substream/zone.
   Camera passwords are encrypted in the Gateway's settings and are shown on this page only as
   `***`; re-saving a camera keeps the stored password, and typing over the `***` replaces it.
3. Save. Camera **passwords** stay on the Gateway: encrypted in its settings, shown on this page
   only as `***`, and never sent to a Perspective session.
4. An unlicensed Gateway allows **3 enabled cameras**; the 4th is refused until the Gateway is
   licensed (Maker Edition is unlimited).

![RTSP Cameras config page with the license tier](images/gateway-config.png)

## Add the component (Designer)
1. Open a **Perspective** view.
2. From the **Parsley Automation** palette category, drag **RTSP Camera Grid** onto the view.
3. Leave `cameras` empty to show all, or list cameras by **name**. Save and open a Session.

*In the Designer, tiles show a placeholder — live video only renders in a browser Session:*

![RTSP Camera Grid in the Ignition Designer](images/designer-preview.png)

Full guide: **[HOWTO.md](HOWTO.md)** in this repo.

---

## Low-latency mode (WebRTC)

By default the component uses **HLS**, roughly **2–6 seconds** behind live — fine for monitoring, not for
someone reacting to what they see. Switching the grid's `transport` property to `webrtc` drops that to
**under a second**.

What it costs you:

| | HLS *(default)* | WebRTC |
|---|---|---|
| Latency | ~2–6s | **< 1s** |
| Works over internet / VPN | **Yes** | No — same network only |
| Setup | Nothing | Set one property; open a UDP port only if needed |

Only the initial handshake goes through the Gateway; the video itself is sent straight from the
Gateway machine to each browser over a UDP port (default `18189`). Many networks need no firewall
change for this, since the Gateway initiates the connection and the reply returns through the same
opening — but if viewers can't get through, allowing that port inbound fixes it.

A tile that can't establish a connection **falls back to HLS automatically**, so a blocked port means
higher latency rather than a black screen — and the Gateway reports which cameras fell back and why.

Configure it under **Config → Networking → RTSP Cameras → Connectivity**, which also shows the
addresses browsers will be told to use and includes a **Test from this browser** button.

> **Free while in preview.** Low-latency mode is included at no cost in this release. It may move to a
> paid tier in a future version; existing installs will be given notice.

---

## Perspective Workstation

RTSP Viewer works in Perspective Workstation, but **not by default**, and **only for H.264**.

Workstation's embedded browser ships with the proprietary codecs **disabled**. Until they are
enabled, camera tiles stay black while the rest of the view renders normally — whether the view holds
one camera or twenty. Inductive Automation document a system property that turns them on, which you
add to your own Workstation installation. It is off in a stock install, and neither Parsley
Automation nor this module turns it on for you.

Verified on Workstation 8.3.9: with that property set, an **H.264** camera plays normally. An
**H.265 (HEVC)** camera still does not play in Workstation, because the property enables H.264 and
AAC rather than HEVC. For Workstation, use H.264 cameras.

### What this module does and does not do

- It **does not** contain, install, bundle, or distribute an H.264 decoder or any other codec.
- It **does not** transcode or re-encode video. Frames are relayed from your camera to your browser
  exactly as the camera produced them.
- It **does not** modify Ignition, Workstation, or any Inductive Automation software.
- The decoder used in Workstation is part of Workstation, not part of this module.

### Your responsibility

Decoding H.264 can carry patent-licensing obligations, depending on your jurisdiction, your
deployment, and how you use it. If you enable H.264 playback in Workstation, **you are responsible for
determining and meeting any licensing obligations that apply to you**, including any AVC/H.264
patent-pool terms. Parsley Automation provides no license, sublicense, or indemnity for H.264
decoding.

---

## Editions

| Edition | Camera feeds | How to get it |
|---------|-------------|---------------|
| **Free** | 3 | This download. Perpetual. |
| **Maker** | Unlimited | Automatic on **Ignition Maker Edition** (non-commercial use). Nothing to buy or enter. |
| **Pro** | Unlimited | Bought from [Parsley Automation](https://www.parsleyautomation.com). Single gateway, perpetual; capacity depends on your hardware. The [Module Showcase](https://inductiveautomation.com/moduleshowcase/) listing is still titled **RTSP Viewer Unlimited** — same tier, older name. |

Upgrading needs no reinstall and no restart: activate the RTSP Viewer license on the Gateway's
**Licensing** page, like any other module, and the camera limit lifts immediately.

## Upgrading from v1.0.x

v1.0.x was published under a different module ID, so the Gateway treats v2.0.0 as a **new module**
rather than an upgrade. Both versions share this module's settings folder, so nothing needs importing —
but they cannot run at the same time, because they use the same local video ports.

1. Uninstall the old RTSP Viewer under **Config → Modules**.
2. Install the v2.0.0 `.modl` and accept the new certificate **and** the license agreement (v1.0.x
   shipped no agreement, so this prompt is new). Your cameras and settings are already there.

**Your existing Perspective views keep working.** Views built with v1.0.x reference the old component
type, which this version still answers to, so there is nothing to edit or rebuild.

**License keys from v1.0.x no longer apply** — licensing is now Ignition's own (see
[Editions](#editions)). If you were on a paid v1.0.x tier, email Support before upgrading.

## Support
- Issues / questions: open an **[Issue](https://github.com/ParsleyAutomation/RTSP-Ignition-8.1/issues)** or email **Support@parsleyautomation.com**.
- Include your Ignition version and, for camera problems, the camera make/model + stream codec.

## License
Proprietary. Use is governed by **[EULA.md](EULA.md)**. Not open source.

This module bundles two open-source components, used under their own licenses:
[MediaMTX](https://github.com/bluenviron/mediamtx) (MIT) and
[hls.js](https://github.com/video-dev/hls.js) (Apache-2.0). See
**[THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt)** (full license texts also included in the
module). React/React-DOM are provided by the Ignition Perspective runtime and are not bundled.
