# 3587 Robust Hive Detection

The upward-facing Logitech webcam should tell which hive cell is up, reliably, at any venue. That is hard because the hive's AprilTag clusters sit on the underside of each cell, facing down. The camera looks toward the ceiling lights at tags in shadow, and auto-exposure turns them dark or washed out.

The approach is a short exposure calibration we can run anywhere. Prove it first under several lighting setups in the lab, so we arrive at each venue confident that ten minutes of calibration will give dependable detection there too.

The match camera is `TurretCamera`, config name `hive_camera`. It already opens that camera at 640×480 through `VisionPortal`, and it already treats a cluster roll under 90 degrees as upright. The exposure check uses that same resolution. The SDK sample `ConceptAprilTagOptimizeExposure` shows how to turn auto-exposure off and set exposure, in milliseconds, and gain from code.

## At the lab: build the preset ladder

- Set up the camera through `VisionPortal` with auto-exposure turned off.
- Define five presets in code, labelled 1 (darkest) to 5 (brightest). Anchor preset 3 to what reads the tags best in the lab, and set 1 and 5 well beyond it in each direction. Prefer short exposure with higher gain. Blur from a moving robot breaks detection sooner than grainy noise does.
- Keep everything else fixed across the presets: tag size 3.25 in, the same resolution, and the same camera calibration.
- Add a small printed visor over the top of the lens so the ceiling lights stay out of frame. Tune with the visor on.

## The test program

A TeleOp named **Hive Exposure Check** runs with the robot parked at one shooting spot. It applies each preset in turn, discards the first half-second while the camera settles, then records about 60 frames and shows a ranked list on the Driver Station:

- **Detection rate.** Percent of frames that see the up-cell cluster. This is the main score.
- **Tags seen.** How many of the cluster's four tags were found. One of four is fragile.
- **Up-cell agreement.** Percent of frames where the cluster's roll (under 90 degrees means upright) matches the hive state we set.

Gamepad: **A** records the next spot, **B** marks that the hive was tipped by hand, **X** saves the winner. The winner is the preset with the best worst spot, not the best average. Pass bar: 95% detection and 95% agreement at every spot, in both hive states.

## Prove it in the lab first

Before any event, run the full calibration under at least four lighting setups. Each one has to end in robust detection.

| Setup | How to create it in the lab |
| --- | --- |
| Normal | Room lights as they usually are |
| Bright overhead | All lights on, plus a work light pointed down past the hive |
| Dim | Half the room lights off |
| Glare | A bright lamp behind and above the hive, in the camera's view |

For each setup, run Hive Exposure Check from scratch, save the winner, then drive a 2-minute practice session using it.

**Done when** every setup meets the pass bar, with the preset chosen for that setup, at every spot and in both hive states. If one setup cannot pass with any preset, fix the cause (visor, camera tilt, or a wider preset ladder) and rerun all four. Record which preset won each setup. If one preset wins most of them, it becomes the default.

## At each venue

1. At load-in, take the robot to the practice field and run Hive Exposure Check at every marked spot, in both hive states, plus one slow drive past the hive. About 10 minutes.
2. Save the winner. It is written to a small settings file on the Control Hub with the venue name, so no laptop or rebuild is needed. Every program loads it at INIT.
3. If nothing passes, deepen the visor or change the camera tilt, then rerun.
4. Log the detection rate during each match. If it drops on the competition field, rerun the check between matches.

Vision only confirms which cell is up. The robot tracks the hive itself: the starting state is known, each tip flips it, and a driver button overrides it. A bad lighting moment costs an aim check, not a shot at the wrong cell.
