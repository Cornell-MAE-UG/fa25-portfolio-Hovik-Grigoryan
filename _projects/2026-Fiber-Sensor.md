---
layout: project
title: Aerospace Adversary Lab Project - Fiber Optic Sensor
order: 1
description: Fiber Optic Based Laser Position Sensing System
technologies: [Fusion 360, MATLAB, 3D Printing]
image: /assets/images/fiber1.JPG
---

As an undergraduate researcher in Cornell's Aerospace Adversary Lab , I codesigned a fiber-optic sensor that localizes where a laser hits a curved surface, using a 4×4 fiber array paired with camera-based detection. This is an ongoing project aimed at building a low-cost, mechanically simple alternative to traditional position-sensing detectors for laser incidence tracking.

**System Overview**

The sensor works by discretizing a curved surface into 16 sampling points, each connected via fiber-optic cable to a camera-based readout grid. When a laser hits the curved surface, nearby fibers pick up light at intensities that depend on proximity to the exact point of incidence — this intensity pattern is what the software later uses to back out a precise location.

**Mechanical Design**

I was responsible for the mechanical side of the system, including:

- Designing the curved detection surface (1/8th of a sphere) in Fusion 360, sized to accommodate the 16-fiber array
- Designing and 3D-printing the fiber grid mount, which holds the fiber bundle in a precise 4×4 arrangement aligned with the camera's field of view
- Designing and printing the camera housing to keep the sensor rigidly positioned relative to the fiber grid
- Designing enclosures to shield the system from ambient light interference
- Iterating on the design across several print revisions to improve fiber alignment and reduce stray light leakage between adjacent channels

<div style="text-align:center;">
<img src="{{ 'assets/images/fiber2.JPG' | relative_url }}" alt="Fiber grid and camera mount" width="400">
</div>

**Signal Processing and Position Estimation**

On the software side, I developed a MATLAB script that:

- Reads intensity values from all 16 fibers via the camera
- Maps the discrete fiber-grid readings back onto the geometry of the spherical detection surface
- Applies an interpolation algorithm across the 16 intensity values to estimate laser position with resolution well beyond what the physical fiber spacing alone would allow
- Outputs a continuous 2D position estimate on the curved surface, rather than just identifying the nearest fiber

**Real-Time Visualization**

I also contributed to a MATLAB-based user interface, built around the CAD model of the sensor, that:

- Displays the estimated laser position in real time as the camera streams data
- Overlays the estimated hit location directly onto the 3D model of the curved surface, making results intuitive to interpret
- Allowed the team to validate the interpolation algorithm interactively during testing, rather than relying on post-processing recorded runs

<div style="text-align:center;">
<video controls muted style="max-width:450px; width:100%; border-radius:8px; box-shadow: var(--shadow); margin: 1.5em 0;">
  <source src="{{ '/assets/images/IMG_4773.mp4' | relative_url }}" type="video/mp4">
  Your browser doesn't support embedded videos.
</video>
</div>

**Portion of Code I Developed**

```matlab
clear; clc; close all;

%% --- USER SETTINGS ---
gridSize    = 4;        % 4x4 grid
threshold   = [];       % leave empty to auto-calibrate; or set 0-255 manually
intensityThresholdPct = 5;  % any intensity above this % counts as a "yes"/high signal

% Blob-detection tuning (used only during corner calibration)
diffSensitivity = 0.5;   % adaptive threshold sensitivity for the on/off diff image
minAreaPx       = 15;    % ignore diff blobs smaller than this (noise)

%% --- CONNECT TO CAMERA ---
camList = webcamlist;
if isempty(camList)
    error('No webcam detected. Check connection/drivers.');
end
cam = webcam(2);              % change index if you have multiple cameras
cam.Resolution = '1280x720';  % set to a resolution your camera supports

pause(3); % let camera warm up / auto-exposure settle

% Live preview so you can see the camera feed while calibrating
preview(cam);
disp('Live preview open -- watch it while you flip signals on/off during calibration.');

%% --- STEP 1: BASELINE (ALL OFF) ---
disp('Turn ALL 16 signals OFF, then press Enter to capture the baseline.');
pause;
frameOff = snapshot(cam);
grayOff  = rgb2gray(frameOff);

%% --- STEP 2: CORNER CALIBRATION (ONE CORNER AT A TIME) ---
cornerNames = {'Top-Left', 'Top-Right', 'Bottom-Right', 'Bottom-Left'};
corners     = zeros(4,2);
cornerRadii = zeros(4,1);
cornerHigh  = zeros(4,1);   % ON brightness at each corner (captured here)
cornerLow   = zeros(4,1);   % OFF brightness at each corner (from baseline)

for i = 1:4
    fprintf('Turn ON ONLY the %s corner signal (all others OFF), then press Enter.\n', ...
        cornerNames{i});
    pause;

    [pt, r, onVal] = detectSinglePoint(cam, grayOff, diffSensitivity, minAreaPx);
    corners(i,:)   = pt;
    cornerRadii(i) = r;
    cornerHigh(i)  = onVal;
    cornerLow(i)   = mean(samplePatch(grayOff, pt, r), 'all');

    fprintf('  -> %s detected at (%.0f, %.0f), radius %.1f px, ON=%.1f OFF=%.1f\n', ...
        cornerNames{i}, pt(1), pt(2), r, onVal, cornerLow(i));
end

TL = corners(1,:); TR = corners(2,:);
BR = corners(3,:); BL = corners(4,:);
patchRadius = max(3, round(mean(cornerRadii) * 0.7));

%% --- STEP 3: INTERPOLATE THE 16 GRID POINTS FROM THE 4 CORNERS ---
points = zeros(gridSize*gridSize, 2);
idx = 1;
for row = 0:gridSize-1
    v = row / (gridSize-1);           % 0 -> top, 1 -> bottom
    leftEdge  = TL + v*(BL - TL);
    rightEdge = TR + v*(BR - TR);
    for col = 0:gridSize-1
        u = col / (gridSize-1);       % 0 -> left, 1 -> right
        pt = leftEdge + u*(rightEdge - leftEdge);
        points(idx,:) = pt;
        idx = idx + 1;
    end
end

closePreview(cam);

% Sanity-check overlay
figure('Name','Calibrated Grid Points');
imshow(frameOff); hold on;
plot(corners(:,1), corners(:,2), 'ms', 'MarkerSize', 16, 'LineWidth', 2);
plot(points(:,1), points(:,2), 'c+', 'MarkerSize', 12, 'LineWidth', 2);
for i = 1:size(points,1)
    text(points(i,1)+8, points(i,2), num2str(i), 'Color','y','FontWeight','bold');
end
title('Magenta squares = detected corners, cyan = interpolated grid. Close to continue.');
disp('Detected corners and interpolated grid shown in figure. Close it to continue.');
waitfor(gcf);

%% --- STEP 4: THRESHOLD & INTENSITY SCALE (from the 4 corner samples) ---
if isempty(threshold)
    threshold = mean( (cornerLow + cornerHigh) / 2 );
    fprintf('Auto-calibrated threshold (from corners): %.1f\n', threshold);
end

% Intensity scale: corner ON brightness = 100% (maximum intensity),
% baseline OFF brightness = 0% (minimum intensity).
maxIntensityRaw = mean(cornerHigh);
minIntensityRaw = mean(cornerLow);
fprintf('Intensity scale: OFF (0%%) = %.1f raw, ON (100%%) = %.1f raw\n', ...
    minIntensityRaw, maxIntensityRaw);

%% --- MAIN LOOP ---
fig = figure('Name','Live Grid Detection');
disp('Running... close the figure window to stop.');

while ishandle(fig)
    frame = snapshot(cam);
    gray = rgb2gray(frame);

    vals = sampleAllPoints(gray, points, patchRadius);

    % Convert raw brightness to a 0-100% intensity using the scale
    % established from the corner calibration (OFF=0%, ON=100%).
    intensity = 100 * (vals - minIntensityRaw) / (maxIntensityRaw - minIntensityRaw);
    intensity = min(100, max(0, intensity));   % clamp to [0, 100]

    % Decision is based on intensity, not raw brightness: anything
    % above intensityThresholdPct counts as a "yes"/high signal.
    state = intensity > intensityThresholdPct;

    stateGrid     = reshape(state, gridSize, gridSize)';     % row-major 4x4
    valGrid       = reshape(vals, gridSize, gridSize)';
    intensityGrid = reshape(intensity, gridSize, gridSize)';

    % --- Display overlay ---
    imshow(frame); hold on;
    for i = 1:size(points,1)
        c = points(i,:);
        if state(i)
            plot(c(1), c(2), 'go', 'MarkerSize', 16, 'LineWidth', 2);
            text(c(1)+10, c(2), sprintf('%.0f%%', intensity(i)), ...
                'Color', 'g', 'FontWeight', 'bold', 'FontSize', 10);
        else
            plot(c(1), c(2), 'ro', 'MarkerSize', 16, 'LineWidth', 2);
        end
    end
    title(sprintf('Signal = intensity > %.0f%%  (green = yes, red = no; %% = intensity)', ...
        intensityThresholdPct));
    hold off;
    drawnow;

    % Print the 4x4 signal matrix and intensity matrix to the console
    disp('Signal (1 = yes, 0 = no):');
    disp(stateGrid);
    disp('Intensity (%, only meaningful where signal = 1):');
    disp(round(intensityGrid));
    fprintf('---\n');
end

clear cam;

%% --- HELPER FUNCTIONS ---

function patch = samplePatch(grayImg, pt, r)
    [h, w] = size(grayImg);
    cx = round(pt(1)); cy = round(pt(2));
    xr = max(1,cx-r):min(w,cx+r);
    yr = max(1,cy-r):min(h,cy+r);
    patch = grayImg(yr, xr);
end

function vals = sampleAllPoints(grayImg, points, r)
    n = size(points,1);
    vals = zeros(n,1);
    for i = 1:n
        patch = samplePatch(grayImg, points(i,:), r);
        vals(i) = mean(patch(:), 'all');
    end
end

function [pt, radius, onVal] = detectSinglePoint(cam, grayOff, sensitivity, minAreaPx)
% Grabs a frame, diffs it against the "all off" baseline, and returns
% the centroid/radius/ON-brightness of the single largest new bright
% blob (i.e. the one signal the user just turned on). Retries a
% couple of times with looser settings if nothing is found.

    attemptsSensitivity = [sensitivity, sensitivity*0.7, sensitivity*1.4];

    for attempt = 1:numel(attemptsSensitivity)
        frame = snapshot(cam);
        gray = rgb2gray(frame);

        diffImg = imsubtract(gray, grayOff);  % positive where it got brighter
        diffImg(diffImg < 0) = 0;

        bw = imbinarize(diffImg, 'adaptive', 'Sensitivity', attemptsSensitivity(attempt), ...
            'ForegroundPolarity', 'bright');
        bw = imopen(bw, strel('disk', 1));
        bw = bwareaopen(bw, minAreaPx);

        props = regionprops(bw, 'Centroid', 'Area', 'EquivDiameter');

        if ~isempty(props)
            [~, biggest] = max([props.Area]);
            pt = props(biggest).Centroid;
            radius = max(3, round(props(biggest).EquivDiameter / 2 * 0.7));
            onVal = mean(samplePatch(gray, pt, radius), 'all');
            return;
        end
    end

    error(['Could not detect a new lit point. Make sure exactly one signal is ON ' ...
        'and it is visibly brighter than the dark baseline, then try again.']);
end
end
```