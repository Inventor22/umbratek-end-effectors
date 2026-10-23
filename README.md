# umbratek-end-effectors

End effectors for the Umbratek UTRA 6-1000 robotic arm. The arm itself, its cart and the work cell are in
[umbratek-cad](https://github.com/Inventor22/umbratek-cad).

| End effector | What it is |
|---|---|
| [Chess Gripper](Chess%20Gripper/README.md) | Three claws that lift a chess piece by its neck. An SG90 servo pushes them open, and a rubber band closes them and holds the piece. SolidWorks parts, STLs and an animation. |
| [Pen Holder](Pen%20Holder/README.md) | Holds a pen on the flange axis. A coil spring lets the pen give up to 15 mm if the arm presses too far. |
| [Quick-Change Pen Holder](Quick-Change%20Pen%20Holder/README.md) | Holds any pen from 6 to 17 mm perpendicular to the flange, in cartridges that swap without tools, with 15 mm of spring travel. |
| [Pen Holder v2](Pen%20Holder%20v2/README.md) | The Pen Holder with its coil spring and sliding guides replaced by one printed PETG flexure. |
| [Quick-Change Pen Holder v2](Quick-Change%20Pen%20Holder%20v2/README.md) | The Quick-Change Pen Holder on a printed PETG flexure stage instead of rods and coil springs. |
| [Pen Holder v2 Leaf Spring](Pen%20Holder%20v2%20Leaf%20Spring/README.md) | The Pen Holder with its coil spring swapped for a stack of printed PETG bow springs. The sliding guides stay. |
| [Quick-Change Pen Holder v2 Leaf Spring](Quick-Change%20Pen%20Holder%20v2%20Leaf%20Spring/README.md) | The Quick-Change Pen Holder with its coil springs swapped for printed PETG bow springs on the same guide rods. |

Each folder holds one end effector: its README, the scripts that model and render it, its CAD exports in `STEP/`
and `STLs/`, and its renders in `Renders/`.

The pen holders are parametric [build123d](https://github.com/gumyr/build123d) models: run `uv run <model>.py` in a
folder to rebuild its exports. The v2 and quick-change holders import parts from the earlier holders' folders, so
keep the folders side by side.
