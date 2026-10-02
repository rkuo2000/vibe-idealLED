# idealLED Curtain Studio

A dependency-free progressive web app implementing the BLE protocol from
[8none1/idealLED](https://github.com/8none1/idealLED).

## Run

Keep `idealLED.html`, `idealLED.webmanifest`, `idealLED-sw.js`, and both PNG icons together.
From this folder run:

```sh
python3 -m http.server 8000 --bind 127.0.0.1
```

Open **http://localhost:8000/idealLED.html** in a supported Chrome or Edge browser.
Bluetooth needs a secure context: use HTTPS for a hosted deployment or for an Android
phone accessing a server on another computer. An ordinary LAN HTTP address will not work.
Web Bluetooth availability also depends on the browser, OS and Bluetooth adapter.
Safari and Firefox cannot connect with this API.

1. Turn on Bluetooth and power the curtain. Close its phone app to release its connection.
2. Select Type 1 or Type 2 under Connection settings if you know the firmware.
   Type 2 is the upstream default; there is no automatic firmware detection.
3. Click **Connect curtain** and choose the IDL- or ISP- device. If its name differs,
   enable **Show all Bluetooth devices**. Only compatible controllers will work.
4. Select color, brightness or an effect, then click **Apply to curtain**.
   On and Off send immediately. Apply does not implicitly turn the curtain on.
5. Use the browser's install action, or the page's Install app button when available.
   After its first successful online load, the app shell is available offline.
   Bluetooth access and device selection still require browser permission.

## Protocol

Service: `0000fff0-0000-1000-8000-00805f9b34fb`.
Commands: `d44bc439-abfd-45a2-b575-925416129600`.
Palette data: `d44bc439-abfd-45a2-b575-92541612960a`.
Notifications: `d44bc439-abfd-45a2-b575-925416129601`.
Notifications are enabled before any user commands.

Commands are exactly one AES-128-ECB block, with the key documented upstream.
Web Crypto AES-CBC with a zero IV produces that same first block; its trailing
padding block is discarded. There are no external scripts or runtime dependencies.
Type 1 has power and solid color controls; Type 2 additionally has effects 1–10,
reverse direction and speed. Solid colors use brightness-scaled 5-bit channels.
Type 2 effects send an encrypted header followed by the upstream format's single
23-byte unencrypted rainbow palette. Firmware may vary in effect appearance and
brightness behavior. A palette write failure can leave the effect partially applied;
select solid color and apply again to recover. No arbitrary commands, LED-count
configuration, timers, or firmware updates are exposed.

The curtain illustration represents selected settings. It does not reconstruct actual
curtain geometry or device state. Notifications are logged as raw bytes, and writes
are reported as sent, not verified physical changes. Neither reconnecting nor loading
the page sends a power or scene command.

## Validation

```sh
node tests/protocol.test.cjs
node tests/offline.test.cjs
node --check idealLED-sw.js
```

Tests cover captured on/off/red ciphertext, both color layouts, effect bounds,
palette framing, connection without writes, write ordering, disconnect, cancellation
and write-error recovery. A separate test checks cached offline serving and asset availability. The BLE tests use a simulated device. Physical BLE hardware
and browser installation/offline behavior need validation on your target device.

Protocol-derived code is attributed to Will Cooke / 8none1 under the MIT license;
see `LICENSE-idealLED.txt`.
