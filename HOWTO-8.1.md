# RTSP Viewer — Quick Start

View live IP-camera RTSP streams natively in Ignition Perspective. This guide covers
installing the module, adding cameras on the Gateway, and dropping the component into a
Perspective view in the Designer.

Video reaches the browser one of two ways. **HLS** is the default: it works everywhere, needs no
extra ports, and runs a few seconds behind live. **WebRTC** is optional, under a second behind, but
same-network only — see *Low-latency mode* (section 5).

**Requirements:** Ignition **8.1.5+** (standard, **Maker Edition**, or trial mode; Edge not supported). Cameras must provide an **H.264** RTSP stream (see
Troubleshooting for H.265). FREE tier is limited to **6 enabled cameras**.

---

## 1. Install the module (Gateway)

1. Open the Gateway web UI → **Config → Modules**.
2. Scroll to the bottom → **Install or Upgrade a Module…**
3. Choose the downloaded `RTSPViewer-8.1-Free-*.modl` → **Install**.
4. This build is **self-signed**, so the Gateway shows a certificate prompt the first time —
   review and **accept** it to finish the install.
5. The module starts immediately — no Gateway restart needed on 8.1. It shows as **Running** under
   Config → Modules within a second or two.
   *(The 8.3 edition does require a restart here; 8.1 hot-installs.)*

---

## 2. Add cameras (Gateway)

Camera URLs and credentials live **only on the Gateway** — they are never sent to a browser.
Perspective views reference cameras **by name**.

1. Go to **Config → Networking → RTSP Cameras**.
2. Click **+ Create new Camera** and fill in:
   - **Name** — unique; this is how views reference the camera (e.g. `Dock Door`).
   - **RTSP URL** — full `rtsp://` or `rtsps://` URL including any credentials, e.g.
     `rtsp://user:pass@10.0.0.50:554/stream1`.
   - **Substream URL** *(optional)* — a lower-res stream used for grid tiles to save client CPU.
   - **Zone / Group** *(optional)* — free-text zone for zone dashboards / patrol tours.
   - **Enabled** — on = viewable.
3. **Save.** The tile should begin playing within a few seconds.

**Bulk import:** click **Bulk import** and paste one camera per line:
`Name, rtsp://url[, substreamUrl][, zone]`.

> **FREE tier:** you can enable up to **6** cameras. The 7th enable is rejected with a
> "license limit reached" message. Disable one, or upgrade, to add more.

### Optional: UniFi Protect auto-recovery
Expand the **UniFi Protect NVR** card if your cameras are on a UniFi Protect NVR. With an
API key or a Super-Admin account entered, a choppy/frozen feed is automatically recovered by
asking the NVR to restart that camera's stream (and, if **Allow camera reboot** is on, reboot
the camera). Each camera row also gets a manual **Reboot** button. This is UniFi-only; other
cameras still stream fine, they just don't get the NVR extras.

---

## 3. Add the component to a view (Designer)

1. Open the **Designer** → open (or create) a **Perspective** view.
2. In the **Perspective Components** palette, find the **Central Valley Ignition** category.
3. Drag **RTSP Camera Grid** onto the view.
4. Out of the box it shows **all** configured cameras. To pick specific ones, edit the
   `cameras` prop and add camera **names** (the same names from step 2).

### Properties — full reference

**Cameras & selection**
| Property | How to use |
|----------|-----------|
| `cameras` | The cameras to show, **by name**. Leave empty to show **all** configured cameras. The order you list them sets tile order. |
| `group` | Show only cameras in this zone/group (matches the camera's Zone field). Blank = no zone filter. |
| `activeCamera` | **Bindable.** Set to a camera name to pop that camera full-screen — e.g. bind to an alarm/motion tag to auto-focus. Clear it to return to the grid. |
| `pipCamera` | **Bindable.** Float a small picture-in-picture inset of this camera over any layout. Blank = none. |
| `expandedIndex` | In Grid: which tile is expanded (`-1` = normal grid). Clicking a tile toggles this; also bindable. |

**Layout & appearance**
| Property | How to use |
|----------|-----------|
| `layout` | `Grid` (N-wide), `Single` (one feed), `2-Up`, `Quad` (2×2), or `Hero` (one big + thumbnail strip). |
| `columns` | Grid only: max columns. `0` = auto-fit as many as fit. For 6 cams: `3` = 2 rows of 3, `6` = one row, `2` = 3 rows of 2. |
| `minColumnWidth` | Smallest tile width (px) before the grid drops to fewer columns / switches to a scrolling list. Default 300. |
| `gap` | Pixels between tiles. `0` = flush. |
| `aspectRatio` | Tile shape, e.g. `16:9`. |
| `objectFit` | How video fills a tile: `fill` (stretch, no black — default), `cover` (fill + crop, no black, no stretch), `contain` (letterbox, may show black bars). **Use `cover` or `fill` to remove black gaps.** |
| `hideScrollbars` | `true` (default) = tiles divide the component's height so it never scrolls. `false` = tiles keep true `aspectRatio`, pack from the top, and the grid scrolls if it overflows. |
| `showLabels` | Overlay each camera's name on its tile. |
| `muted` | Mute audio (on by default; required for browser autoplay). |
| `showFps` | Show a live FPS badge per tile — handy for spotting choppy feeds; turns red at `autoRestartMinFps`. |
| `blur` / `blurAmount` | Privacy blur the video (labels stay sharp) — for sanitized screenshots/demos. `blurAmount` = blur strength in px. |
| `emptyMessage` | Text shown when no cameras are configured/selected. |

**Rotation & patrol** (cycle through more cameras than fit)
| Property | How to use |
|----------|-----------|
| `rotate` | Turn on auto-cycling. Pauses while the pointer is over the component. |
| `rotateSeconds` | Seconds each page/camera is shown before advancing. |
| `pageSize` | How many cameras per page while rotating (`0` = as many as fit). |
| `patrolByGroup` | With `rotate` on, cycle through **zones** (a patrol tour) instead of paging cameras. |
| `heroThumbnails` | Hero layout only: how many thumbnails to show under the big feed. |

**Auto-recovery** (heal a frozen/choppy feed without a refresh)
| Property | How to use |
|----------|-----------|
| `autoRestart` | Master switch for auto-recovery (on by default). Off = never auto-restart. |
| `autoRestartAfterSeconds` | How long a feed must stay frozen/choppy before it's restarted. Default 8. |
| `autoRestartMinFps` | FPS below which a still-moving feed counts as "choppy." Default 2. Raise to react to mild stutter. |
| `autoRestartCooldownSeconds` | Minimum seconds between restarts of the same tile (anti-storm). Default 25. |
| `rebootCameraOnLowFps` | When a feed trips the above, **reboot the camera at the NVR** instead of just reconnecting. UniFi only — needs the NVR **account login + Allow reboot** on the config page. Default off. |

**Delivery** (how video reaches the browser — see section 5)
| Property | How to use |
|----------|-----------|
| `transport` | `hls` (default) or `webrtc`. **HLS** works everywhere and needs no extra ports (~2–6s behind live). **WebRTC** is under a second, but it's **same-network only** and needs a UDP port open on the gateway. A WebRTC tile that can't connect falls back to HLS by itself. |
| `showTransport` | Show a small badge on each tile naming the transport in use — `WEBRTC`, `HLS`, or an amber `HLS` when a tile wanted WebRTC and had to fall back. Handy while setting low-latency mode up; turn it off afterwards. |

**Advanced**
| Property | How to use |
|----------|-----------|
| `proxyUrl` | Override the gateway proxy base URL. Leave blank — auto-detected from the session. |
| `debug` | Log verbose player diagnostics to the browser console (per tile), including WebRTC negotiation timings. Turn on only when diagnosing a stuck tile. |

5. **Save** the project. Open the view in a Perspective **Session** (browser) to see live video.

### Layout tip — filling the screen with no gaps
If you see **black bars/gaps** between tiles, `objectFit` is set to `contain` (letterboxing).
Set **`objectFit` = `cover`** (fills every tile, no black, no stretch — crops slightly) or
`fill` (fills with slight stretch). Keep `gap` at `0` for flush tiles. To instead keep true
16:9 tiles packed at the top (dead space collects at the bottom), set `hideScrollbars` = `false`.

---

## 5. Low-latency mode (WebRTC) — optional

By default the component uses **HLS**, which runs about **2–6 seconds behind live**. That's fine for
monitoring, but not for someone reacting to what they see.

**WebRTC** cuts that to **under a second**. It's free to turn on. The trade-off is how the video
travels:

- With **HLS**, video goes *through* the Gateway's normal web port. Nothing else to configure.
- With **WebRTC**, only the initial handshake goes through the Gateway. The **video itself is sent
  straight from the Gateway machine to each browser over a UDP port**, and viewers must be on the
  **same network**. It does not work over the internet or most VPNs.

Many networks work with no firewall change at all, because the Gateway reaches out to each viewer
first and the reply comes back through the same opening. If your viewers connect fine, you're done.
If they don't — tiles show an amber `HLS` badge — **allow the UDP port inbound** on the Gateway
machine and they will.

> **Free while in preview.** Low-latency mode is included at no cost in this release. It may move to a
> paid tier in a future version; existing installs will be given notice.

### Turning it on
1. Designer → select the **RTSP Camera Grid** → set **`transport`** to `webrtc`. Set
   **`showTransport`** to `true` while you're testing.
2. Save, then open the view in a browser and hard-refresh (**Ctrl+Shift+R**).
3. Check the badges. Green **`WEBRTC`** means you're done — no firewall change needed.
4. Only if you see amber **`HLS`**: Gateway → **Config → Networking → RTSP Cameras** → expand
   **Connectivity — video delivery**, note the **Media UDP port** (default `18189`), and allow it
   inbound on the Gateway machine's firewall.

Each tile should show a green **`WEBRTC`** badge, and the Connectivity card's counter should switch to
show WebRTC viewers.

### Checking it works
The Connectivity card has a **Test from this browser** button. It opens a real low-latency connection
to one of your cameras and reports whether video actually arrived, and from which address. Run it
**from a machine that will really be watching the cameras** — testing on the Gateway itself always
passes, because that traffic never crosses a firewall.

### If tiles show an amber `HLS` badge
That's the automatic fallback: the tile asked for WebRTC, got no video, and switched to HLS so you'd
still see the camera. Almost always the UDP port is blocked between that machine and the Gateway. The
Connectivity card lists which cameras fell back and why.

---

## 6. Troubleshooting

| Symptom | Fix |
|---------|-----|
| Tile stuck on **Connecting…** then **OFFLINE** | Camera unreachable, wrong URL/credentials, or the stream is **H.265**. Browsers can't decode H.265 — set that camera's stream to **H.264** in its settings. |
| Video won't play in **Perspective Workstation** (works in a browser) | Workstation's embedded browser ships with H.264 playback disabled, so tiles stay black. **Use a normal browser** (Chrome/Edge) — that is the supported configuration. Workstation can be made to play H.264 by changing its own launcher config, but that is your change to make and your licensing responsibility; see the README section *Perspective Workstation*. |
| Can't enable a 7th camera | FREE tier cap (6). Disable another camera or upgrade. |
| UniFi camera won't stream over `rtsps://:7441` | Use the plain-RTSP **:7447** endpoint (UniFi → camera → Manage → RTSP). |
| Certificate warning on install | Expected for this self-signed build — accept it once. |
| Installed the module but nothing appears | On Ignition **8.1.5+** a module install or upgrade needs a **Gateway restart** to take effect. Until then the module is staged but not running (and an upgrade keeps serving the previous version). |
| `transport` isn't in the Property Editor | The Designer cached the old component list. **Close and reopen the Designer.** |
| Set `transport` to `webrtc` but nothing changed | The browser is running the cached component code. **Hard-refresh** the session (Ctrl+Shift+R). |
| Amber **`HLS`** badge instead of `WEBRTC` | The tile fell back. Open the Gateway's UDP media port (default 18189) inbound and make sure viewers are on the same network. |
| Connectivity card says **only loopback is available** | The Gateway can't see a network address of its own (usually a container or NAT). Enter the address viewers actually use under *Additional advertised hosts*. |

---

*RTSP Viewer · Central Valley Ignition · www.CentralValleyIgnition.com*
