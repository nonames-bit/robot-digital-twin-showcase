# Security

Connecting a PC to an industrial controller is a safety and security decision, not only a technical one. This document explains how the project is designed to stay safe, and what to harden before using anything like it on a real plant.

## Design principles

1. **Read-only by construction.** The HSES client implements only read requests (status, position, alarm, I/O read). There is no code path that sends a motion command, writes a variable or changes the robot's mode.
2. **No write path from the HMI.** The robot tags are exposed as read-only values. Operator controls in the views change only local view state (language, clean mode).
3. **Fail safe.** If communication fails, the twin marks the data as stale and counts errors. It never retries by "forcing" anything on the controller.
4. **Least data.** Only the values the twin needs are published (axes, TCP, status, alarm, tool outputs).

## Before connecting to a production controller

- Get **written authorization** from the cell owner and from IT/OT security.
- Reading works in teach and play modes and needs neither remote mode nor a higher security level on the controller. Do not change either for this project.
- Prefer a **dedicated, segmented network** or a port agreed with the plant. Allow only the HSES UDP port, only between the bridge host and the controller.
- Use a **separate, isolated host** for the bridge. Do not expose it to the office or guest network.

## Hardening checklist for the software side

| Area | Recommendation |
| :-- | :-- |
| Bridge service | Bind the API and the OPC-UA endpoint to `127.0.0.1` unless another host really needs them. Firewall the rest. |
| OPC-UA | A demo can run without security; for real use enable `SignAndEncrypt` with certificates, and a user/password or certificate identity. |
| Ignition Gateway | Change all default credentials, enforce strong passwords, use HTTPS, give operators least-privilege roles, disable anonymous access. |
| Updates | Keep Ignition and its modules patched; pin dependency versions. |
| Secrets | Keep credentials out of the repository and out of screenshots. Use environment variables or a secret store. |
| Logging | Log connection errors and who changed what; review them. |
| Backups | Back up the Ignition gateway and project; never run an untested `docker run` over a configured gateway. |
| Data | Treat recordings of real cell positions as plant data: keep them private. |

## What this repository deliberately leaves out

- Network addresses, hostnames and ports of the real plant.
- Credentials and tokens of any kind.
- Recordings and logs from the real cell.
- Names and photos of people.

If you reuse this approach, apply the same rule to your own screenshots and videos: crop out the gateway address, user names, tag paths that identify the company, and anything on the screen that is not the demo.

## Reporting an issue

If you notice something that looks unsafe in what is described here, please open an issue without including any plant-specific data.
