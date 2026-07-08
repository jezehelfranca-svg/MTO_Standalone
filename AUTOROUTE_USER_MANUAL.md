# Cable Autoroute — User Manual

**Applies to:** MTO_Standalone_App.html (Telecom Material Takeoff Tool)

The autoroute feature finds the shortest cable path between two tagged devices by following the tray/conduit corridors you draw on the drawing, then creates a cable takeoff with full length breakdown (horizontal run, vertical rises/drops, hook-up allowances) and containment fill tracking.

---

## 1. Concepts

| Term | Meaning |
|---|---|
| **Route Corridor** | A polyline you draw over the tray/conduit/duct run on the plan. Cables can only travel along corridors. |
| **Route Node / Exit Point** | A point marker on (or near) a corridor where cables enter/exit the containment — exit points, junctions, pull boxes, risers, penetrations, manholes. |
| **Endpoint (From/To Tag)** | Any placed takeoff item (device symbol) with a tag. The autoroute connects two of these. |
| **Hook-up** | The connection from a device to the corridor network. **Max Plan Hook-up** is the search radius (in real units) used to find a Route Node or corridor near the device. |
| **Route Class** | Corridors are classed Telecom / Power / Fiber / ELV / Shared. A cable can only use corridors compatible with its material (Telecom and ELV are interchangeable; Shared accepts everything). |
| **Layer / Elevation** | Corridors and nodes carry an elevation label (e.g. `EL +3.500`). Elevation differences add vertical length to the route and penalize the path cost. |
| **Capacity / Fill** | Corridors can carry tray/conduit dimensions and fill limits. Full corridors are skipped during routing. |

### How the router works (so the settings make sense)

1. All corridors on the current page that match the cable's route class (and aren't full) are turned into a graph. Corridor points closer than ~12 px are joined automatically.
2. Route Nodes attach to any corridor line within ~24 px of them.
3. Each endpoint device hooks to up to 4 Route Nodes within the **Max Plan Hook-up** radius. **If no Route Node is in range, the device hooks directly onto the nearest corridor line instead** (fallback — see §6).
4. Dijkstra shortest-path runs across the network. Nearly-full corridors cost extra, so the router prefers emptier trays.
5. A cable takeoff is created along the found path with all lengths and allowances computed.

---

## 2. One-Time Setup (do this before any routing)

### 2.1 Load the drawing
Load your PDF, image, or DXF layout.

### 2.2 Calibrate the scale — **the most important step**

The hook-up radius, all lengths, and coverage circles depend on `pixels per unit`. Two methods:

- **Reference line (recommended):** Click the calibrate/scale tool, draw a line over a known dimension (a dimensioned wall, gridline spacing, or the drawing's scale bar), and enter its real length. This is immune to PDFs that were plotted off-scale or cropped.
- **Scale ratio (1:xxx):** Enter the drawing's plot ratio. Only accurate if the PDF was really plotted at that ratio on the true paper size. For exported/cropped PDFs this is often wrong — prefer the reference line.

**Verify:** use the measure/dimension tool on another known dimension. If it reads wrong, recalibrate. *If calibration is wrong, every routing radius and every cable length will be wrong.*

> **Uncalibrated drawings:** autoroute still works — the hook-up radius falls back to a fixed 120 px — but all lengths are reported in pixels and are useless for takeoff. Always calibrate.

### 2.3 Place your devices
Place point takeoffs for the devices (cameras, APs, card readers, panels...). Give each a **Tag Name** in the properties panel — tags are how you pick routing endpoints. Optional per-device routing metadata (properties panel):

- **Hook-up Allow.** — extra cable length (m) added at this device's end (service loop / termination slack).
- **Cable Entry Elevation** — elevation where the cable enters the device (used for vertical length).
- **Termination Point Type / Entry Direction** — recorded into the cable schedule.

### 2.4 Define the cable material
Add a **Linear** material for the cable (e.g. "Cat6A UTP", "FOC 12C"). Note:

- The material name determines the inferred route class: names containing *fiber/FOC* → Fiber, *power/kV* → Power, *ELV/security/CCTV* → ELV, everything else → Telecom.
- Waste % and slack on the material are applied on top of the routed length.

---

## 3. Build the Route Network

All of this happens in the floating **Cable Routing Plan** panel (drag by its header; fold with the minimize button).

### 3.1 Set corridor properties *before* drawing

These are captured at the moment a corridor/node is created:

- **Layer / Elev.** — e.g. `EL +3.500`. The number in the label is parsed as the elevation.
- **Route Class** — who may use this corridor.
- **Route Capacity** (optional but recommended): Containment type (Tray/Conduit/Duct/Trench), Width/ID and Height (mm), Max Fill %, Spare %, Existing Fill %. Corridors with capacity configured will reject cables once full — and then you **must** enter a Cable OD when routing.

### 3.2 Draw corridors — `Draw Route Corridor` button

1. Click the button (or the corridor tool in the toolbar).
2. Click along the tray/conduit run on the drawing. **F8/Shift** toggles ortho for clean 90° runs; snap works on DXF drawings.
3. **Double-click or Enter** to save the corridor. **Backspace/Ctrl+Z** removes the last point, **Esc** cancels.

**Connectivity rules — this is where most routing failures come from:**

- Two corridors only connect if their points come within **12 px** of each other. Start a branch corridor *exactly on* the trunk line (click on it — at high zoom).
- Corridors on different elevations connect only where points coincide in plan and the elevation difference is ≤ 30 m — use a Route Node of type *Riser / Drop* at the transition and keep the plan position identical.
- Draw one continuous polyline per run rather than many tiny disconnected pieces.

### 3.3 Place Route Nodes — `Place Route Node / Exit Point` button

Select the **Node Type** (Exit Point, Junction, Pull Box, Riser / Drop, Penetration, Manhole / Handhole) and click on the drawing. Place them:

- **On or within ~24 px of a corridor line** — farther and the node won't attach to the network.
- Near each device or device cluster where cables leave the containment.
- At risers, assign a **Vertical Material** if the vertical run uses its own containment.

> Since the corridor-fallback update, route nodes are **optional** for simple hook-ups — devices can hook straight to the nearest corridor. Still place explicit nodes where you want to force a specific exit location, model a riser/penetration, or control which side of the tray the cable leaves.

---

## 4. Run the Autoroute

Work top-to-bottom in the lower half of the Cable Routing Plan panel:

| Field | What to enter |
|---|---|
| **Max Plan Hook-up** | Search radius (in drawing units, usually m) from a device to its Route Node / corridor. Set it to the realistic plan distance between devices and the containment — 10–20 m is a good start. Too small = "no Route Node / corridor within the hook-up allowance". |
| **From Tag / To Tag** | The two devices. The lists show all tagged takeoffs. Both must be on the same drawing page. |
| **From / To Hook-up Allow.** | Extra cable (m) at each end. Pre-filled from each device's metadata; override here per-route. Added to the final length, not used for searching. |
| **Vertical Drop/Rise Allow.** | Extra manual vertical allowance (m). If left blank, the devices' own drop/rise allowances are used. |
| **Cable Material** | The linear material for this cable. Determines route-class compatibility. |
| **Cable OD (mm)** | Outer diameter — required if any compatible corridor has capacity configured. Drives the fill calculation. |

Press **Auto Route Cable**. On success the status line turns green and reports the routed length and fill; a new cable takeoff appears on the drawing and in the takeoff table, selected and ready for review.

---

## 5. Reading the Results

The created takeoff's metadata (properties panel / cable schedule) contains:

- **Route Distance / Horizontal Length** — the plan length along the corridors.
- **Vertical Length** — computed from elevation changes along the path (corridor/node elevations and cable entry elevations).
- **Allowance Length** — from hook-up + to hook-up + vertical allowance, all added to the cable length.
- **Cable Length (takeoff)** — `(horizontal + vertical + allowances)` × `(1 + material waste %)` + material slack. Aerial materials additionally get catenary sag applied per span.
- **Route Notes / Remarks** — a full audit trail: corridor and node counts, which hook-up mode was used (explicit route nodes vs. nearest-corridor fallback), all allowances, and fill summary.
- **Fill Notes** — per-segment fill after adding this cable; the routing panel's network list also shows per-corridor utilization (warning ≥ warn threshold, red = full).

Routing is cumulative: every routed cable's cross-section is deducted from corridor capacity, so later cables avoid full trays automatically.

---

## 6. Troubleshooting

Status messages and what to do:

| Message | Cause | Fix |
|---|---|---|
| *Select both source and destination tags* | Missing From/To selection. | Pick both tags; give devices Tag Names if the lists are empty. |
| *Source and destination must be on the same drawing page* | Endpoints on different pages. | Route each page separately (use a riser node + continue on the other page as a second route). |
| *Select a linear cable material* | Chosen material isn't a path/linear type. | Create the cable as a Linear material. |
| *No compatible route corridors on page X for … cable* | No corridors on this page, or route class mismatch (e.g. Fiber cable but corridors are class Power). | Draw corridors, or set corridor class to match / Shared. |
| *All compatible corridors are full for this cable* | Fill check rejects every corridor. | Increase tray size / Max Fill %, reduce Existing Fill %, or add another corridor. |
| *Enter Cable OD (mm) before autorouting…* | A capacity-configured corridor exists but no OD given. | Enter the cable's outer diameter. |
| *X has no Route Node / corridor within the hook-up allowance. Nearest is … away; allowance is …* | The device is farther from the network than the search radius. The message tells you the actual gap in real units. | Increase **Max Plan Hook-up** past the reported distance, extend the corridor closer, or place a Route Node near the device. |
| *No connected route was found between the selected tags* | Both ends hooked, but the network between them is broken. | Zoom in on corridor junctions — branches must touch the trunk (≤ 12 px). Check that elevations of joining corridors match at the junction. Check whether a full corridor (noted in the message) was the only link. |
| *Route result was too short…* | Degenerate route (endpoints hooked to the same spot). | Check endpoint positions and rerun. |

### Large-scale drawings (1:2000 – 1:6000 site plans)

At extreme scales, one pixel can represent more than a meter. Three behaviors help here:

1. **Minimum hook-up radius:** the device search radius never drops below 40 px on screen, even if *Max Plan Hook-up* × scale would be smaller. On a 1:6000 plan, 40 px ≈ 40 m — the status/notes always report the effective allowance actually used.
2. **Corridor fallback:** if no Route Node is within range, devices hook directly to the nearest corridor line, so you don't need pixel-perfect node placement.
3. **Distance diagnostics:** hook-up failures report the real gap ("nearest is 213 m away, allowance is 43 m") so you immediately know whether to move something or raise the allowance.

Tips at these scales: draw corridors at maximum zoom, prefer few long polylines, and treat reported lengths as plan approximations — add generous hook-up allowances for the detail the drawing can't show.

---

## 7. Quick Reference

**Pre-flight checklist**

1. ☐ Drawing loaded, **scale calibrated and verified** with the measure tool
2. ☐ Devices placed with Tag Names (+ entry elevation / hook-up allowance if known)
3. ☐ Linear cable material created (waste %, slack set)
4. ☐ Layer/Elev., Route Class, capacity set **before** drawing each corridor
5. ☐ Corridors drawn, branches touching the trunk; Route Nodes at exits/risers
6. ☐ Max Plan Hook-up set to realistic device-to-tray distance
7. ☐ From/To tags picked, Cable OD entered → **Auto Route Cable**

**Keys while drawing corridors**

| Key | Action |
|---|---|
| Click | Add corridor point |
| Double-click / Enter / Space | Save corridor |
| Backspace / Ctrl+Z | Undo last point |
| Esc | Cancel corridor |
| F8 / hold Shift | Toggle ortho (90° runs) |

**Clearing:** individual corridors/nodes can be deleted from the network list in the panel or with the delete tool; *Clear Route Network* removes all of them (does not delete routed cables).
