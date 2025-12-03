---
title: "One Mini PC, Many Jobs: How I Run My Fabrication Lab on an i7-6700T"
author: Silas
date: 2025-12-02 22:34:52 -0800
categories: [Fabrication Lab  Infrastructure]
tags: [Fabrication Lab  Infrastructure]
render_with_liquid: false
---

Most people look at Raspberry Pis for lab automation and small hosting jobs. I’ve used them plenty, but for a real production workflow — printers, cameras, network services, and backups — they just don’t have the headroom.  

So I’ve been running my fabrication infrastructure on a single **EliteDesk G3 Mini** powered by an **Intel Core i7-6700T @ 2.8 GHz**. It’s quiet, efficient, and handles everything I throw at it without breaking 20 % CPU utilization.

---

## The Stack

Running **Unraid** as the base lets me spin up and manage multiple containers cleanly. Current lineup:

| Container | Purpose |
|------------|----------|
| **duckDNS** | Dynamic DNS for secure remote access |
| **Pi-hole** | Network-wide ad blocking and DNS cache |
| **Heimdall** | Dashboard for quick access to internal services |
| **OctoPrint – The Forge** | Production printer #1 |
| **OctoPrint – The Anvil** | Production printer #2 |
| **OctoPrint – The Billows** | Production printer #3 |

Each OctoPrint instance runs in its own container with dedicated USB passthrough and config volumes. I can watch all three cameras, start or pause jobs, and handle slicing uploads directly through the web UI.

---

## Why Not a Raspberry Pi?

When you factor in a Pi 5, case, PSU, and active cooling, you’re already near **$150–$170**.  
My used EliteDesk cost about the same — but it has:

- Quad-core x86 performance  
- 16 GB RAM  
- Real NVMe storage  
- Built-in virtualization support  
- And plenty of power headroom  

Under full load with three printers running and file shares active, CPU sits around **20 %**, temps under **60 °C**, and the box idles around **30 W**. For a small-form-factor PC that’s driving a production lab, that’s nothing.

---

## Integrated Services

Since the machine never struggles, I also use it for light file hosting:

- Company docs and design models live on an Unraid share  
- Automatic backups replicate to my personal workstation (an old gaming rig with a GTX 970)  
- Once a month, that system syncs to an offline drive for cold storage  

It’s simple, cheap, and resilient — no cloud dependence, no recurring cost.

## Simple Management

Rebooting or restarting services doesn’t require extra hardware like KVM switches or smart plugs.  
With Unraid’s container interface, it’s literally a click: **restart** or **stop**.  

No physical labor, no added cost — just one dashboard controlling the entire lab.  
I can spin up, tear down, or update services in seconds without touching a single cable.

---

## Remote Workflow

Everything runs behind a **WireGuard VPN**.  
From anywhere, I can open a secure tunnel, check printer status, or even start a prototype job mid-meeting if a production printer’s free.  

That setup means the fabrication lab behaves like a remote node of my studio network — same access controls, same file structure — just physically separate.

---

## Why It Works

A single small PC with real cores and storage throughput beats clusters of underpowered boards every time. It’s easier to manage, cheaper to cool, and scales horizontally through containers instead of extra hardware.

The result is a **self-hosted manufacturing hub** that’s efficient, secure, and fully under my control — no cloud, no middlemen, just local compute doing real work.

---

### Takeaway

If you’re running multiple printers or small fabrication systems, look beyond hobby boards. A used business-class mini PC with Unraid can run half a dozen services, three production printers, and your network backbone — all at Raspberry Pi power levels and price.

Sometimes “upcycling” enterprise hardware is the real optimization.
