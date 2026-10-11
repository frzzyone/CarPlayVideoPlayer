# Carlinkit Mini Ultra 3: reverse engineering dossier (October 2026)

Goal: run our own firmware on a Mini Ultra 3 so that it accepts AirPlay mirroring from an
iPhone and forwards the picture to the car through the dongle's wired CarPlay output. The
Carlinkit Mini Ultra 6 ships this as a feature, so the hardware class is known to be capable.

## 1. What a Mini Ultra 3 is, physically

Carlinkit reuses the "Mini Ultra" name across unrelated boards and changes the PCB without
changing the product ID. Three Ultra 3 variants are reported:

| Variant | SoC | Radio | Stock firmware family | Open firmware status |
|---|---|---|---|---|
| A (most reports) | Allwinner V821B, 32-bit RISC-V, 64 MB RAM, 16 MiB NOR | AIC8800D80 | `cpbox-abroad` ("Liaoyuan", lighttpd web UI at 192.168.50.100) | LIVI Link: supported (`v821b_aic8800d80`). catplay: port in progress, no public build, test builds on request. ocbm: "C2Air" port, OCBM daemons proven |
| B | Axera AX520CE | AIC8800D80 | `cpbox-abroad` | LIVI Link: supported (`ax520_aic8800d80`). catplay: queued |
| C | Anyka AK3918AN (ARM9) or Allwinner V851SE, firmware reports `product_Model: v851se-yunlian-cp`, `custom: yunlian` or `suding` | unknown | different vendor stack, `/cgi-bin/index.cgi?id=host` returns 404 | Nothing supports it. catplay maintainer says an Anyka port may come |

Mini Ultra 1 is a fourth hardware family (Ingenic X1600EN + AIC8800D80, later V821B +
RTL8733BS) and is the one catplay ships stable builds for. It is not an Ultra 3.

## 2. Identify the variant before doing anything

Non-destructive. Plug the dongle into USB power, join its Wi-Fi hotspot (password `12345678`
or `88888888`), then from a browser or curl:

```
http://192.168.50.100/cgi-bin/getversion.cgi
http://192.168.50.100/cgi-bin/index.cgi?id=host
```

Read `product_Model`, `custom`, `platform`, `system_version`. If `index.cgi?id=host` is 404 and
`product_Model` contains `yunlian`, it is variant C and none of the open tooling applies yet.
Also note the USB ID the dongle enumerates with on a computer: stock firmware presents as an
Apple iPhone (`05ac:12a8`).

Opening the case and reading the SoC and radio markings settles it definitively. One owner
also found Mini Ultra 1 units whose MFi chip package was a dummy, so the MFi chip marking is
worth photographing too.

## 3. Prior art we stand on

- **catplay** (catplay-labs/catplay). A from-scratch Rust implementation of CarPlay with no
  Apple SDK: iAP2 over Bluetooth and USB, Bonjour, HomeKit pairing, FairPlay, MFi, and both
  `catplay_carplay_rx_gadget` (receiver toward the iPhone) and `catplay_carplay_tx_gadget`
  (transmitter posing as an iPhone toward the car). Ships Yocto firmware for Mini Ultra 1
  and the V821B + RTL8733BS boards. Its wizard flows: patch a vendor OTA, reboot to Allwinner
  FEL, load a recovery system, back up the 16 MiB flash, flash. No LICENSE file at the repo
  root as of this writing. Only iOS 26+ tested. Known car incompatibilities exist.
- **LIVI Link** (f-io/LIVI). Reflashes a dongle into a network accessory: MFi over the
  network, Wi-Fi AP, Bluetooth. All CarPlay logic runs on the LIVI host. `livi-link-provision`
  takes a full backup first, has a rescue system reachable at `telnet 10.10.10.1` over USB
  NCM, and restores stock from the backup via its web UI. Supports V821B and AX520 with
  AIC8800D80, which covers Ultra 3 variants A and B. Does not do what we want by itself (it
  is a head unit, not a transmitter) but it is the proven, reversible foothold.
- **ocbm** (lvalen91/ocbm). Open Rust userspace for the CPC200-CCPA with a "C2Air" port to an
  Allwinner V821 RISC-V board with AIC8800 radio, kernel 5.4.220, ADB over functionfs. Public
  domain. Receiver only; decoding happens on a host.
- **ludwig-v** dongle reverse engineering. The i.MX6UL generations only. Documents Carlinkit's
  activation lock (`/etc/uuid_sign`) and the SPI flash dump route. Background, not directly
  applicable.
- **UxPlay** (GPL-3). Mature open AirPlay mirroring receiver (mirroring and audio, not AirPlay 2
  video). Reference for the mirroring protocol, FairPlay for mirroring, and the H.264 path.

## 4. Why the V821B platform is the right target

catplay's workspace already has the two halves this project needs on one board: a receiver
stack that pairs with the iPhone using the dongle's MFi chip, and a transmitter stack that the
car accepts as an iPhone. What is missing is an AirPlay mirroring receiver feeding the
transmitter's video channel instead of the CarPlay receiver's. That is one new module on a
stack that already decodes nothing and forwards H.264, which is exactly the constraint a 64 MB
RISC-V dongle imposes.

## 5. Constraints and risks

- Flash writes can brick the unit. Every tool above takes a backup first. Keep it off-device.
- Carlinkit runs an activation server. A stock image restored from backup keeps its
  activation; a foreign stock image may come up locked.
- Variant C has no open foothold. Buying an Ultra 3 today is a lottery between A, B and C.
- Only the A and B variants have a known stock-to-open path, and for A the catplay build does
  not exist yet. The first real milestone is a backup plus a shell, not AirPlay.
- Netflix will never work through mirroring. Jellyfin, Plex, YouTube and anything without DRM
  will.
- Video visible to the driver is illegal in many places. Parked use only.
- catplay's license is not stated. Clarify before depending on it for anything distributed.

## 6. Sources

- https://github.com/catplay-labs/catplay and its firmware docs
  https://github.com/catplay-labs/catplay-firmware/blob/master/docs/V821.md
- https://github.com/catplay-labs/catplay/issues/31 and /issues/32
- https://github.com/f-io/LIVI/blob/main/LIVI-LINK.md and https://github.com/f-io/LIVI/issues/368
- https://github.com/lvalen91/ocbm (c2air/)
- https://github.com/ludwig-v/wireless-carplay-dongle-reverse-engineering
- https://github.com/fdh2/uxplay
- https://www.carlinkit.com/faq_firmware_update_issue.html
