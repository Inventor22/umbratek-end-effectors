# AGENTS.md

How to work in this repo. Each end effector's README is the reference for its design; this file covers what they
share: layout, commands, design rules and the checks to run before you finish.

## What this repo is

- End effectors for the owner's Umbratek UTRA 6-1000 arm: a pen holder on the flange axis and a quick-change pen
  holder across it, each as v1 (steel springs), v2 (one printed PETG flexure) and v2 Leaf Spring (printed PETG bow
  springs), plus a chess gripper. The table in [README.md](README.md) lists them.
- The arm, its cart and the work cell are SolidWorks models in
  [Inventor22/umbratek-cad](https://github.com/Inventor22/umbratek-cad), whose root also holds the arm's
  `Quick_Start_Guide_UTRA_6-1000.pdf`, the source of the flange dimensions. This repo was split out of its
  `End Effectors/` folder, history included.
- The repo is public: commit no secrets, personal details or local paths.

## Layout

One top-level folder per end effector. A pen-holder folder holds its `README.md` (specs, how it works, BOM, printing,
assembly, robot setup with TCP and payload, tuning, regenerating), `<model>.py` (the parametric build123d model),
`render_<model>.py` (technical renders 1 to N with pyvista/VTK; render 0 is a Blender hero), `STEP/` (each part, plus
`<Prefix> - Assembly.step` with the arm flange as a reference), `STLs/` (in print orientation), `Renders/` and
`<Prefix> - Assembly.glb`. `Chess Gripper/` differs: SolidWorks `Parts/`, `STLs/` exported from them, and a Blender
animation (`animate_chess_gripper.py`, `make_video.py`; `frames/` is git-ignored).

Scripts import their neighbours with `sys.path.insert(0, str(HERE.parent / "<folder>"))`, so folder names, and keeping
the folders side by side, matter:

| Folder | Imports from |
|---|---|
| `Pen Holder` | nothing. It is the base: flange interface, tool adapter, hardware, pens, geometry helpers, and the shared renderer `render_pen_holder.py` |
| `Quick-Change Pen Holder` | `Pen Holder` |
| `Pen Holder v2` | `Pen Holder`, plus its own `flexure.py` |
| `Pen Holder v2 Leaf Spring` | `Pen Holder`, plus its own `bow_spring.py` |
| `Quick-Change Pen Holder v2` | `Pen Holder`, `Quick-Change Pen Holder`, `Pen Holder v2` (`flexure.py`) |
| `Quick-Change Pen Holder v2 Leaf Spring` | `Pen Holder`, `Quick-Change Pen Holder`, `Pen Holder v2 Leaf Spring` (`bow_spring.py`) |
| `Chess Gripper` | nothing |

Hero renders use `Pen Holder/blender_hero.py` or `Quick-Change Pen Holder/blender_hero_quick_change.py`. Editing a
shared file changes every folder that imports it.

## Commands (PowerShell)

The scripts declare their dependencies inline (PEP 723), so [uv](https://docs.astral.sh/uv/) needs no venv or install
step. Outputs land next to the script wherever you run it, but the READMEs' commands assume you are in the end
effector's folder:

```powershell
Set-Location "Pen Holder"                    # quote the spaces
$env:PYTHONIOENCODING = "utf-8"              # else characters like ³ and · in reports come out garbled when piped
uv run pen_holder.py --no-export             # design report only; exits with an error if a check FAILs
uv run pen_holder.py                         # report, then rewrites STEP/, STLs/ and the .glb
uv run render_pen_holder.py                  # every technical view into Renders/ (needs an OpenGL GPU)
uv run render_pen_holder.py 3 --out "$env:TEMP\renders"     # just view 3, to a scratch folder
uv run render_pen_holder.py --hero-glb "$env:TEMP\hero.glb"
& "$env:ProgramFiles\Blender Foundation\Blender 5.2\blender.exe" --background --factory-startup `
    --python blender_hero.py -- "$env:TEMP\hero.glb" "Renders\Pen Holder End Effector Render 0.png"
```

- Every model script takes `--out` and `--no-export`; every renderer takes view numbers, `--out` and `--hero-glb`.
  Each README's "Regenerating" section has its folder's exact commands, the chess animation's included.
- `blender` may not be on PATH, so call the default install as above. Keep `hero.glb` out of the repo: it isn't
  git-ignored.
- The first run downloads build123d, and about 80 MB of VTK for renders, into uv's cache. After that a model rebuilds
  in 15 s to 1 minute, a full set of renders takes about 1.5 minutes, a hero about 1 minute (GLB export, then Blender
  on the GPU) and the chess animation about 40 minutes, so allow long timeouts or run them in the background.
- Verified here on 2026-10-26 for `Pen Holder` (report, rebuild, view 3 and the Blender hero, to a scratch folder);
  the other folders' commands follow the same pattern but haven't been re-run since the move out of umbratek-cad.

## Design conventions

**Mounting.** The UTRA 6-1000 tool flange has 6x M5 on PCD 64, a Ø73 h7 pilot, a Ø3 H7 dowel on PCD 52 and a Ø47
tool-I/O cover in the middle ([full reference](Pen%20Holder/README.md#flange-reference)). Its dimensions are section 1
of `Pen Holder/pen_holder.py`: import them (`import pen_holder as ph`), don't retype them. A tool either bolts to
the v1 tool adapter's 3x M3 interface (captive nuts on PCD 43 at 30°, 150° and 270°), as the coaxial holders and
Quick-Change v2 do, or bolts straight to the flange's M5 holes, as the quick-change bases (with their own pilot recess,
dowel hole and I/O relief) and the chess gripper do. Include `ph.make_arm_flange()` in the assembly as a reference.

**Frames and units.** The coaxial models put the origin at the centre of the flange face, +Z into the arm (the tool
hangs along -Z) and +X towards the dowel. The quick-change models use a frame M, defined in their docstrings. READMEs
give the TCP and payload in the robot's flange frame, with Z out of the flange. Units are mm, N, MPa, g and degrees;
the Blender scripts work in metres.

**PETG printability.** Everything prints in PETG (0.2 mm layers, 4 perimeters, 40 % infill) without supports: overhangs
are at most 45° (chamfers, cones, teardrop roofs over horizontal holes) or short bridges. STLs are exported in print
orientation; `printed_parts()` returns each part in both its assembly and print poses. The models' checks use these
rules:

- PETG is about 45 MPa strong along the extrusion lines but only 25 MPa across layers, with E about 2 GPa ±20 %. Lay
  parts out so bending stress runs along the lines, and don't hang small pins, knobs or fingers off a layer line. Of
  the quick-change jaw pins the owner asked, "Are the track knobs on the jaws large enough to be sturdy if printed in
  PETG? … Don't want them to break off." They weren't (Ø3 mm, printed upright: twice the layer strength); Ø3.6 mm
  pins printed on their side with a teardrop fixed it. Give such features a root-stress check.
- Printed compliance means a flexure or bow springs, not a printed coil spring ("printing an actual spring isn't
  feasible, so it should be a sort of compliant 3D printable mechanism"). Reuse the folded double-parallelogram
  flexure in `Pen Holder v2/flexure.py` or the bow-spring stack in `Pen Holder v2 Leaf Spring/bow_spring.py`. Beams
  are at least 0.8 mm (two 0.4 mm perimeters); 0.5 mm leaves print as a single extrusion line (Arachne). Keep a
  safety factor of 1.8 at the hard stop and 8 MPa or less at rest, against creep. Print the mechanism relaxed, preload
  it at assembly and end its travel on a hard stop. Force goes with thickness cubed, so make thickness a parameter.
- Clearances: 0.4 mm on the diameter for sliding fits (`SLIDE_CLEARANCE`, `RING_CLEARANCE`), 0.3 mm for the pilot and
  the dovetail (`PILOT_CLEARANCE`, `DOVETAIL_CLEARANCE`), and 0.5 mm over the pen's diameter. Leave 1 to 1.5 mm round
  moving parts (`CLR`). Printed holes come out about 0.1 mm small. M3 holes are Ø3.3 for clearance and Ø2.5 for
  thread-forming; M5 clearance holes are Ø5.5.
- Walls in the checks: 1 to 1.5 mm for skins and slot walls, 2 mm round nut slots, 2.5 to 3 mm for bushings and
  housings. 0.7 to 0.8 mm appears only as a local minimum.
- Keep tools compact: the v2 holders stay narrower than the Ø86 flange. Of a first flexure 147 mm wide the owner
  asked, "must they be so wide? Any way to design them to be more compact?"

**Versions and names.** A new design generation is a new sibling folder that imports the one before ("Make these
version 2 of each variant"): `Pen Holder`, then `Pen Holder v2` and `Pen Holder v2 Leaf Spring`. Fixes to an existing
design happen in place. Exports are named `<Prefix> - <Part>.step` and `.stl`, with any print quantity in the name
(`Quick-Change - Jaw (print 6).stl`). Renders are `<Folder> Render N.png`, except the v1 Pen Holder's and the chess
gripper's `... End Effector Render N`. The chess gripper versions its parts inside its folder (`Gripper Claw v3`).

**Adding an end effector.**

1. Make a top-level folder, named in Title Case with spaces like the others.
2. Write `<name>.py` with the PEP 723 header and structure of `Pen Holder/pen_holder.py`: a docstring that defines the
   frame, UPPER_CASE parameters, `design_checks()`, `design_report()`, `assembly()`, `printed_parts()`, `export_all()`,
   and a `main()` with `--out` and `--no-export` that refuses to export if a check fails.
3. Write `render_<name>.py` on `render_pen_holder`'s `Scene`, `annotate`, `title` and `footer`, with a hero from one of
   the `blender_hero*.py` scripts.
4. Write `README.md` with the same sections as the other holders, and link it from related READMEs.
5. Add a row to the table in the root README.

## Before calling a change done

There is no CI for the models (the only workflow auto-merges Dependabot security fixes), so these checks are the gate:

1. `uv run <model>.py --no-export` shows every check `[ok]`. Fix the design rather than loosening a check, and add a
   check for any new way it could fail.
2. `uv run <model>.py` rewrites the exports. If you touched a shared file, rebuild every folder that imports it.
3. Render to a scratch folder with `--out`, look at downscaled copies, then render into `Renders/`, and redo the hero
   if the look changed. Show the owner the renders: design requests ask for them ("show me renderings").
4. Update the README's numbers (force, stress, TCP, payload, size, BOM) from the report.
5. `git status` lists only what you meant to change. With no design change a rebuild leaves the STLs and the .glb
   byte-identical but rewrites every STEP (new timestamps; the assembly also reorders its colours), so `git restore`
   the STEP files of parts whose STL didn't change. The committed renders predate the move and still carry an
   "umbratek-cad / End Effectors" footer, so every technical view you re-render will change.
6. Optionally, check that the STLs are watertight, one body each and sitting on z = 0, using trimesh through
   `uv run --with trimesh --with networkx --with scipy python -`.

A design task is finished when steps 1 to 5 are done. A slow render is a reason to run it in the background, not to
stop early.

## Context hygiene

- STEP, STL, GLB, PNG, JPG, GIF, MP4 and SolidWorks files are large or binary (assembly STEP and GLB files run to
  megabytes). Don't open, diff or edit them as text: regenerate them, and take numbers from the design report.
- To look at a render, save a copy no wider than about 1000 px in a scratch folder and view that.
- `*.SLDPRT` and `*.SLDASM` need SolidWorks, so changes to the chess gripper's geometry go through the owner.
- Put scratch output (test renders, `hero.glb`, animation frames) outside the repo.
- Other agent sessions may be working in other worktrees of this repo. Stay in yours.
