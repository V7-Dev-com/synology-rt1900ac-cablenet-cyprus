# Synology RT1900ac as router on Cablenet (Cyprus) fiber

How to replace the provider router with a **Synology RT1900ac** (SRM) on a **Cablenet** GPON connection behind a **Huawei OptiXstar HG8010Hv6** ONT, and how to hand out **Pi-hole** as DNS for the whole network.

**Result that worked:** Internet connection type **Automatic (DHCP)** with **VLAN ID 42** on the WAN side. No username or password needed.

> Status: the Internet part (DHCP + VLAN 42) was verified (ping 3 ms, HTTPS works). The Pi-hole DNS part is the standard SRM setting but was not yet verified on this line when this was written. Menu names are from SRM in German translated back to English and may differ slightly by SRM version. Corrections welcome as issues or pull requests.

## What you have

| Device | Role |
|---|---|
| Huawei OptiXstar HG8010Hv6 | **ONT only** (fiber modem, bridge). No routing, no DHCP, no Wi-Fi. Its management page `192.168.100.1` is usually **not** reachable from your home network. |
| Provider router (here: Nokia Beacon G3.1) | Did the Internet login. It used **DHCP with VLAN 42**, not PPPoE. |
| Synology RT1900ac | Replaces the provider router. Must run in **Wireless Router** mode, **not** Access Point mode. |

An access point has no connection to the Internet by itself. If you set the Synology up as an access point and removed the old router, it falls back to a link-local address and nothing works (see troubleshooting).

## Cabling

| Plug | Into Synology port |
|---|---|
| Huawei ONT | **WAN** |
| Your computer | LAN 1 |
| Raspberry Pi / other wired devices | LAN 1 or LAN 2 (see note about LAN 3 and 4) |

## Setup

1. **Open SRM** at `http://192.168.1.1:8000` (after a factory reset, default address).
2. **Operation mode:** *Wireless Router*. Leave "External access to SRM" **disabled**.
3. **Network Center → Internet → Connection:**
   - Connection type: **Automatic** (DHCP)
   - Set as default gateway: **Enabled**
4. **VLAN 42 (without this there is no address):**
   - Click **ISP Settings (IPTV & VoIP)** (not the plain "ISP Settings" button).
   - Tick **Enable IPTV/VoIP**, mode **Manual**.
   - In the top section ("LAN 1, LAN 2 and Wi-Fi (Internet only)") set **VID = 42**, priority **0**.
   - Leave LAN 3 and LAN 4 empty. Do **not** pick an ISP profile such as "M1-Fiber".
   - **OK**, then **Apply**. Wait about one minute.
5. **Check:** the status page shows the connection as connected and a web page loads.
6. **Wi-Fi:** use the same name and password as before so devices reconnect on their own.

### Pi-hole for the whole network (optional)

1. Network Center → Internet → Connection → *Configure DNS server manually*:
   - Preferred DNS: your Pi-hole, e.g. `192.168.1.9`
   - Alternative DNS: `9.9.9.9` (keeps the network working if the Pi is down, without filtering)
2. Reserve a fixed address for the Pi-hole in the DHCP settings (Network Center → Local Network → DHCP clients / static assignment, depending on SRM version) using the Pi's MAC address.
3. Note: if the Synology forwards DNS, Pi-hole sees the router as the only client. For per-device statistics, hand out the Pi-hole directly via DHCP if your SRM version allows it.

## Troubleshooting

| Problem | What to try |
|---|---|
| No Internet, no WAN address | ONT in the **WAN** port? VLAN 42 saved? Some lines bind to the old device: in *ISP Settings* enter the **WAN MAC address of the old router** by hand (do not use the "Clone" button, it copies your computer's MAC). |
| Wired device not reachable after enabling IPTV/VoIP | Move its cable to **LAN 1 or LAN 2**. LAN 3 and LAN 4 belong to the IPTV/VoIP sections and may be separated from the rest. (Suspected cause, not confirmed.) |
| SRM page not reachable | Without DHCP the router uses a link-local address. Give your computer `169.254.1.2` with mask `255.255.0.0` and open `http://169.254.1.1:8000`. |
| Old router needed to read settings | Connect it **only** to your computer and disconnect the Synology (both use `192.168.1.1`). The PPPoE/DHCP password is often hidden in the web page. |

## Printable version

[`guide-printable.pdf`](guide-printable.pdf) has the same steps with checkboxes on three A4 pages.

## Notes

- Cablenet is a Cypriot ISP; this is based on one household's line, not official documentation. Other ISPs/regions use different VLANs.
- Do not post passwords, Wi-Fi keys or your public IP in issues.

License: MIT, see [LICENSE](LICENSE).
