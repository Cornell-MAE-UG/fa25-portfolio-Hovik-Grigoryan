---
layout: project
title: Aerospace Adversary Lab Project - Fiber Optic Sensor
order: 1
description: Fiber Optic Based Laser Detector 
technologies: [Fusion 360, MATLAB, 3d printing]
image: /assets/images/fiber1.JPG
---

As an undergraduate researcher in Cornell Aerospace Adversary Lab, a designed and built a fiber optic-based sensor which detects when a laser hits the surface of a curved surface. The device consists of a curved surface (1/8th of a sphere), camera mount and enclosures. 

<img src="{{ 'assets/images/fiber2.JPG' | relative_url }}" alt="Render 1" width="200">

16 small fiber cables transfer the light to a 4 by 4 grid. The camera is mounted right below the grid, and it detects which signal is on and which one is off, as well as their relative intensities. 
The MATLAB script takes the information from the camera, backtracks the information to the surface of the sphere (through mapping), and the implements an interpolation algorithm to calculate the location on the surface that is getting hit by a laser. 
I have also contributed in the creation of a special UI (MATLAB based), which reports results real-time. 

<video controls muted style="max-width:300; border-radius:8px; box-shadow: var(--shadow); margin: 1.5em 0;">
  <source src="{{ '/assets/videos/your-video.mp4' | relative_url }}" type="video/mp4">
  Your browser doesn't support embedded videos.
</video>