---
title: "Fabrication Lab"
author: Silas
date: 2025-11-19 07:39:34 -0800
categories: [fabrication lab]
tags: [fabrication lab, infrastructure, 3d printing, automation]
render_with_liquid: false
---

# Fabrication Lab

The fabrication lab is the physical side of my engineering setup — a small but fully networked space where I prototype hardware, print production parts, and test automation workflows. It runs much like a software stack: modular, containerized, and built for reliability.

---

## Core Infrastructure

- **Unraid Server:** Central hub running multiple OctoPrint containers through a single USB hub. This setup lets me manage three printers independently without juggling cables or devices.  
- **WireGuard VPN:** Integrated directly into Unraid for secure remote access — I can start, monitor, or pause prints from anywhere.  
- **Pixel Phone Hack:** A retired Pixel phone serves as an additional OctoPrint host. It’s been rock-solid for three years, and it solved both the USB mapping headaches and the Raspberry Pi shortage problem.

---

## Network Layer

- **Pi-hole:** Handles local DNS and ad-blocking, so all lab services live under easy-to-remember hostnames (e.g., `print1.local`, `slicer.lab`).  
- **NAS Storage:** Acts as the lab’s single source of truth — storing 3D models, client documentation, fabrication logs, and photo archives of completed work.

---

## Lessons Learned

- USB mapping with identical printers is fragile; container isolation helped but didn’t eliminate quirks.  
- Repurposed hardware like the Pixel phone can outperform “maker-grade” boards in stability and heat management.  
- Phone-based time-lapse capture looks great but leads to thermal throttling — not worth the downtime.

---

## Current Focus

I’m building a **fully containerized fabrication workflow** — from CAD modeling and slicing to job submission and monitoring. The goal is a **remote-first production pipeline** that lets me move from design to print from any device with full traceability and feedback.

---

## Next Steps

- Expand container orchestration to include slicing and monitoring nodes.  
- Explore lightweight, passive camera options for stable time-lapse capture.  
- Continue refining remote workflows to support live client demos and scaled production runs.

---

This lab is where digital design meets real-world fabrication — a quiet blend of code, hardware, and process that keeps evolving alongside my software stack.

## 5 Years of My 3D Printing Lab

<iframe width="560" height="315" 
src="https://www.youtube.com/embed/-sjD4Fp9mOI" 
title="YouTube video player" frameborder="0" 
allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
allowfullscreen></iframe>


