---
layout: project
title: Engineering Materials - Torque Wrench
description: Class project 
technologies: [Fusion 360, ANSYS Workbench]
image: /assets/images/render_2.png
---


As a final project for my mechanics of engineering materials class, I have designed and tested a torque wrench, using Fusion 360 to make the CAD model, and ANSYS Workbench to perform FEM analysis. The wrench has to comply with a set a requirements
- Sensitivity of a strain gauge of at least 1.0 mv/V output at the rated toruq of 600 in-lbf.
- safety factor of X0 = 4 for yield failurure
- safety factor of Xk = 2 for crack growth from an assumed crack depth 0.04 inches
- fatigue stress safety factor of Xs = 1.5
- material must be a steel, aluminum or titanium alloy 

After performing mathematical analysis using the beam theory, which turns out to be a great approximation for this case, I came up with a design and built a CAD model for the torque wrench. 

<img src="{{ 'assets/images/render_1.png' | relative_url }}" alt="Render 1" width="275">
<img src="{{ 'assets/images/render_3.png' | relative_url }}" alt="Render 3" width="275">

The CAD model details with the dimensions:

<img src="{{ 'assets/images/cross-section.PNG' | relative_url }}" alt="Cross-Section" width="275">
<img src="{{ 'assets/images/wrench_head.PNG' | relative_url }}" alt="wrench head" width="275">

<img src="{{ 'assets/images/wrench_body.PNG' | relative_url }}" alt="wrench body" width="552">

The material I picked is AISI4340 low-allow normalized steel. Particularly, this steel is has a strong resistence against fracture failure. 
Its Young's Modulus is 29.7e6 psi, and its fracture toughness is 51900 psi per square inch. Those numbers help us yield appropirate safety factors against yield and fracture. 
AISI 4340 steel also has a large fatigue strenght, which yields a high safety factor against fatigue failure (the fatigue strength is 68500 for 10^6 cycles).
Therefore, AISI4340 steel is very strong and durable, making it a great candidate for tools such as skrewdrivers, torque wrenches, etc. 

A 600 lbf-in torque applied to the wrench base is equavalent to 600/18 = 33.3 lbs force applied at the end of the wrench handle. 
Therefore, when performing FEM in ANSYS Workbench, we add a 33.3 lbs force applied to the end of the handle, in the x direction. We also apply a boundary condition that the head of the wrench is not moving. 

<img src="{{ 'assets/images/force.PNG' | relative_url }}" alt="Force" width="552">

After meshing and solving, we can evaluate the strains and the stresses at any locations of the wrench. 

<img src="{{ 'assets/images/Strain_Gauge.png' | relative_url }}" alt="Strain Gauge" width="552">
<img src="{{ 'assets/images/max_stress_2.png' | relative_url }}" alt="Max Stress" width="552">
<img src="{{ 'assets/images/maximum_principal_stress.png' | relative_url }}" alt="diagram" width="552">

The strain gauge location is shown in the following diagram
<img src="{{ 'assets/images/diagram.PNG' | relative_url }}" alt="Render 1" width="552">

In our case b = h = 0.51'', c = 1'', L = 18''

From the principal stress and normal strain plots and solutions, we observe that normal stress is 54144 psi (25853 psi at the strain gauge location), load point deflection is 0.44'' and strain at the strain gauge location is 865.9 microstrains.

I decided to choose a full bridge strain gauge in order to extract maximum sensitibity. Thus, I have achieved a strain gauge output of 1.726 mV/V (sensitivity = (2.0 * strain)*10^3)

I also had to pick a srain gauge which would fit on the wrench (0.51 by 0.51 inches area), and which is full bridge. A choice I decided to go with is HBM VY4x-3/350 strain gauge. It is full-bridge, thus allowing higher sensitivity. It is 3mm by 3mm, which makes it easily fit on the wrench. 
[Strain Gauge Page](https://www.hbm.com/tw/3446/vy-full-bridge-strain-gauges-with-4-measuring-grid/?product_type_no=VY%20Full%20Bridge%20Strain%20Gauges%20With%204%20Measuring%20Grids)