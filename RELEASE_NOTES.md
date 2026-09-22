# RTSP Viewer — Ignition 8.1 Edition — v2.0.0

Live IP-camera **RTSP** viewing inside **Ignition Perspective** on the **8.1** line. Configure cameras
on the Gateway, drop the **RTSP Camera Grid** component into a view, and you have live cameras on
screen — with no browser plugins, and credentials that never leave the Gateway.

The module is now published by **Parsley Automation** and licensed through **Ignition's own
licensing**. Both changes mean a new major version and a **new module** on the Gateway rather than an
in-place upgrade of v1.0.x — see [Upgrading from v1.0.x](#upgrading-from-v10x).

**Free — up to 3 camera feeds.** Perpetual, no time limit. **Maker Edition gets unlimited cameras**,
also free. To go beyond 3 on a standard Gateway, buy **RTSP Viewer Unlimited** from **Parsley
Automation** (https://www.parsleyautomation.com) and activate it on the Gateway's Licensing page — no license keys
to paste, and the new limit applies immediately, without a restart.

## What changed
- **Licensing is Ignition's own.** License keys are gone; the Gateway's license decides the camera
  limit, and a change takes effect at once.
- **New publisher and signing certificate** — the Gateway asks you to trust it once on install.
- **Camera credentials are protected.** The configuration page no longer shows camera passwords
  (they appear as `***`, and saving a camera back keeps the stored one), and they are encrypted
  in the Gateway's settings files instead of sitting there in plain text - so a screenshot, a
  support bundle or a Gateway backup no longer hands them over. Existing cameras are encrypted
  automatically on first start.
- **Clearer video-server failures.** When another program already holds one of the module's local
  ports — most often a second Gateway on the same machine also running RTSP Viewer — the Gateway log
  now names the port, what it is for and how to move it, instead of retrying silently forever. The
  configuration page shows the same reason while cameras are offline, and it recovers on its own once
  the port is free.
- **Survives a hard Gateway crash** — a video server left running by a killed Gateway is cleaned up at
  startup instead of blocking every camera.

## Highlights
- **Any RTSP camera** (H.264), delivered two ways:
  - **HLS** (default) — reverse-proxied over the Gateway's own web port (its TLS, **no extra
    ports**; viewing is open to anonymous sessions so a kiosk works without a login - set
    `-Drtsp.requireAuth=true` to require one), works anywhere the Gateway is reachable, a few seconds behind live.
  - **WebRTC** (optional) — **under a second** of latency for same-network viewers; a tile that
    cannot get through falls back to HLS on its own.
- **RTSP Camera Grid component** — Grid / Single / 2-Up / Quad / Hero layouts, rotation, patrol tours,
  bindable full-screen focus + picture-in-picture.
- **Self-healing feeds** — a frozen or choppy tile recovers without a page refresh.
- **UniFi Protect extras** (optional) — auto stream-restart and camera reboot when a feed degrades.

## Requirements
- **Ignition 8.1.5 or newer** — standard, **Maker Edition**, or unlicensed trial mode
- Cameras providing an **H.264** RTSP stream (switch H.265 to H.264 on the camera)
- A normal browser (Chrome/Edge) for viewing

## Install
1. Gateway → **Config → Modules → Install or Upgrade a Module…** → choose the `.modl`.
2. Accept the one-time certificate prompt (fingerprints are in the README) and the license
   agreement shown with it. Until both are accepted the Gateway quarantines the module.
3. The module starts immediately — **no Gateway restart needed** on 8.1.
4. Add cameras under **Config → Networking → RTSP Cameras**, then drop **RTSP Camera Grid** onto a
   Perspective view.

## Upgrading from v1.0.x
v1.0.x used a different module ID, so this installs alongside it rather than over it. The two share
this module's settings folder, so nothing needs importing — but they cannot run together, because they
use the same local video ports.

1. Uninstall the old RTSP Viewer under **Config → Modules**.
2. Install this `.modl` and accept the new certificate **and** the license agreement (v1.0.x shipped
   no agreement, so this prompt is new). Your cameras and settings are already there.

**Existing Perspective views keep working** — views built with v1.0.x reference the old component type,
which this version still answers to, so there is nothing to edit or rebuild.

**License keys from v1.0.x no longer apply.** If you were on a paid v1.0.x tier, email Support before
upgrading.

## Assets
- `RtspViewer81-2.0.0.<build>.modl` — the module
- `HOWTO.md` — install + configuration guide (in the repo, not attached)

## Support
**Support@parsleyautomation.com** — www.parsleyautomation.com

_Running Ignition 8.3+? Use the [8.3 build](https://github.com/ParsleyAutomation/RTSP-Ignition-8.3)
instead — same product, same licensing; one Unlimited license covers either edition._

_Module version reports as shown in Config → Modules. Use governed by the EULA in this repo._
