# Parametric Sensor Mount in OpenSCAD with Claude Code: A Home Assistant 3D Print

> Have Claude Code write a parametric OpenSCAD wall mount for a Zigbee sensor, then check it for real: manifold test, fit test and preview. Includes the full file.

Source: https://exitcode0.net/posts/openscad-claude-code-parametric-sensor-mount/
Author: Tom Cocking (https://tomcocking.com)
Published: 2026-10-27
Updated: 2026-10-27
Tags: openscad, 3d-printing, claude-code, home-assistant, zigbee, sensors


Small Zigbee and Bluetooth sensors tend to ship with a sticky pad and nothing else. The pad fails the first time you move house or the wall is cold and damp, and the sensor ends up behind the radiator. A printed bracket fixes that, but I'd rather not spend an evening in CAD for a plastic box.

OpenSCAD is the opposite of CAD. The model is a text file, so Claude Code can write it, render it and check it in a loop. This post has the file I ended up with, the checks I ran, and the one that caught a mistake the obvious check missed.

My [Fire HD8 dashboard post](https://exitcode0.net/posts/home-assistant-firehd8-dashboard/) covers printed wall mounts for a tablet. This is the same idea at sensor size. The [mini graph card post](https://exitcode0.net/posts/mini-graph-card-temperature-humidity-home-assistant/) shows where the readings end up.

## What the mount is

A U-shaped bracket. The back plate takes two countersunk screws. The sensor slides in from the top and rests on the bottom wall. A 2.5 mm lip down the left, right and bottom edges keeps it from falling forward, and the window in the front leaves its face visible and its sensing side open to the room.

## The OpenSCAD file

Save this as `sensor_mount.scad`. Measure your sensor with calipers and change the first four values. The rest follow from them.

```openscad
// Wall mount for a small square Zigbee sensor. All sizes in mm.
// Measure your sensor with calipers and change the first four values.
sensor_w = 40;      // sensor width (left to right)
sensor_h = 40;      // sensor height (top to bottom)
sensor_d = 14;      // sensor thickness, front to back
clearance = 0.4;    // printer tolerance, per side
wall = 2.4;         // side and back wall, six perimeters at a 0.4 mm nozzle
front = 1.6;        // front lip thickness
lip = 2.5;          // how far the lip overlaps the sensor face
screw_d = 4.0;      // screw shaft
head_d = 8.0;       // screw head, for the countersink
$fn = 48;

inner_w = sensor_w + 2 * clearance;
inner_h = sensor_h + 2 * clearance;
outer_w = inner_w + 2 * wall;
outer_h = inner_h + wall;          // bottom wall only; the top is open
depth = wall + sensor_d + front;

module screw_hole(x, y) {
  translate([x, y, -0.01]) cylinder(d = screw_d, h = wall + 0.02);
  translate([x, y, wall - (head_d - screw_d) / 2])
    cylinder(d1 = screw_d, d2 = head_d, h = (head_d - screw_d) / 2 + 0.01);
}

difference() {
  cube([outer_w, outer_h, depth]);
  // sensor pocket, open at the top so the sensor slides in
  translate([wall, wall, wall]) cube([inner_w, inner_h + 1, sensor_d]);
  // front window, so the sensor reads room air and you can see its face
  translate([wall + lip, wall + lip, wall + sensor_d - 0.01])
    cube([inner_w - 2 * lip, inner_h - lip + 1, front + 0.02]);
  // two screw holes through the back plate, reachable through the window
  screw_hole(wall + 9, wall + inner_h / 2);
  screw_hole(outer_w - wall - 9, wall + inner_h / 2);
}
```

The defaults are a 40 x 40 x 14 mm block. That's a placeholder, not any specific sensor. Measure yours.

## Render it

```bash
openscad -o sensor_mount.stl sensor_mount.scad
```

I ran that inside a throwaway Debian 12 container, which gives OpenSCAD 2021.01. It took half a second. The preview below came from the same file, with the front on the left and the back on the right:

![Two renders of the sensor mount. The left shows the pocket, the front lip and two countersunk screw holes. The right shows the flat back plate with the screw holes](https://exitcode0.net/images/openscad-ha-sensor-mount-preview.jpg)

## Checking the model

This is the part worth giving to the agent. Three checks, from least to most revealing.

### 1. Is the mesh printable?

OpenSCAD prints `Simple: yes` when the mesh is a valid closed solid, and I confirmed it with the `trimesh` Python package:

```python
import trimesh
m = trimesh.load("sensor_mount.stl")
print(m.is_watertight, m.volume / 1000, m.extents)
```

That printed `True`, 9.83 cm³ and 45.6 x 43.2 x 18.0 mm, which matches what the parameters add up to. At roughly 1.24 g/cm³ for PLA that's about 12 g if it were solid, and less with normal infill.

### 2. Does the sensor fit?

A valid mesh doesn't mean a useful one. Intersect the mount with a block the size of the sensor, placed where the sensor would sit. If anything overlaps, the pocket is too tight:

```openscad
intersection() {
  mount();
  translate([2.4 + 0.4, 2.4 + 0.4, 2.4]) cube([40, 40, 14]);
}
```

`mount()` is the main `difference()` wrapped in a module. The result was empty, so the block seats cleanly with 0.4 mm to spare per side.

### 3. Look at it

My first version passed check 1. It also had no front lip at all, because the pocket cut ran straight out through the front face. A watertight mesh with no way to hold the sensor in is still a watertight mesh.

The fit test said nothing wrong either, since an open pocket has no overlap. Only the render showed it. The fix was to stop the pocket at the sensor's thickness and cut a smaller window through the front plate.

If you ask Claude Code to do this, tell it to render a PNG and look at it. In my container, `xvfb-run openscad -o front.png --camera=... --imgsize=900,800 sensor_mount.scad` worked with the `xvfb` and `libgl1-mesa-dri` packages installed.

## Not printed yet

I haven't printed this. The numbers above come from the model, not from a sensor in the bracket. The lip is a 2.5 mm overhang over an empty pocket, which most printers bridge, but I haven't confirmed that on mine. Print the back plate down and see.

If the sensor is loose or tight, change `clearance` by 0.1 and reprint. That's the whole reason for making it parametric.

## Giving it to your agent

A prompt along these lines works. The useful parts are the measurements and the instruction to render and look:

```text
Write an OpenSCAD wall mount for a 40 x 40 x 14 mm sensor. Parameters at the top. Back plate with two countersunk M4 screws, open top so the sensor slides in, a front lip that holds it in, and a window so it still reads the room. Render to STL and check it is watertight. Then run an intersection test with a sensor-sized block. Then render a PNG of the front and describe what you see before you tell me it's done.
```

## Wrap-up

The model took more thought than the file suggests. The checks matter more than the code, and the one that caught the real mistake was the least automated. I'd trust the agent to write the SCAD. I wouldn't trust it to tell me the model works without a picture.

## Or just hand this page to your agent

If you'd rather skip the copying, hand the page over:

```text
Read https://exitcode0.net/posts/openscad-claude-code-parametric-sensor-mount/ and make me an OpenSCAD wall mount for my sensor. Before you write anything, ask me for the sensor's width, height and thickness, or measure them from a photo I give you. Don't guess the dimensions. Render the STL, run the watertight check and the intersection test from the page, then render a PNG and describe it.
```

The page gives the agent the design, the checks and the mistake to avoid. Only you have the sensor, the calipers and the printer.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/openscad-claude-code-parametric-sensor-mount/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
