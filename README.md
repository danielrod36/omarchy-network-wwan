# daniel.network — Omarchy network widget with cellular (WWAN/LTE) support

A fork of Omarchy's stock [`omarchy.network`](https://github.com/basecamp/omarchy/tree/main/shell/plugins/panels/network)
bar widget that also understands mobile broadband modems.

## Why

Omarchy's network panel is built on Quickshell's `Networking` service, whose
`DeviceType` covers only `Wifi` and `Wired` — a WWAN modem is invisible to it.
With LTE as your only link, the widget reads **disconnected** and offers no way
to manage the connection.

This fork adds cellular as a polled `mmcli`/`nmcli` probe alongside the native
service (Quickshell cannot see modems, so there is nothing to bind to):

- Bar pill tracks the modem even with the panel closed (15s cadence, 4s open)
- Signal bars, access technology (2G/3G/4G/5G) and carrier name in the hero
- CELLULAR section listing GSM connection profiles — click to connect, click
  again to drop the data bearer
- Cellular radio kill switch in the hero (same slot as the Wi-Fi one), plus an
  `omarchy-shell omarchy.network toggleWwan` IPC route
- Throughput/ping stats keep working: they ride the existing
  `omarchy-network-status` pipeline for whatever interface is routed

Profiles are created once with `nmcli` (`nmcli connection add type gsm …`);
the panel connects and disconnects them but never edits APN or SIM settings.

## Install

```bash
omarchy plugin add https://github.com/danielrod36/omarchy-network-wwan.git
```

(Replaces the stock network widget — the clone owns the `omarchy.network` IPC
target, so existing keybinds and `omarchy-shell` routes keep working.)

Requirements on the machine: `modemmanager` (daemon + mmcli), `networkmanager`,
`jq`. With no modem present the probe prints nothing and every cellular affordance
stays hidden — behaviour is identical to stock.

## Layout

This repository is the plugin directory itself; the layout mirrors upstream:

```
manifest.json   # id: daniel.network (renamed from omarchy.network)

Model.js        # pure logic: probe script, parsers, icons, labels
Panel.qml       # bar widget + popup panel
```

Offered upstream as [omacom/omarchy#10722](https://github.com/omacom/omarchy/pull/10722);
HEAD tracks that branch (rebased onto quattro's captive-portal rework).
Once merged there, this clone can be retired in favour of the stock widget.

## Tracking upstream

The first commit of this repo is the stock plugin (omarchy 4.0.2), the second
is the `omarchy plugin clone` rename, and everything after is the cellular
feature. To pick up upstream changes:

```bash
git remote add omarchy https://github.com/basecamp/omarchy.git
git fetch omarchy
git diff HEAD omarchy/main -- shell/plugins/panels/network  # upstream movement
```

The feature is also offered upstream; once merged there, this clone can be
retired in favour of the stock widget.
