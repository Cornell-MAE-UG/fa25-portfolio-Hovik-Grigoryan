---
layout: project
title: System Dynamics - Coffee Heater 
description: PID control system that heats up coffee
technologies: [MATLAB, System Dynamics]
image: /assets/images/coffee-heater-plate.jpg
---

As a final project for my system dynamics course, my teammates and I were assigned the task of creating, characterizing and analyzing a dynamic system. 
We decided to construct a coffee heater. Assuming that we have cold coffee, which has the same temperature as the air, given a reference input (the final desired temperature of the coffee), the coffee plate heats up in a PID control system. 

<img src="{{ 'assets/images/cofee-heater.PNG' | relative_url }}" alt="Coffee Heater" width="552">

We evaluated all the relevant equations which relate the various terms of the system with each other. 
Thus, the open-loop system has the following characteristic equation: 

<img src="{{ 'assets/images/open-loop-eqn.PNG' | relative_url }}" alt="Block Diagram" width="552">

Evaluating the Laplace transforms of all time-dependent variables, we derived a final transfer function equation. 

<img src="{{ 'assets/images/eqn.PNG' | relative_url }}" alt="Block Diagram" width="552">

Given all the various disturbances in the system, our PID control parameters, and the system structure, as well as the equations, we simplified the system into a block diagram, with the Laplace transforms of all the time-dependent variables. 

<img src="{{ 'assets/images/block_diagram.PNG' | relative_url }}" alt="Block Diagram" width="552">

Using the block diagram and the equations, we made a MATLAB script to plot how the temperature, error and the control effort (temperature of the hot plate) change in the system, thus evaluating important properties such as rise time, settling time and maximum control effort

<img src="{{ 'assets/images/temp-data.PNG' | relative_url }}" alt="Block Diagram" width="275"> <img src="{{ 'assets/images/error-data.PNG' | relative_url }}" alt="Block Diagram" width="275"> 

<img src="{{ 'assets/images/control-effort-data.PNG' | relative_url }}" alt="Block Diagram" width="552">

We managed to achieve a rise time of 144 seconds and a settling time of 487.5 seconds, all while keeping the control effort under 375 degrees celsius. 

In the team, my main responsibilities were writing the MATLAB script which performs the calculations and creates the plots. In the meantine, I have also helped my teammates with evaluating the relevant equations. 

<pre> 
```
My MATLAB code for the project

function [t, T, e, u] = coffee_pid(C, Gbot_min, Gcup_min, Kp, Ki, Kd, ...
                                T_init, Tref, Tair, tf)
% PID-controlled coffee heater system
% Gbot_min, Gcup_min are given in J/min*C

    % Convert J/min*C --> J/s*C
    Gbot = Gbot_min / 60;
    Gcup = Gcup_min / 60;

    tspan = [0 tf];

    % State vector:
    % x(1) = T  (coffee temperature)
    % x(2) = I  (integral of error)
    % x(3) = e_prev (for derivative)

    x0 = [T_init; 0; 0];

    function dx = dynamics(t, x)
        T = x(1);
        I = x(2);
        e_prev = x(3);

        e = Tref - T;

        d_e = (e - e_prev);
        inner_sum = Ki*I + e + Kd * d_e;
        u = Kp * inner_sum;
        dT = ( Gbot*(u - T) + Gcup*(Tair - T) ) / C;
        dI = e;
        d_e_prev = d_e;

        dx = [dT; dI; d_e_prev];
    end

    [t, X] = ode45(@dynamics, tspan, x0);

    T = X(:,1);
    I = X(:,2);
    e_prev = X(:,3);
    e = Tref - T;

    % Compute derivative of error using stored state
    d_e = e - e_prev;

    % Compute control effort u(t) for plotting
    u = Kp * ( Ki*I + e + Kd * d_e );

end

clc; clear; close all;
C     = 1000;     % J/C
G_bot = 80;      % J/min*C
G_cup = 20;      % J/min*C


Kp = 8;
Ki = 0.009;
Kd = 0;

T_init = 20;   % initial coffee temperature
T_ref  = 65;   % reference
T_air  = 20;   % ambient

tf = 1000;      % 16.6 minutes

[t, T, e, u] = coffee_pid(C, G_bot, G_cup, Kp, Ki, Kd, ...
                       T_init, T_ref, T_air, tf);


% Coffee Temperature 
figure;
plot(t, T, 'LineWidth', 2);
grid on;
xlabel('Time (s)');
ylabel('Temperature T(t) [°C]');
title('PID-Controlled Coffee Temperature');

% Error
figure;
plot(t, e, 'LineWidth', 2);
grid on;
xlabel('Time (s)');
ylabel('Error = T_{ref} - T(t)');
title('Tracking Error Over Time');

% Control Effort
figure;
plot(t, u, 'LineWidth', 2);
grid on;
xlabel('Time (s)');
ylabel('Plate Temperature Command u(t) [°C]');
title('Control Effort (Heater Plate Temperature)');

```
</pre>

We came to a conclusion that that the PI control is the best option. When the derivative control coefficient is above 0, we end up getting a very large value of the initial control effort (as well as very quick rise and settling times), which is something we are trying to minimize. The PI control, on the other hand, is a good balance between a reasonable settling and rise times, and a reasonable control effort. The initial control effort is not as extreme when the derivative control coefficient is 0, and it drops down below 100 degrees celsius rather quickly. 
