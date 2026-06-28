# NetJutsu

A native macOS debugging proxy for inspecting and mocking HTTP/HTTPS traffic — a free, open alternative to Charles and Proxyman, focused on the things mobile developers actually reach for: clean HTTPS inspection, unlimited Map Local mocking, and one‑click setup across Mac, iOS, and Android.

> **Status: 1.0 Beta.** Usable day to day; some rough edges. Feedback and issues welcome.

---

## What it does

- **TLS interception (MITM).** Opt‑in per host — traffic is blind‑tunneled by default, and you enable SSL proxying only for the hosts you want to decrypt. Right‑click a host → *Enable SSL Proxying*.
- **Full HTTP inspection.** Request/response headers and bodies, gzip‑decoded, with JSON and XML pretty‑printing and syntax highlighting.
- **Map Local (unlimited).** Return a canned response for any matching request — wildcards supported, rules persist across launches, and each rule opens in its own resizable editor. Great for forcing edge cases (empty feeds, error codes, specific ad markup) without touching a server.
- **Import from Proxyman.** Bring your existing Map Local rules over from Proxyman’s `.config` exports (*Map Local ▸ Import ▸ From Proxyman*).
- **Automatic Mac system proxy.** NetJutsu points your Mac at itself on launch and cleans up on quit.
- **Mobile support.** Guided setup for the iOS Simulator, a real iPhone (scan a QR to install the certificate), and the Android emulator.

NetJutsu is built for inspecting traffic **you control** — your app’s API calls, your test endpoints. Like every HTTP debugging proxy, it can’t see QUIC/HTTP‑3 traffic or hosts that use certificate pinning (e.g. inspecting Instagram, Reels, or other hardened third‑party apps won’t work — that’s a property of those apps, not a NetJutsu limitation).

---

## Requirements

- macOS 26 or later
- For mobile: Xcode (Simulator), or an Android SDK / emulator (`adb` on your `PATH` or in the standard SDK location)

NetJutsu runs with the App Sandbox **off**, because it configures your system network settings (`networksetup`). This is the same reason Charles and Proxyman aren’t on the Mac App Store, and it’s why NetJutsu is distributed directly here on GitHub rather than through the store.

---

## Install

1. Download the latest `NetJutsu.zip` from [Releases](../../releases).
2. Unzip and drag **NetJutsu.app** to your Applications folder.
3. Open it. Release builds are signed and notarized, so it launches without a Gatekeeper warning.

On first launch NetJutsu generates its own root certificate authority (CA), stored on disk with the private key in your Keychain. Nothing is inspected until you trust that CA and enable SSL proxying for a host — see below.

---

## Trusting the certificate

Decrypting HTTPS requires the client to trust NetJutsu’s CA. Each platform has its own trust store, so you trust it once per device/target. Everything below is automated or guided from the in‑app **Setup** menu (top of the window).

### Install on Mac

Open the NetJutsu app and navigate to: `Setup ▸ Install Certificate on Mac`

macOS will prompt for your admin password (that’s the OS, not NetJutsu). The sheet has a one‑click button; the manual command is shown there too as a fallback. **Restart your browser afterward** — Chromium caches trust at launch.

> **Note on trust stores:** the macOS keychain covers Safari and Chrome/Vivaldi. **Firefox** and **curl** ship their own trust stores and won’t use the keychain — for curl, pass `--cacert <path-to-netjutsu-ca.pem>`.

### Install on the iOS Simulator

Open the NetJutsu app and navigate to: `Setup ▸ Install Certificate on iOS Simulator`

### Install on a real iPhone

Open the NetJutsu app and navigate to: `Setup ▸ Install Certificate on iPhone`

1. Set the iPhone’s Wi‑Fi proxy to the Mac address shown in the sheet (Settings ▸ Wi‑Fi ▸ ⓘ ▸ Configure Proxy ▸ Manual).
2. Scan the QR code to open `http://netjutsu.profile` in Safari and download the profile.
3. Install it: Settings ▸ *Profile Downloaded* ▸ Install, then enable it under Settings ▸ General ▸ About ▸ Certificate Trust Settings.

### Install on the Android emulator

`Setup ▸ Install Certificate on Android Emulator`

Android apps ignore user‑installed CAs by default, so rather than rooting the emulator, NetJutsu uses the **network‑security‑config** approach — your *debug* build opts into trusting user certificates. No rooted or writable‑system image required.

1. **Push the certificate and set the proxy** (one click in the sheet). This `adb push`es the CA to the emulator’s Downloads and points its proxy at `10.0.2.2:9988` (the emulator reaches your Mac at `10.0.2.2`, not the LAN IP).
2. **Install it** in the emulator: Settings ▸ Security & privacy ▸ More security settings ▸ Encryption & credentials ▸ Install a certificate ▸ CA certificate ▸ pick `netjutsu-ca.cer` from Downloads.
3. **Let your debug build trust user certs** — add `res/xml/network_security_config.xml`:

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <network-security-config>
       <debug-overrides>
           <trust-anchors>
               <certificates src="system" />
               <certificates src="user" />
           </trust-anchors>
       </debug-overrides>
   </network-security-config>
   ```

   and reference it in `AndroidManifest.xml` on the `<application>` tag:

   ```
   android:networkSecurityConfig="@xml/network_security_config"
   ```

   `<debug-overrides>` applies to debug builds only — your release builds are unaffected.

---

## Using it

1. **Trust the CA** and set the proxy for your target (above).
2. Hosts appear in the list as they connect. By default they’re blind‑tunneled (you’ll see `CONNECT` rows).
3. **Right‑click a host → Enable SSL Proxying** to start decrypting its traffic. Requests then show full headers and bodies.
4. **Map Local:** open *Map Local*, add a rule (or import one), and set the URL pattern, method, status, headers, and body. Enabling a rule auto‑enables SSL proxying for its host so it can match.

Use **Pause/Resume** to stop proxying without quitting, **No Caching** to add cache‑busting headers, and **Clear** to empty the request list.

---

## Known limitations

- **HTTP/2 is forced to HTTP/1.1** when intercepted; tunneled (uninspected) hosts keep full h2/h3. **HTTP/3 (QUIC) is not inspected**
- **Certificate‑pinned hosts** can’t be intercepted (the client rejects the forged certificate). This is by design on the client’s side

---

## License

NetJutsu is **free to use** but is **not open source**. It is distributed as a
compiled application under a freeware license — see [LICENSE.txt](LICENSE.txt).

In short: you may use it freely, but you may not reverse‑engineer, modify, or
redistribute it, and it is provided **as is, without warranty of any kind**.