# Tunnel Term

**A remote field test kit for Windows.** The technician on site plugs a
laptop into the equipment. A remote engineer opens a link in a browser and
works the laptop's serial console and test Ethernet port as if they were
there: no VPN, no screen sharing, nothing to install on the engineer's side.

![The engineer's view: a shared serial console and the laptop's network adapters](docs/screenshots/eng-2-console.png)

> This repository hosts the downloads and this guide; the source code is
> private. To share a laptop you need a **setup code** from your admin. The
> local console works without one.

**[Download the latest release](https://github.com/illos/tunnel-term-releases/releases)**

## What it does

**For the technician** (Windows app)

- Serial console for USB-serial cables on any COM port, with BREAK, live
  serial settings and safe multi-line paste.
- One click shares the laptop. You get a link and an 8-character pairing
  code to read out to the engineer.
- You see everything the engineer does, can type in the same terminals, and
  can end the session at any time.

**For the remote engineer** (any modern browser)

- Shared terminal tabs: the serial console, plus SSH, telnet, raw TCP,
  ping, traceroute and TCP port checks, all run *from the laptop*.
- Control of the laptop's test Ethernet port: VLAN ID, static IP, DHCP,
  routes and bounce. Every change undoes itself after 60 seconds unless you
  click **Keep**.
- Files: upload firmware or configs, serve them to the device over TFTP or
  HTTP, and push or pull them over SFTP.
- Paced paste for long configs, so a console without flow control doesn't
  drop characters.

**Safe by design**

- Nothing is shared until the technician clicks **Start remote session**.
  Sharing stops at **End session**, after 60 minutes without activity, or
  after 4 hours (the tech can extend).
- A pairing code works once and only for 10 minutes. Five wrong tries burn
  it.
- The laptop's internet connection (Wi-Fi, hotspot, Starlink) is locked.
  Nobody can change it from the browser, so a session can't cut itself off.
  Default gateways are refused on the test port.
- When the session ends, network changes are put back, files are deleted
  and the link stops working.
- Nothing typed or shown in a terminal is ever written to disk or to a log.
  Only events are logged: who joined, what was opened, what changed.

## How it works

```
 Device ──serial / Ethernet──▶ Technician's laptop ──Cloudflare Tunnel──▶ Engineer's browser
                               (Tunnel Term)          (outbound only)
```

The laptop connects *out* to Cloudflare, so there are no open ports, and it
works behind phone hotspots, Starlink and NAT. The engineer opens the
laptop's own link, `https://<laptop>-remote.<your domain>`.

## Install

**You need:**
- Windows 10 or 11 (64-bit). The WebView2 runtime it uses is built into
  Windows 11 and present on almost all Windows 10 PCs.
- An admin account, so you can allow network control (Windows asks once
  per session).
- The driver for your USB-serial cable.

1. Open [Releases](https://github.com/illos/tunnel-term-releases/releases)
   and download `TunnelTerm-<version>-win64.zip` from the newest release.
   Releases marked *Pre-release* are field-test builds.
2. Optional: check the download. In PowerShell,
   `Get-FileHash .\TunnelTerm-<version>-win64.zip` should print the SHA-256
   in the release's `.sha256` file.
3. Unzip it anywhere, e.g. `Desktop\TunnelTerm`. Keep `cloudflared.exe`
   next to `tunnelterm.exe`.
4. Run `tunnelterm.exe`. The app isn't code-signed yet, so Windows shows
   "Windows protected your PC". Click **More info**, then **Run anyway**.

`README.txt` in the zip is a one-page quick start.

## How to: technician

### 1. Set up the laptop (once)

Paste the setup code from your admin into **Set up this laptop** and click
**Set up**. You only do this once per laptop.

![First run: the setup card, serial ports and network adapters](docs/screenshots/tech-1-first-run.png)

### 2. Use the console on site

Click **Open** next to the COM port. The default is 115200 8N1, and settings
are remembered per port. Close PuTTY first: only one program can use a COM
port at a time.

![A local serial console](docs/screenshots/tech-2-local-console.png)

### 3. Share it with the engineer

Click **Start remote session**. Windows asks for permission: click **Yes**,
so the engineer can manage the test Ethernet port. If you click No, serial
and SSH still work. Send the engineer the **link**, then read out the
**pairing code**. Click **New code** if a second engineer needs to join.

![A live session: the engineer link, the pairing code, and the tunnel and network control status](docs/screenshots/tech-3-session-live.png)

### 4. While the engineer works

The engineer is listed under **Engineers**, with **Kick** next to their
name. Their tabs show up in your window, and you can both type. **+1 h**
extends the session.

![An engineer connected: their ping tab appears in the tech's window](docs/screenshots/tech-4-engineer-connected.png)

### 5. Files

Click **Files** to see what the engineer uploaded and what the device
fetched. Add files from the laptop with **Add files…**, or drag them onto
the window. The save button keeps a copy.

![The Files panel in the tech window](docs/screenshots/tech-5-files.png)

### 6. End the session

Click **End session** when the engineer is done. The link stops working,
the test port is put back, and the session's files are deleted. Your own
terminal stays open, marked *remote session ended*.

![After the session: the test port back on DHCP and the files gone](docs/screenshots/tech-6-after-end.png)

## How to: engineer

### 1. Join

Open the link from the technician, and enter your name and the pairing
code. The dash and capitals are optional. If your connection drops, the
page reconnects by itself and the terminal picks up where it left off.

![The pairing page](docs/screenshots/eng-1-pairing.png)

### 2. Work the console

Tabs are shared with the technician. The toolbar has **Send BREAK**, a
**Ctrl** menu, **Paste…**, live serial **Settings** (baud, parity…) and
**Download scrollback**, which saves the tab's recent output to your own
computer.

![The shared console with the Network panel](docs/screenshots/eng-2-console.png)

### 3. Configure the test port

Open **Network**, then **Change** on the test adapter. You can set the VLAN
ID, a static IP or DHCP, add routes, or bounce the adapter. Every change
shows a countdown. Click **Keep** within 60 seconds or it undoes itself,
like `commit confirmed`. If a change cuts you off, it reverts and you're
back.

<img src="docs/screenshots/eng-3-vlan-dialog.png" width="49%" alt="Setting a VLAN ID"> <img src="docs/screenshots/eng-4-auto-undo.png" width="49%" alt="The auto-undo countdown with Keep and Undo">

### 4. Reach the device through the laptop

Use **Open…** for SSH, telnet, raw TCP, ping, traceroute or a TCP port
check. They run on the laptop, so they reach the device's network. SSH
logins happen inside the terminal, like OpenSSH: confirm the host key, then
type the password. Passwords are never stored.

<img src="docs/screenshots/eng-5-open-connection.png" width="49%" alt="Opening a ping through the laptop"> <img src="docs/screenshots/eng-6-ping.png" width="49%" alt="Pinging the device over the test port">

### 5. Paste a config

**Paste…**, or Ctrl+V with several lines, shows a preview first. Lines go
out one at a time, each waiting for the device's echo, so nothing is
dropped. You can cancel mid-paste.

<img src="docs/screenshots/eng-10-paste.png" width="49%" alt="The paste preview with line endings and pacing"> <img src="docs/screenshots/eng-11-pasting.png" width="49%" alt="A paced paste in progress">

### 6. Get firmware onto the device

Open **Files**, then **Upload…** or drop files on the page. Big files go up
in pieces; if one stops, click **Retry**. Under **Serve to devices**, pick
the test adapter and TFTP and/or HTTP, then click **Start serving**. Each
file gets a URL to paste into the device's copy command. **Device
transfers** lists what the device fetched, and suggests the right name when
it asks for one that doesn't exist.

<img src="docs/screenshots/eng-7-serving.png" width="75%" alt="Files served to the device over TFTP and HTTP"> <img src="docs/screenshots/eng-8-transfers.png" width="22%" alt="Device transfers with a did-you-mean hint">

### 7. Push and pull over SFTP

**Push to device…** and **Pull from device…** open a tab that logs in like
SSH. It shows progress, then a summary with the SHA-256. Pulled files appear
in the Files list.

![An SFTP push completed](docs/screenshots/eng-9-sftp.png)

### 8. When it ends

The technician ends the session, or it times out. You'll see why it ended.

![The session-ended screen](docs/screenshots/eng-13-ended.png)

Dark mode follows your system setting:

![The engineer view in dark mode](docs/screenshots/eng-12-dark.png)

## Limits in this version

- Files: 2 GiB per file, 4 GiB per session. Files exist only during a
  remote session.
- Serial flow control: none only.
- SFTP is supported. SCP isn't yet.
- One test VLAN per adapter (it uses the network card driver's VLAN ID
  setting).

## Updates

The app shows **Update available** next to its version when a newer
release is out. Download the new zip and unzip it over the old folder; your
settings and setup are kept. The app never downloads or installs anything by
itself, and the check can be turned off in Settings.

---

*Screenshots are from a test lab: a Windows 11 VM with a simulated router.
Addresses, names and the link are examples.*
