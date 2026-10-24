# Tunnel Term — downloads

Release builds of **Tunnel Term**, a Windows tool that lets a remote engineer
use a field technician's laptop (serial console and test network port) from
a browser. The source code is private; this repository only hosts the
downloads.

The app is useless without a **setup code** from your admin.

## Get it

1. Open [Releases](https://github.com/illos/tunnel-term-releases/releases)
   and download `TunnelTerm-<version>-win64.zip` from the newest release.
   Releases marked *Pre-release* are field-test builds.
2. Optional: check the download. In PowerShell:
   `Get-FileHash .\TunnelTerm-<version>-win64.zip` should print the SHA-256
   from the `.sha256` file of the same release.
3. Unzip it anywhere, e.g. `Desktop\TunnelTerm`. Keep `cloudflared.exe`
   next to `tunnelterm.exe`.
4. Run `tunnelterm.exe`. The app isn't code-signed yet, so Windows shows
   "Windows protected your PC": click **More info**, then **Run anyway**.

`README.txt` inside the zip is the quick start for techs. The app shows a
notice when a newer version is out. It never updates itself: download the
new zip and replace the folder.
