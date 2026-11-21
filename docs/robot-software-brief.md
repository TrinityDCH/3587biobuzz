# 3587 Scrimmage Work Plan

December 12 is the scrimmage. The full robot may only be here for about a week before that. This page is the work, in order. Finish a step before you start the next one.

The hardware walkthrough from November 21 is the robot you are programming. Some names already in the code are from an older sketch. When a name and the robot disagree, the robot wins. Ask a mentor before you delete old names.

## What "working" means on December 12

A working robot can do four things:

1. A driver can drive it for a whole match.
2. It can pick balls up and move them to a shooter.
3. At least one shooter can score, with a person aiming.
4. In auto, it leaves the start and parks.

Leave-and-park is the right auto. Do not build a scoring auto for this scrimmage.

You are done with the scrimmage list when those four things work twice in a row. Practice driving that robot. Extra ideas are at the bottom, and they wait until the four things work.

## Words

- **Teleop.** The part of the match where a driver drives. Two minutes.
- **Auto.** The first 30 seconds. The robot runs a program by itself.
- **OpMode.** One program in the list on the driver station.
- **Config.** The list of device names on the driver station. The name in the config and the name in `HardwareNames` have to be the same, or the program crashes when you press init.
- **Subsystem.** One job on the robot, in its own class, with its own test OpMode. The intake is one. A shooter is one.
- **Servo.** Moves to a position you choose and holds there. Hoods, gates, turrets, and the flower arm are servos.
- **Encoder.** A counter on a motor. Shooter speed control needs it. The intake and transfer cables are not plugged in yet.

## The robot, one job at a time

Balls go in the intake, ride the transfer, and the ramp legs sort them. Nectar goes to the front shooter. Pollen goes to the back shooter. No code chooses which shooter gets the ball. Each shooter has a flywheel motor, a hood servo, a gate servo, and a turret servo. The gate is the door. Closed holds the ball. Open lets one ball through. The transfer can keep running while a gate is closed.

| Job | What it is | Where you are |
| --- | --- | --- |
| Drive | 4 mecanum motors | Started. Program name on the driver station: **TestDriveOpMode** |
| Pinpoint | Tells you where the robot is and which way it faces | Started. The offset numbers in the file are from an example robot, not ours |
| Hive camera | Looks up at the hive and says which cell is up | Started. More work left. A driver button is the backup |
| Pedro | Drives a path by itself | Started, on the Pedro branch. Scrimmage goal is leave and park |
| Intake | 1 motor. In, out, stop | Not started |
| Transfer | 1 motor. In, out, stop. Feeds both shooters | Not started |
| Pollen shooter | Back shooter. Motor, hood, gate, turret | Not started. Build this shooter first |
| Nectar shooter | Front shooter. Same four parts | After pollen can score |
| Flower arm | 1 servo. Stow, hover, down | The arm is not built yet. Code waits |

Two color sensors sit on the transfer. For this scrimmage, drivers count balls out loud and stop at 4. Sensor code waits until a shooter can score.

The Limelight looks down at balls. It also waits.

## How you build each new piece

Every new device goes through the same four steps. Learn this once.

1. Put it in the driver station config and in `HardwareNames`, with the same name.
2. Make a test OpMode that runs only that device. The empty files `IntakeTest`, `ShooterTest`, and `TurretZeroTest` are there for this.
3. Drive it from the gamepad until it does the job.
4. Only then add it to the match OpMode. One person adds things to `Robot`. Everyone else stays in a test OpMode.

Use **Test Hardware** in the Utility menu on the day a wire is new. It moves one device at a time. Let go if a servo buzzes or anything hits a stop. Buzzing means it is pushing against metal.

Shooter motors use the encoder. Intake and transfer do not have encoder cables yet. Leave those two in the normal power mode. Turning on "run using encoder" with no cable can send a motor to full power. Ask before you change a motor mode.

## Names to use

Use these so the config, the code, and the conversation match.

| Device | Name |
| --- | --- |
| Drive motors | `fl` `bl` `fr` `br` |
| Pinpoint | `pin` |
| Intake motor | `intake` |
| Transfer motor | `transfer` |
| Pollen flywheel, hood, gate, turret | `pollen_flywheel` `pollen_hood` `pollen_gate` `pollen_turret` |
| Nectar flywheel, hood, gate, turret | `nectar_flywheel` `nectar_hood` `nectar_gate` `nectar_turret` |
| Flower arm | `flower` |
| Hive webcam | `hive_camera` |
| Limelight | `limelight` |

`traction`, `backspin`, and `intake_deploy` are from the old sketch. Do not wire a device to those names. The single names `flywheel`, `hood`, and `turret` are the same idea: this robot has two of each. Color-sensor names wait until you can see where the sensors sit.

## This week, before the rest of the robot is here

You already have a drive, a Pinpoint, and a hive camera. Two people can split steps 3 and 4. Steps 1 and 2 come first.

### 1. Drive

Use TestDriveOpMode. There is a second drive program called DriveOpMode. Ignore it, and ask a mentor to hide it so nobody inits the wrong one.

**Done when** a driver can go forward, back, strafe, and turn, and can say which stick does which. If the robot keeps rolling after the sticks come back, ask about brake mode.

### 2. Pinpoint

Push the robot by hand. Forward should change one number. Strafing should change the other. Turning should change the heading. The B button sets the position back to zero.

**Done when** those three pushes do what you expect, and you have written down that the offset numbers still belong to the example robot. Measuring our offsets is step 7. Do not tune shots or paths on the example numbers.

### 3. Hive camera

The camera looks up. Each hive cell has a group of AprilTags. The code already ignores a cell that is tipped over, ignores the other alliance, and waits until it sees the same cell a few times.

**Done when** you can tell it which alliance you are on, point it at the hive, and it reports one cell and keeps that answer. Write down what it does when the light is bad or the hive is moving. The scrimmage plan still has a driver button to pick the cell. The camera helps. It does not have to be perfect, and it does not block the shooter work. Keep one person on this in the background.

### 4. Pedro, leave and park

Pedro is how auto will drive. It is started on the Pedro branch. The hive camera work is on `master`. Ask a mentor before you merge the two. A bad merge can wipe the camera code.

On the drive you have now:

1. Get Pedro running and follow one straight line.
2. Add a turn.
3. Write one auto that leaves the starting line and ends in the parking spot your coaches show you.

**Done when** that path finishes near the right place two times in a row. No shooting in auto.

## The week the robot is on the table

Same order. Do not start the shooter because the shooter is more interesting. A shooter with no intake is a demo. A robot that can drive, pick up, and shoot is the scrimmage.

### 5. Name every wire

Put each motor, servo, and sensor in the config with the names above. Test Hardware on each one. Write down which way is forward.

**Done when** every device moves or reads, and a servo has never been driven into a stop.

### 6. Drive on the real frame

Directions can flip when the motors move to the new robot. Fix them in the drive code, then drive slowly around the field.

**Done when** a driver can take it around the field without a mentor on the sticks.

### 7. Measure the Pinpoint, then park

The odometry pods are in a new place, so the example offsets are wrong on this frame. Use the Pinpoint tuner in Pedro. Check which way each pod counts by pushing the robot.

**Done when** a leave-and-park auto works on the field twice in a row. That finishes auto for the scrimmage.

### 8. Intake

Test OpMode first. One button in, one button out, one button stop. Out is how you clear a jam.

**Done when** it takes balls in and can spit a jammed ball back out.

### 9. Transfer

Same three buttons. This one belt feeds both shooters. The gates decide which shooter receives a ball.

**Done when** balls move to the shooter, and reverse brings them back.

### 10. Pollen shooter, by hand

Rear shooter. Pollen first, because the field has many more pollen balls than nectar. Three parts, in this order, in `ShooterTest`:

1. **Flywheel.** Read the sticker on the motor and write down the ticks per revolution. You need that number before speed control means anything. Find one speed that launches a pollen ball.
2. **Hood.** Find one angle that makes that shot go in. Write the number down.
3. **Gate.** The program starts with the gate closed. Open it only after the wheel is already up to speed. One open lets one ball through.

Aim by hand for this step. A person can point the robot, or point the turret once step 11 is safe.

**Done when** you can score a pollen ball from one marked spot on the field, several times.

### 11. Pollen turret

Do this before you trust the turret in a match.

1. Find the gear ratio from the servo to the turret, if there is one.
2. Move the servo a small step at a time. Write down where it hits a stop or pulls a wire. Stay away from that point.
3. If the extra feedback wire is plugged into an analog port, write down the voltage at each end of the safe range. That reading can tell you the angle later without a homing sweep.
4. When the program starts, command the turret to stay where it already is. A turret that snaps to the middle on init will hit something.

**Done when** the driver can aim it inside the safe range, and it has not hit a stop. Write the range in `TurretConstants`.

### 12. One match program

Put drive, intake, transfer, and the pollen shooter into one TeleOp. This is the program you will drive at the scrimmage if time runs out.

**Done when** one driver can pick up balls and score them with pollen, with no test OpModes open. Then practice.

### 13. Nectar shooter

Front shooter. Same pieces: flywheel, hood, gate, turret. Nectar is a different size, so the hood number and the wheel speed will be different. Test them. Do not copy the pollen numbers and stop.

**Done when** it scores from one spot, or the team decides pollen-only for the scrimmage. One good shooter is a working robot. A second turret that is not ready stays off the match program.

## Rules to follow while you drive, even before there is code for them

- Hold at most 4 balls, pollen and nectar together. Count out loud.
- Do not shoot nectar into flowers until the last minute of the match. If you are not sure, don't.
- Turrets, hoods, and the flower arm need metal stops. A limit in the code is not enough to keep the robot inside the starting size.

## If the four scrimmage goals already work

Only then, and in this order:

1. **Field-centric drive.** The robot drives the same direction no matter which way it is facing, using the Pinpoint heading. The drive you have now does not do this yet.
2. **Second turret**, with a written rule for angles they are not both allowed to use, so they cannot hit each other.
3. **A shot sequence** on one shooter: spin up, aim, wait until the speed holds steady, open the gate for one ball, wait for the speed to come back, repeat. Spin the two flywheels up one at a time so the battery does not sag as hard.
4. **Flower arm**, after a builder says the wedge is actually done. Three positions: stow, hover, down. Check that stow fits in the 18 inch starting box.
5. **Color sensors.** First print "ball" or "no ball" from each sensor. Look at where they sit relative to the ramp legs before you try to count. Counting and "what color is this ball" come after that printout looks right with 1, 2, 3, and 4 real balls.
6. **Limelight**, to find balls on the floor.

Shot presets for lots of field spots, and 16 makes out of 20 shots, are the tuning project after one spot already scores. They are not the scrimmage gate.

## For mentors

Students can stop reading here.

- Hardware source of truth is the November 21 walkthrough: 8 motors, 7 servos, pollen rear, nectar front, geared counter rollers, flower arm unfinished. `master` at `781ce24` is still the single-shooter sketch plus a working robot-centric drive, Pinpoint, and `TurretCamera`.
- `MecanumDrive` never uses the Pinpoint it is given. Gamepad B zeros the pose on every loop while held. `writeTelemetry` calls `telemetry.update()` and splits the driver-station packet. `DriveOpMode` is also enabled.
- Pinpoint offsets `-84, -168` mm and the 4-bar pod setting are copied from the SDK sample `SensorGoBildaPinpoint`.
- `origin/Pedropathing` is one commit past the skeleton and does not contain the hive camera. `pedro/Constants.create` returns null. Rebase onto `master` after offsets are measured. Do not merge it over the camera work.
- `HiveAimTest` builds a full `Robot`, so init requires the drive and the Pinpoint, and it locks the alliance to red. `TurretCamera` itself fails soft if `hive_camera` is missing.
- `Baskspin.java` is a typo for a backspin motor this robot does not have. Do not let anyone implement it.
- Open on the real robot, and blocking only the steps named above: color-sensor placement, turret range and feedback wires, whether the turrets overlap, shooter motor ticks per rev, Pinpoint version, and the wiring map. The port numbers in `MecanumDrive` are comments.
- Pollen-only, driver-aimed, with leave-and-park, meets the scrimmage bar if step 13 does not land.
