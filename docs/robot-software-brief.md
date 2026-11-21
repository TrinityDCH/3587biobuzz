# 3587 Robot Software Brief (BIOBUZZ 2026-27)

21 Nov 2026. Hardware is the team's walkthrough from that day. Code status is `master` at `781ce24`, the FTC SDK 12.0.0 app in this repo (Android Studio Narwhal 3 Feature Drop or later).

This file is the plan for the code we are about to write. Where the walkthrough and the skeleton disagree, follow the walkthrough, and change the skeleton before filling in the empty classes.

## Summary

The robot has eight motors and seven servos, all identified, so each subsystem can be named and constructed. A mecanum drive and a goBILDA Pinpoint move and locate the robot. One intake and one transfer feed two shooters in series, nectar in front and pollen behind. Ramp legs sort the two elements with no actuator. Two REV Color Sensor V3 units sit on the transfer and report presence. A Logitech webcam faces up at the hive. A Limelight faces down at the balls.

`master` already runs three things: a robot-centric mecanum drive, a Pinpoint wrapper, and a hive AprilTag reader. Everything else is an empty class. The empty classes were sketched for a different machine: one flywheel, one hood, one turret, a backspin motor, a traction motor, and an intake-deploy servo. That sketch does not fit this robot. The counter roller on each launcher is geared to the flywheel, so there is no backspin motor. Eight motors fill the legal budget, so there is no traction motor either. The seventh servo is the flower arm.

Three hardware facts are still open, and they block the counting code and the turret code: where the color sensors sit relative to the ramp legs, the turret range and homing method, and whether the two turret sweeps overlap. The turret question has a 10-minute test below.

Build in this order: make the config names match the robot, fix the drive loop that is already running, measure the Pinpoint, then intake and transfer, one shooter by hand, turrets, the second shooter, then shot presets.

## Labels

- **Confirmed.** The team said it on 21 Nov 2026.
- **Inferred.** Our reading. Still to check on the robot.
- **Verified.** A vendor page or the rules, opened 21 Nov 2026. The sources are at the bottom.
- **In the repo.** Read from `master` on 21 Nov 2026.

## How the code is organized

Team code lives in `TeamCode/src/main/java/org/firstinspires/ftc/teamcode`. The pattern already in use:

- `Subsystem` requires `periodic()` and `writeTelemetry(Telemetry)`. `periodic()` runs every loop and returns quickly.
- `Robot` constructs the subsystems, calls `periodic()` on them, and writes their telemetry. A test OpMode that needs one device should construct that device itself. `new Robot(...)` pulls in the drive and the Pinpoint, and both use `hardwareMap.get`, so a missing motor or a missing Pinpoint throws during init.
- `HardwareNames` is the only place a config string should appear. `MecanumDrive` and `Pinpoint` already do this. `DriveOpMode` still has three raw `"bl"`, `"fr"`, and `"br"` literals.
- Constants classes sit next to the subsystem. They are empty.
- Match behavior goes through `opmodes/teleop/MainTeleOp`. Per-device checks go through `opmodes/test`. Autonomous is `opmodes/auto`, and it is empty.

`Robot.periodic()` calls `pinpoint`, `drive`, and `turretcam` by hand. The `subsystems` list is used only for telemetry. A new subsystem has to be added in both places.

The driver station currently shows three team OpModes, all enabled:

| File | Driver station name | What it does |
| --- | --- | --- |
| `DriveOpMode` (teamcode root) | DriveOpMode | A second copy of robot-centric drive. Does not use `Robot`. `@Disabled` is commented out. |
| `opmodes/teleop/MainTeleOp` | TestDriveOpMode | The real teleop loop. Drive, slow mode, pose reset, hive camera. |
| `opmodes/test/HiveAimTest` | HiveAimTest | Builds a full `Robot` and locks the alliance to red. |

`IntakeTest`, `ShooterTest`, `TurretZeroTest`, and `BallFollowTest` are empty files with no `@TeleOp`, so they do not appear on the driver station. `Auto_Preload_Tip_Park` is an empty class. That name is a leftover; leave autonomous until teleop can score.

The SDK ships a Utility OpMode, "Test Hardware" (`UtilityTestHardware`). It is in the Utility menu, steps through the active config one device at a time, jogs a motor or servo while the right bumper is held, and reads encoder counts, color, distance, and analog voltage. Use it to prove a config name before writing a team OpMode. It will not log a time series, and it will not run a shot sequence.

## Hardware inventory and port budget

The robot uses all 8 motors and 7 of the 8 servos that R503 allows. No spare motor. One spare servo.

| Group | Device | Qty | How software drives it | Status |
| --- | --- | --- | --- | --- |
| Motor | Drive | 4 | Open loop, no encoder feedback | Confirmed count. Mecanum is in the repo. |
| Motor | Intake | 1 | On, off, reverse | Confirmed |
| Motor | Transfer | 1 | On, off, reverse | Confirmed |
| Motor | Pollen launcher | 1 | Velocity from its encoder | Confirmed. Rear shooter. |
| Motor | Nectar launcher | 1 | Velocity from its encoder | Confirmed. Front shooter. |
| Servo | Hood | 2 | Position, one per shooter | Confirmed |
| Servo | Gate (ball stopper) | 2 | Position, one per shooter | Confirmed |
| Servo | Turret | 2 | Position. Analog feedback available | Inferred |
| Servo | Flower arm | 1 | Stow, hover, down | Confirmed. Wedge and wheels unfinished. |
| Sensor | Pinpoint | 1 | I2C, SDK `GoBildaPinpointDriver`, config `pin` | Confirmed device. Offsets in the repo are the sample values. |
| Sensor | REV Color Sensor V3 | 2 | I2C, address 0x52 each, one port each | Confirmed. Address Verified. |
| Camera | Logitech webcam | 1 | USB webcam, config `hive_camera`, faces up, reads the hive | Confirmed. Class `TurretCamera` is running. |
| Camera | Limelight | 1 | Faces down, finds balls, config `limelight` | Confirmed device. `BallCamera` is empty. |

- Each launcher drives its counter roller through gears. No separate motor and no backspin code. Confirmed.
- The intake and transfer encoder cables are not plugged in yet. Confirmed.
- Eight motors fill both hubs, four each, on one Control Hub and one Expansion Hub. Inferred.
- All seven servos are Axon MAX units. Confirmed. The turrets were described as servos earlier on 21 Nov and left off the later servo list, so the count of seven assumes both turrets are servos.

`HardwareNames` today still has the old sketch: `traction`, `intake_deploy`, `intake_color`, `shooter_color`, `flywheel`, `hood`, `backspin`, `turret`. Those singular names cannot represent two shooters. `traction` plus `backspin` would also be a ninth and tenth motor, which R503 does not allow. This plan retires them.

## Config names to write into HardwareNames

Short lowercase names, matching `fl` and `pin`. One class, two instances, for the shooter, the hood, the gate, and the turret. Front is nectar. Rear is pollen. The ramp legs make that assignment mechanical.

| Device | Target config name | In `HardwareNames.java` today |
| --- | --- | --- |
| Drive, four motors | `fl` `bl` `fr` `br` | Same |
| Pinpoint | `pin` | Same |
| Intake motor | `intake` | Same |
| Transfer motor | `transfer` | Missing. `traction` is the extra motor name. |
| Nectar flywheel (front) | `nectar_flywheel` | `flywheel` (one name only) |
| Pollen flywheel (rear) | `pollen_flywheel` | Missing |
| Nectar hood | `nectar_hood` | `hood` (one name only) |
| Pollen hood | `pollen_hood` | Missing |
| Nectar gate | `nectar_gate` | Missing |
| Pollen gate | `pollen_gate` | Missing |
| Nectar turret | `nectar_turret` | `turret` (one name only) |
| Pollen turret | `pollen_turret` | Missing |
| Flower arm | `flower` | `intake_deploy` |
| Hive webcam | `hive_camera` | Same. The Pedro branch still says `ball_camera`. |
| Limelight | `limelight` | Same |
| Two color sensors | Leave unset | `intake_color`, `shooter_color` |

Leave the color-sensor names unset until someone looks at the robot (open question 1). `intake_color` and `shooter_color` already pick a placement the walkthrough did not confirm. If both sensors are on the shared belt, name them `transfer_near` and `transfer_far` from the intake. If each sits on a lane after the legs, name them `nectar_color` and `pollen_color`.

Retire, and do not write a subsystem for: `traction`, `backspin`, `Baskspin` (the filename is misspelled; `BackspinConstants` is the other half), and `intake_deploy`. `intake_deploy` is the old name for the flower-arm servo. If a look at the robot finds a device that still uses one of those names, the inventory above is wrong and this section stops.

Comments in `MecanumDrive` mark the drive motors as hub ports 0, 1, 2, and 3 (`fl` 0, `fr` 1, `br` 2, `bl` 3). Those are comments. The wiring map is still open question 5.

## Classes

Keep the empty files that match a real device. Add the three that the walkthrough has and the skeleton does not.

| Class | Package | Instances | Repo today |
| --- | --- | --- | --- |
| `MecanumDrive` | `subsystems.drive` | 1 | Running. Robot-centric. |
| `Pinpoint` | `subsystems.drive` | 1 | Running. Sample offsets. |
| `Intake` | `subsystems.intake` | 1 | Empty |
| `FlowerArm` | `subsystems.intake` | 1 | Does not exist. Add when the wedge exists. |
| `Transfer` | `subsystems.transfer` | 1 | Does not exist. Owns the belt and, once placed, the two color sensors. |
| `Shooter` | `subsystems.shooter` | 2 | Empty. This class is the flywheel: `setVelocity`, ready when held inside tolerance. |
| `Hood` | `subsystems.shooter` | 2 | Empty |
| `Gate` | `subsystems.shooter` | 2 | Does not exist. Two positions and one open pulse. |
| `Shot` | `subsystems.shooter` | 2 | Does not exist. The state machine. Owns one `Shooter`, one `Hood`, one `Gate`, one `Turret`. |
| `Turret` | `subsystems.turret` | 2 | Empty |
| `TurretCamera` | `subsystems.vision` | 1 | Running. This is the upward hive webcam, config `hive_camera`. |
| `BallCamera` | `subsystems.vision` | 1 | Empty. The Limelight. |
| `MatchState` | `common` | 1 | Empty. Match clock and G410. It is not the shot state machine. |

`common.Alliance` and `common.HiveCell` are unused string holders. The hive reader uses its own enums, `TurretCamera.Alliance` and `TurretCamera.Cell`, and those match the tag names (`RED SCORING`, `RED AUDIENCE`, `BLUE SCORING`, `BLUE AUDIENCE`). Keep the camera enums. `HiveCell` cannot say which alliance is up.

`DriveConstants` is the home for the measured pod offsets, encoder directions, and the slow-mode scale once they are real. `Pinpoint.java` currently inlines the sample offsets.

## How balls move

The ramp legs split the balls, so no code chooses a shooter for a ball.

Balls enter at the intake and ride the transfer past the two color sensors. At the ramp legs, nectar rolls up to the front shooter and pollen passes between the legs to the rear shooter. The flower arm's wedge frees balls at a flower base so the intake can take them.

One transfer motor feeds both shooters. The gates are the only per-shooter feed control. A closed gate holds its lane while the transfer keeps running.

## Subsystems

Only the two flywheels are closed-loop. Everything else is open loop, a servo position, or a sensor read.

### Drive

Plan: field-centric mecanum. Rotate axial and lateral by the Pinpoint heading inside `MecanumDrive.setDriverInput`'s math, then mix the wheels. A driver button resets the pose to a marked field corner, on a press, once.

In the repo: the mix is robot-centric POV. `MainTeleOp` passes `axial = -left_stick_y`, `lateral = left_stick_x`, `yaw = right_stick_x`. Left motors are `REVERSE`, right motors are `FORWARD`. Gamepad A toggles slow mode, which halves wheel power. Gamepad B is level-triggered and writes pose `(0, 0, 0)` on every loop while it is held. The `Pinpoint` argument on the `MecanumDrive` constructor is unused. No code sets `RUN_WITHOUT_ENCODER`; that is already the SDK default, which is what open-loop drive wants. No code sets zero-power behavior, so the default is coast. No code turns on bulk reads.

`MecanumDrive.writeTelemetry` calls `telemetry.update()` in the middle of `Robot.writeTelemetry`. The driver station packet splits, and the Pinpoint and camera lines land late or in the next packet. Remove that `update()`. `MainTeleOp` should be the only caller of `telemetry.update()`.

Watch for: confirm directions on the real drivetrain before changing the reversal constants. After heading is trustworthy, add the field-centric rotation. Set `ZeroPowerBehavior.BRAKE` if the drivers want the robot to stop when the sticks return. Turn on manual bulk caching once, in `Robot`, and `clearBulkCache()` at the top of `Robot.periodic()`.

### Intake

Plan: on, off, reverse to clear a jam. `RUN_WITHOUT_ENCODER`. The intake stops early enough that balls already between the mouth and the first sensor still fit under the 4-ball limit.

In the repo: `Intake.java` is empty. `IntakeTest` has no `@TeleOp`.

### Transfer

Plan: on, off, reverse. Balls press against a closed gate, so a stall has to back the belt off. With the encoder cable plugged in and the mode left at `RUN_WITHOUT_ENCODER`, `getCurrentPosition()` should still advance. A one-minute bench test confirms that before any counting code trusts it. Motor current is the fallback if the cable stays off.

In the repo: no transfer class. Do not put this motor in `Intake`.

### Launchers

Plan: `setVelocity` in ticks per second. Ready means inside a tolerance for a hold time. Each ball drops the RPM, and the sequence waits for it to recover. Needs ticks per revolution of the installed motor (open question 4). Pollen is the first shooter brought up, because the field has 40 pollen and 16 nectar.

In the repo: `Shooter.java` is empty. There is no velocity loop and no motor type.

### Gates

Plan: closed and open. One open pulse feeds one ball, and only while that launcher is ready. Calibrate the two positions per servo.

In the repo: no gate class.

### Hoods

Plan: a position looked up from the marked spot. Separate tables for nectar (about 91 mm) and pollen (about 71 mm). Redo a spot's table after any hood change.

In the repo: `Hood.java` and `HoodConstants.java` are empty.

### Turrets

Plan: turret angle = bearing to the selected goal, minus robot heading, plus the turret's zero offset, clamped to the measured range. A keep-out table stops the two turrets colliding or blocking each other's shot. R105 still needs a mechanical limit; a software clamp is not enough.

In the repo: `Turret.java` is empty. `TurretZeroTest` is the file for the range test below, and it has no `@TeleOp` yet.

The hive camera also publishes a bearing. That bearing is in the camera frame. Use it to trim the Pinpoint aim, or to confirm which cell is up, after adding the camera's yaw relative to the robot. If the camera turns out to be bolted to a turret, that trim is the wrong model and this sentence changes.

### Flower arm

Plan: three positions, stow, hover, and down. The intake still has to stop at 4 balls while the wedge is freeing balls. Check the stowed arm against the 18 inch starting cube.

In the repo: nothing. Add `FlowerArm` when the wedge and wheels exist. Until then, do not block the shooter work on it.

## Shot sequence

One `Shot` instance per shooter. Not written yet. `MatchState` stays the match-level object (alliance, and the G410 clock).

1. **IDLE.** Flywheel off or at a low idle speed. Gate closed.
2. **SPINUP.** Flywheel to the RPM from the spot preset. Hood set.
3. **AIM.** Turret to the angle from the Pinpoint pose toward the selected goal.
4. **READY.** RPM inside tolerance for the hold time, and the turret settled.
5. **FEED.** Gate opens for one pulse. One ball leaves.
6. **RECOVER.** Wait until RPM is back inside tolerance. Return to READY if balls remain, otherwise IDLE.

RECOVER is what lets 4 balls in 3 seconds beat 2 or 3 in 1 second. Spin the two flywheels up one after the other, pollen first, so the battery sags once. That order is untested.

Targeting. The hive is bi-stable, so there are two goal poses, and a driver button picks the up cell. Presets keyed to marked spots come first. 16 hits in 20 shots at a spot is the gate to add the next spot. Distance interpolation comes after the presets hold.

Who fires (one button per shooter, one driver or two, one ball or a volley) only changes the trigger into `Shot`. It does not change the six states. Open question 6.

## Sensors and ball counting

The two color sensors tell software that a ball is present at two points on the transfer. They do not, in the first version, say which element it is or how many balls are aboard. No sensor code exists yet.

- **Placement (Inferred, about 70%, range 55 to 85%).** Both sensors sit on one side of the transfer, ahead of the ramp legs, so every ball passes both. If they sit after the legs, each sees one lane and a single shared count for G407 will be wrong.
- **Wiring (Verified).** The Color Sensor V3 is fixed at I2C address 0x52. Two devices with that address need two buses. REV sells no cable that puts both on one port. The original Pinpoint is 0x31 at up to 400 kHz, so it takes a third port. The Pinpoint V2 page does not state an address. The repo uses `GoBildaPinpointDriver` and never sets an address. Check which unit is installed.
- **Proximity before color (Verified range).** The V3 reports proximity from 1 cm to 10 cm. Holes in a ball break color readings, and pollen is perforated. Threshold proximity at the real mount, and debounce for a few tens of milliseconds. "Test Hardware" can show that a sensor is alive. The 1, 2, 3, and 4 ball log is a team OpMode, because the utility does not record proximity against time.
- **Direction.** Sensor A then B is forward. B then A is reverse.
- **Counting (Inferred, about 70%, range 55 to 85%).** Edge counting on balls that touch will probably fail. The sensor reads present the whole time a ball is in front of it, and holes add false dips. Log proximity first, then choose the method.
- **Belt position (Inferred).** Plug the encoder cables into the intake and the transfer. Leave `RUN_WITHOUT_ENCODER`. Confirm with the bench test that position still counts, then use belt travel to place balls between the sensors and to flag a stall.
- **Count latency.** Balls between the mouth and the first sensor are not counted yet. Measure how many fit in that gap, and stop the intake that many balls early.
- **Color, after the qualifier (Inferred).** Yellow pollen against red or blue nectar is a large difference, and a moving ball gives many samples, so a majority vote can beat the holes. That is also the path to refusing opponent nectar (G408), by reversing the transfer.

## Servos and turrets

All seven servos are Axon MAX units. Both the MAX+ and the MAX MK2 are on the 2026-27 pre-verified list, so legality is settled. Range setup is the open work. No servo is constructed in team code yet.

- **Legality (Verified).** Inspection reference 26-27.1 limits servos to 8 W maximum mechanical power and 4 A stall at 6 V, and pre-verifies both Axon models at 7.59 W and 4 A. R503 caps the robot at 8 motors and 8 servos.
- **MK2 specs (Verified).** 350° maximum, 0.175° per µs, PWM 500 to 2500 µs, 4.8 to 8.4 V, 0.12 s per 60°, 4 A stall at 6 V. A fourth wire is analog position feedback. The page does not state its voltage range.
- **Range in the hub (Verified; the degrees are our arithmetic).** Hubs default to 600–2400 µs and can expand to 500–2500 µs. The Full Range Servo configuration uses the full span. At 0.175° per µs that is 315° on the default and 350° expanded. Configure every Axon as a Full Range Servo.

### Turret range and homing, about 10 minutes

Do this in `TurretZeroTest`, one turret at a time. "Test Hardware" can read the analog voltage (`readRawVoltage`) before that OpMode exists. Stall current is 4 A, so stop short of a hard stop.

1. Find the gear ratio between the servo and the turret. Turret range is the servo range divided by the ratio.
2. Step the servo position from 0 to 1 slowly. Record the angles where the turret hits a stop or pulls a cable.
3. Plug the feedback wire into a hub analog port. Two turrets need two free analog ports. Read the voltage at each step. That map is absolute position, and it removes the need for a homing sweep.
4. Check whether the voltage wraps inside the working range. Analog feedback on this servo family often wraps once per revolution. That wrap is unverified. Keep the working range off the wrap point.
5. At init, read the feedback first and command that same position, so the turret does not snap to the hub's default.
6. Write the range, ratio, offset, and home angle into `TurretConstants`, and copy the numbers into the table at the bottom of this doc.

If a feedback wire is not connected, home against a hard stop or a limit switch instead.

After both turrets have a range, rotate them by hand together and write the keep-out pairs. Software clamps the combined angles. The mechanical stops still have to satisfy R105 on their own.

## Vision

Two cameras, two jobs. The code has started the hive job and has not started the ball job.

### Hive, `TurretCamera`

In the repo, and running:

- Opens `hive_camera` with `hardwareMap.tryGet(WebcamName)`. A missing camera disables vision and leaves the rest of the OpMode up. The drive and the Pinpoint do not fail soft, so `HiveAimTest` still cannot init unless `fl`, `bl`, `fr`, `br`, and `pin` are in the config. A hive-only test should construct `TurretCamera` directly.
- `VisionPortal` at 640×480, `AprilTagProcessor`, `AprilTagGameDatabase.getBioBuzzTagLibrary()`.
- Cluster detections only. A singleton tag is ignored. This matches the SDK 12 break: pre-September 2026 AprilTag samples do not compile against this tree.
- A cluster is rejected when its frame is older than 200 ms, the short name is unknown, the alliance does not match, the pose is missing or non-finite, the range is not positive, or `|roll| >= 90°`. Roll under 90° is the upright, scorable cell.
- The same cell has to agree for 3 distinct frames before `hasUpwardTarget()` is true. Two different cells passing in one read clears the target as ambiguous.
- Published while confirmed: `getTargetCell()`, `getBearingDegrees()`, `getRangeInches()`. `setAlliance` clears the confirmation.
- `HiveAimTest` calls `setAlliance(RED)` and never changes it. There is no driver control and no use of `common.Alliance`.
- Exposure and gain are left on auto. 3587 has had trouble with hive-tag lighting. Lock both through the portal's `ExposureControl` and `GainControl` once the camera is framed.
- `writeTelemetry` repeats the rejection lines once per fresh detection. The commit message on `80716b0` already notes this. Clean it up when the telemetry `update()` bug is fixed.
- `Robot.stop()` closes the portal. `MainTeleOp` and `HiveAimTest` never call it.

The webcam stays the confirmation path. The driver button that picks the up cell stays the fallback until the camera holds a cell through a real match. BIOBUZZ tags move, so they are not a field localizer. The comment in `MainTeleOp` about seeding the Pinpoint from tags is for a known reset pose, not for tracking the robot off the hive.

### Balls, `BallCamera`

Not started. `limelight` is a config name. `BallFollowTest` is an empty file.

From the project notes, still the plan when this work starts: a color pipeline at about 90 fps, against about 11 fps for a neural detector. Pollen (yellow) is the easy case. Red nectar gets confused with tape, panels, and skin. Ignore the first 1 or 2 frames after a pipeline switch. Keep a saved copy of each pipeline, because a Limelight OS update can wipe them. Ball tracking is off the path to the first 16-of-20 gate. The shooter presets come first.

## Localization and Pedro Pathing

Pinpoint carries the field pose. In `Pinpoint.java` the offsets are `-84.0, -168.0` mm, the pod type is `goBILDA_4_BAR_POD`, and both encoder directions are `FORWARD`. Those three lines are copied from the SDK sample `SensorGoBildaPinpoint`, including the comment "tuned for 3110-0002-0001 Product Insight #1". They are not measurements of this robot. `resetPosAndIMU()` runs inside the constructor, so every init recalibrates the IMU and the robot has to be still.

Pedro Pathing is not on `master`. It is one commit past the skeleton merge, on `origin/Pedropathing` (`6717735`), and it does not contain the hive-camera work. That branch adds `com.pedropathing:revhub:3.0.2` and `com.pedropathing:tuning:1.0.0`, plus tuner procedures. `pedro/Constants.create` returns null, so a follower is not wired to `Robot`. Rebase that branch onto `master` after the drive directions and the pod offsets are measured. The Pinpoint tuner on that branch is the tool that should replace the sample offsets. Merging it as it sits will drop `TurretCamera` back to an empty file and rename `hive_camera` toward `ball_camera`.

## Rules that touch the software

Four limits shape the code. The starting-size and expansion limits are mechanical. A software clamp on a turret, a hood, or the flower arm does not satisfy them.

| Rule | What it says | What it means here | Status |
| --- | --- | --- | --- |
| R503 | 8 motors and 8 servos maximum | 8 motors and 7 servos. Nothing motor-shaped can be added. `traction` and `backspin` cannot come back. | Verified, TU-00 |
| Servo power | 8 W and 4 A at 6 V. Axon MAX+ and MAX MK2 are pre-verified. | No servo documentation packet for the Axons. | Verified, inspection reference 26-27.1 |
| R105 | Start inside an 18 in cube. Expanded size at most 18 × 24 × 29 in, held by construction. | Stops and hard limits on the turrets, hoods, and flower arm. | Verified, Section 12, TU08 |
| G407 | At most 4 balls controlled, pollen and nectar together | One count across both lanes. Stop the intake early by the balls still in transit. | Project notes, V1 manual |
| G408 | No opponent nectar | Needs color from the transfer sensors, or a mouth sensor. After the qualifier. | Project notes, V1 manual |
| G410 | Nectar into flowers is barred until 60 seconds remain | If this robot shoots nectar into flowers, `MatchState` blocks that shot from the match clock. | Project notes, V1 manual |

The G-rule notes came from the V1 manual. Section 12 is already at TU08. Re-read G407, G408, and G410 in the current manual before the scrimmage.

## Bring-up order

One device at a time. Shot tuning is last because it is the long job. The first two steps are in the code already and are worth doing before any new subsystem.

1. **Names.** Rewrite `HardwareNames` to the table above, and the driver-station config with it. Disable `DriveOpMode` so only `TestDriveOpMode` drives. Point `HiveAimTest` at `TurretCamera` alone, and add a way to set the alliance. Remove `telemetry.update()` from `MecanumDrive`.
2. **Prove the config.** "Test Hardware" on every name: direction, servo sweep stopped short of the stops, color sensor alive, Pinpoint pose, analog voltage if a turret wire is plugged in. Done when every device moves or reads.
3. **Drive.** Confirm wheel directions. Edge-trigger the pose reset. Turn on bulk reads. Then field-centric, using the Pinpoint heading, with the reset aimed at a marked corner.
4. **Pinpoint measurement.** Confirm 4-bar pods by looking at them. Push the robot and fix encoder directions. Replace `-84, -168` with measured pod offsets. Record original vs V2.
5. **Intake and transfer.** On, off, reverse. Log proximity for 1, 2, 3, and 4 balls. Run the encoder-position bench test.
6. **One shooter, by hand.** Pollen first. `ShooterTest` sets RPM, hood, and gate from the gamepad.
7. **Turrets.** The 10-minute test, then the keep-out pairs.
8. **Second shooter.** Same `Shooter`, `Hood`, `Gate`, and `Turret` classes. Its own config names.
9. **`Shot` and the spot presets.** Tune the table. Flower arm positions after the wedge exists.
10. **Hive camera.** Manual exposure and gain. Driver button remains the fallback.
11. **Limelight.** After a shooter can hit the 16-of-20 gate without it.

Measuring 16 of 20 takes 20 shots, five 4-ball volleys. Plan 5 to 8 minutes per spot per shooter (our estimate; 80% of the guesses fell between 4 and 12 minutes). Redo a spot after any hood change.

Dates from the software path. The scrimmage date is assumed: scrimmage 12 Dec 2026, release-candidate tag 18 Dec 2026, code freeze 8 Jan 2027, qualifier 16 Jan 2027. If the second turret is under an 80% hit rate at the scrimmage, freeze to one turret.

## Open questions

The first three decide the most code. Question 7 is partly answered by the repo; the hardware half is still open.

| # | Question | Why it matters | How to find out | If the answer differs |
| --- | --- | --- | --- | --- |
| 1 | Where are the color sensors relative to the ramp legs, and how many balls fit between the mouth and the first sensor? | Whether one count covers both lanes, and how early the intake stops | Look, and measure with balls | After the legs, each sensor sees one lane. Add a sensor on the shared path. Then set the config names. |
| 2 | Turret gear ratio, range, homing method, and are the feedback wires connected? | Aim range, and whether init snaps the turret | The 10-minute test | No feedback wire: home on a hard stop or a limit switch. |
| 3 | Do the turret sweeps overlap or spoil each other's shot? | The keep-out table | Rotate both by hand through their ranges | Clamp the combined angles. R105 still needs metal stops. |
| 4 | Launcher motor model, in ticks per revolution | RPM to ticks per second | Read the labels | Velocity code waits on this number. |
| 5 | Which hub and port is each device on, and are two analog ports free? | The config, and turret feedback | Trace the wires. The port numbers in `MecanumDrive` are comments. | Move a servo or a sensor. |
| 6 | Who fires, and how? One button per shooter, one driver or two, one ball or a volley? | The trigger into `Shot` | Ask the drivers | The six states stay. |
| 7 | Which Pinpoint is installed, and what are the real pod offsets? | The pose the aim math uses | Read the unit. The code's offsets are the SDK sample. Pedro is on `origin/Pedropathing`, not on `master`. | V2 may use another I2C address. Rebase Pedro only after the offsets are measured. |
| 8 | Is the flower wedge done, and what are stow, hover, down, and the stowed size? | `FlowerArm`, and R105 | Build it, then measure | If it breaks the 18 in cube, redesign the arm. |
| 9 | Does any wired device still match `traction`, `backspin`, or `intake_deploy`? | Those names are retired above | Look at the robot for two minutes | If one of them is real, the 8-motor / 7-servo inventory is wrong. Stop and recount. |

## Recorded turret numbers

Fill this in from the test. Empty on purpose.

| | Nectar (front) | Pollen (rear) |
| --- | --- | --- |
| Servo model (MAX+ or MK2) | | |
| Gear ratio, servo to turret | | |
| Working servo range (µs, or 0–1) | | |
| Working turret range (deg) | | |
| Feedback volts at each end | | |
| Wrap point inside the range? | | |
| Home angle (deg) | | |
| Keep-out against the other turret | | |

## Sources

Pages opened 21 Nov 2026:

- REV Color Sensor V3 (REV-31-1557): address 0x52, proximity 1 to 10 cm
- REV docs, I2C sensors: same-address devices need separate buses
- Pinpoint original user guide (ServoCity): address 0x31, 400 kHz
- goBILDA Pinpoint V2: differences from the original; address not stated
- goBILDA Axon MAX Servo MK2: range, speed, stall current, feedback wire
- Game Manual 0, choosing a servo: hub pulse range
- FIRST BIOBUZZ Team Update 00: R503
- FIRST inspection reference 26-27.1: servo limits and pre-verified list
- FIRST Competition Manual Section 12 (TU08): R105

Repo, read 21 Nov 2026: `HardwareNames`, `Robot`, `Subsystem`, `MecanumDrive`, `Pinpoint`, `TurretCamera`, `MainTeleOp`, `DriveOpMode`, `HiveAimTest`, the empty subsystem and test files, `build.dependencies.gradle` (SDK 12.0.0), `UtilityTestHardware`, the SDK sample `SensorGoBildaPinpoint`, and `origin/Pedropathing` at `6717735`.

Project notes used for the hardware story and the G rules: the software path, vision research, lessons v2, the robot design interview, and the robotics overview. The robot description is the team's walkthrough on 21 Nov 2026.
