# RTSP Viewer — Ignition 8.1 Edition — v2.1.0

**H.265 cameras now work, and a camera source no longer has to be RTSP.** RTMP, SRT and raw RTP work
alongside it, which matters when something other than the camera is feeding the Gateway.

**Read the breaking changes below before upgrading.** One of them will stop a working display.

---

## Breaking changes

- **The standalone camera wall page is gone.** `/res/pa-rtsp/viewer.html` returns a 404. A kiosk,
  wall display or browser tab pointed at it stops working on upgrade. Build a Perspective view with
  the **RTSP Camera Grid** component and point the display at that instead. There is no replacement
  URL and no Vision component.

- **Perspective is now needed to view anything, and on 8.1 nothing stops you installing without
  it.** With the wall page gone, the Camera Grid component is the only way to watch a camera. The
  8.3 edition marks Perspective a *required* dependency so the Gateway refuses the install
  outright; **that cannot be expressed on 8.1** — `required` on a module dependency is an 8.3
  feature — so this edition will install happily onto a Gateway with no Perspective and leave you
  able to configure cameras you cannot see. Check Perspective is present before upgrading.

- **You will be asked to accept the licence agreement, and the module stays inactive until you do.**
  The agreement ships inside the module now and earlier builds carried none, so Ignition treats this
  as new. Until it is accepted the Gateway holds the module in quarantine and **cameras will not
  stream**, with nothing on the cameras page to explain why — the reason appears in the Gateway log
  as *"Moving module RTSP Viewer to quarantine because license not yet accepted"*. **If you are
  upgrading an unattended site, plan for someone to be at the Gateway.** The signing certificate is
  unchanged and is not re-prompted.

---

## What changed

### Sources

- **Four new source protocols: `rtmp://`, `rtmps://`, `srt://` and `udp+rtp://`.** The camera field
  accepts them anywhere it accepted `rtsp://`, and they behave identically once configured — same
  on-demand pull, same HLS and WebRTC delivery, same credential protection, same **H.264 and H.265**
  support on every one of them.

  **This does not change what you should point at an IP camera.** Essentially every IP camera speaks
  RTSP, and nothing here makes RTSP worse or obsolete. These are for when a camera is *not* what you
  are connecting to: an encoder in front of a legacy camera, an SRT contribution feed from another
  site, a multicast RTP feed from a video distribution system.

  The field is now labelled **Source URL** rather than **RTSP URL**. Nothing stored changes.

- **SRT passphrases are protected the same way camera passwords are.** SRT carries its secret in a
  `passphrase` query parameter rather than in the userinfo, so it needed handling of its own. It is
  encrypted at rest and masked in the UI and in logs, exactly like an RTSP password. `streamid` is
  deliberately left readable — it is an address, not a secret.

- **Raw RTP works without asking you for an SDP.** Raw RTP has no signalling channel, so a receiver
  cannot discover the codec. The Gateway generates the session description itself from the URL:
  `?codec=h265` and `?pt=` override the defaults of H.264 and payload type 96.

### Video

- **H.265 (HEVC) cameras are supported.** Point a camera at the Gateway in H.265 and it streams. The
  Gateway repackages it exactly as it does H.264 — **no re-encoding**, so image quality is untouched
  and the CPU cost is the same. You no longer have to switch the camera to H.264 first, which was
  not always possible when an NVR owned that camera.

  **What decides whether you see a picture is the browser, not the Gateway.** HEVC decoding is
  usually done in hardware, so it depends on the browser *and* the machine.

- **A tile that cannot decode a camera now names the reason.** An H.265 camera used to be
  indistinguishable from a dead one — *Connecting…*, then **OFFLINE** — and an installer would go
  and check cabling, credentials and firewalls for a problem that was none of those. It now reads:

  > This browser can't decode H.265 (HEVC). Try Chrome or Edge on a machine with HEVC support, or
  > set this camera to H.264.

- **Low-latency mode sorts itself out on H.265.** A tile asks the browser what it supports rather
  than assuming. Where WebRTC can carry H.265 it is used; where it cannot, the tile switches
  straight to HLS — which that same browser may well play — instead of sitting dark until the
  connection times out.

- **Fixed: a camera the browser couldn't decode was retried forever.** That failure was treated
  internally as recoverable, so the player looped, never settled and never produced a message. It
  now stops on the first failure and reports it.

- **Player updated** — hls.js 1.5.17 → 1.6.16.

### Licensing

- **The paid tier is called Pro, not Unlimited.** Only the name changes: same licence, same price,
  same unlimited feed count. The licensing card and log line read `PRO`.

  The Module Showcase listing is still titled **RTSP Viewer Unlimited**, so that is the name to look
  for when buying until the listing is renamed.

---

## Requirements

- **Ignition 8.1.5 or newer** — standard, **Maker Edition**, or unlicensed trial mode. On
  **Ignition 8.3+**, use the [8.3 build](https://github.com/ParsleyAutomation/RTSP-Ignition-8.3)
  instead; it is the primary edition.
- **Perspective** — see the breaking change above. Not enforced on 8.1, but nothing works without it.
- A source providing **H.264** or **H.265** over RTSP, RTMP, SRT or raw RTP. H.264 plays in every
  browser; H.265 plays where the browser can decode it.
- A normal browser (Chrome/Edge) for viewing.

RTMP, SRT and RTP need no extra ports on the Gateway and no configuration beyond the URL: the
Gateway makes the outbound connection, exactly as it does for RTSP.

## Upgrading

1. Gateway → **Config → Modules → Install or Upgrade a Module…** → choose the `.modl`.
2. **Accept the licence agreement** — see the breaking change above; the module is inactive until
   you do.
3. The module restarts itself — **no Gateway restart needed** on 8.1.

Cameras, UniFi NVR settings, WebRTC settings, your licence and existing Perspective views all carry
over untouched. No camera URL needs to change. Views built before the rebrand keep
working: the component's previous id is still registered as an alias, so nothing has to be
rebuilt or repointed.

## Known limits

- **MJPEG is not supported**, and is not planned through this route. The bundled media server has no
  MJPEG source type, so it cannot simply be another scheme in the list — it would be a separate
  delivery path. If you need it, say so; it is waiting on demand, not a technical dead end.
- **RTMP cannot carry H.265.** That is a limit of the protocol's container, not of this module — use
  SRT or RTSP for an HEVC feed.
- **Raw RTP assumes payload type 96 and a single video stream.** Both are what encoders
  overwhelmingly send. A different payload type needs `?pt=`; a feed carrying multiple programmes is
  not addressed by this release.
- **H.265 decoding belongs to the browser**, on every one of these protocols. The Gateway will
  deliver the stream; whether a given workstation shows it depends on that machine. Test the viewers
  you actually use before rolling an H.265 camera out to a site.
- **Sub-second latency on H.265 depends on the browser.** WebRTC is what delivers sub-second
  latency. Recent Chrome can receive H.265 over it; browsers that cannot will play those cameras
  over HLS, a few seconds behind live, and nothing announces the difference. If latency is what a
  camera is for — steering a PTZ, watching an interlock — **H.264 is the predictable choice**.

## Assets

- `RTSPViewer-8.1-Free-v2.1.0.modl` — the module
- `HOWTO.md` — install and configuration guide (in the repo, not attached)

## Support

**Support@parsleyautomation.com** — www.parsleyautomation.com

_Module version reports as shown in Config → Modules. Use governed by the EULA in this repo._
