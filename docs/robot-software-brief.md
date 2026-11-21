# 3587 Scrimmage Work Plan

December 12 is likely the scrimmage. The robot is not here this week. This page is what to write now, so that the week the robot arrives is mostly testing and filling in real numbers.

## What you are finishing this week

By the end of this week, with no robot:

- The project builds.
- `HardwareNames` uses the real device names.
- Intake, transfer, shooter, and turret each have a class and a test program.
- One scrimmage TeleOp calls those classes from a button map you wrote down.
- Every number that needs the robot is a named constant, left safe (motors at 0, no servo move on init).
- You have a short checklist for the first hours with the robot.

You will not yet know motor directions, shooter speeds, hood angles, turret limits, or the Pinpoint offsets. That is expected. Those are measurements. The programs that will hold them should already exist.

## What "working" means on December 12

1. A driver can drive for a whole match.
2. The robot can pick balls up and move them to a shooter.
3. At least one shooter can score, with a person aiming.
4. In auto, the robot leaves the start and parks.

Leave-and-park is the right auto. A scoring auto is later. One good shooter is enough if the second one is not ready.

## Words

- **Teleop.** The driver-controlled part of the match.
- **Auto.** The first 30 seconds, when the robot runs alone.
- **OpMode.** One program in the driver station list.
- **Config.** The device names on the driver station. They must match `HardwareNames`, or init crashes. You will type the config on robot day. This week you only type the names in code.
- **Subsystem.** One job, in its own class, with its own test OpMode.
- **Build.** Android Studio compiles the project. A green build is how you finish a step this week. You cannot press init on the new programs until the robot is connected.

## The robot, one job at a time

Balls go in the intake, ride the transfer, and the ramp legs sort them. Nectar goes to the front shooter. Pollen goes to the back shooter. No code chooses the shooter. Each shooter has a flywheel, a hood, a gate, and a turret. The gate is the door: closed holds the ball, open lets one through. The transfer can keep running while a gate is closed.

| Job | This week you write | On the robot you measure |
| --- | --- | --- |
| Drive | Already started. Leave it working. Program name: **TestDriveOpMode** | Which way is forward on the new frame |
| Pinpoint | Leave the example offsets alone, and label them as the example | The real pod offsets |
| Hive camera | Already started. Clean up the test if you finish the list below | Does it hold the right cell |
| Intake | Class + test. In, out, stop | Power and direction |
| Transfer | Same pattern as intake | Power and direction |
| Pollen shooter | Flywheel, hood, and gate in one `Shooter` class | Ticks per rev, one speed, one hood angle, gate positions |
| Nectar shooter | A second `Shooter`, same class, its own constants | Its own numbers. Nectar is a different size |
| Both turrets | One `Turret` class, two of them | Safe range. Do this before you trust a turret |
| Pedro | The leave-and-park program, with the poses left blank | Does the path actually park |
| Flower arm, color sensors, Limelight | Leave them | Later |

Put the flywheel, the hood, and the gate in `Shooter`. Make two Shooters. Keep `Turret` as its own class. Leave `Hood.java` empty if the hood lives in `Shooter`. Leave `Baskspin.java` alone. This robot has no backspin motor.

## Rules for every class you write this week

1. Add the name to `HardwareNames` first. Use the names in the table below.
2. Nothing moves unless a button is held. Init does not start a motor or move a servo. The first time this runs on the real robot, a surprise move can hit a stop.
3. Motor power starts at 0. You will raise it on robot day.
4. A number you do not know yet gets a name in the constants file and a comment: `measure on robot day`. Do not invent a speed or an angle.
5. The test OpMode creates only the device it tests. Do not add intake, transfer, shooter, or turret into `Robot` this week. `TestDriveOpMode` builds `Robot` today. If `Robot` starts requiring devices that are not wired, the drive program will crash on init.
6. Build after each step. Read the error. Fix that step before you start the next one.
7. Intake and transfer do not have encoder cables. Leave them in the normal power mode. "Run using encoder" with no cable can send a motor to full power. Ask before you change a motor mode.

## Names

| Device | Name |
| --- | --- |
| Drive motors | `fl` `bl` `fr` `br` |
| Pinpoint | `pin` |
| Intake | `intake` |
| Transfer | `transfer` |
| Pollen flywheel, hood, gate, turret | `pollen_flywheel` `pollen_hood` `pollen_gate` `pollen_turret` |
| Nectar flywheel, hood, gate, turret | `nectar_flywheel` `nectar_hood` `nectar_gate` `nectar_turret` |
| Flower arm, later | `flower` |
| Hive webcam | `hive_camera` |
| Limelight, later | `limelight` |

`traction`, `backspin`, and `intake_deploy` are old sketch names. Do not use them for a new device. `flywheel`, `hood`, and `turret` as single names are the same problem: this robot has two of each.

## This week, in order

### 1. Write the button map on paper first

List every action. Then pick the button. Do not start coding the gamepad until this list exists. Pollen only, one driver, is enough for the first map. Nectar can wait on a "which shooter" choice if you run out of buttons.

Actions that need a control:

- Drive. Already left stick to move, right stick to turn.
- Intake in, intake out, intake stop.
- Transfer in, transfer out, transfer stop.
- Pollen flywheel on, pollen flywheel off.
- Pollen hood: nudge one way, nudge the other way.
- Pollen gate: hold to open, release to close.
- Pollen turret: nudge one way, nudge the other way.
- Reset the Pinpoint pose. Already the B button in TestDriveOpMode.
- Pick the hive cell, as a backup for the camera.

**Done when** the map is written down and you can say it without looking at the code.

### 2. Put the names in `HardwareNames`

Add the new names. Keep `fl`, `bl`, `fr`, `br`, `pin`, and `hive_camera`.

**Done when** the project builds.

### 3. Intake

Fill in `Intake.java` and `IntakeTest`. Three commands: in, out, stop. Power comes from `IntakeConstants`.

**Done when** it builds, the test uses the buttons from your map, and init does not turn the motor on.

### 4. Transfer

Add `Transfer.java` the same way. One belt feeds both shooters. The gates, not this motor, choose which shooter gets a ball.

**Done when** it builds, and you can point at the difference between this class and intake (the name, and the job). The code shape should look the same.

### 5. Shooter

Fill in `Shooter.java`, `ShooterConstants.java`, and `ShooterTest`. One class. Two sets of constants, pollen and nectar. Wire the pollen test first, then the nectar test as the same buttons with the nectar constants.

The test needs three behaviors:

- Flywheel on and off. The speed constant is a stand-in until robot day. Also add a constant for ticks per revolution, comment `read the sticker on the motor`, and leave the number unset.
- Hood nudge. Store the position in a variable that starts wherever the servo already is. Do not send it to 0 or 1 on init.
- Gate closed unless the open button is held. Add constants for the closed position and the open position, unset until robot day.

**Done when** both tests build, pollen and nectar are two objects of the same class, and you can explain which constant you will change when a shot goes too short.

### 6. Turret

Fill in `Turret.java`, `TurretConstants.java`, and `TurretZeroTest`. One class, two objects.

The test nudges the position a small step while a button is held. Add `min` and `max` constants, unset for now, and write the code so a nudge cannot go past them once those numbers exist. Init commands nothing.

**Done when** it builds, and you can explain why init must not move the turret: on the real robot it can drive into a stop.

### 7. Scrimmage TeleOp

Make a new TeleOp for the match. Leave `TestDriveOpMode` as the drive-only program. The new one uses your button map and creates intake, transfer, the pollen shooter, and the pollen turret. Include nectar in the file only if the button map has a place for it.

**Done when** it builds, every button from the paper map appears in this file, and `TestDriveOpMode` was not changed to require the new devices.

### 8. Shot sequence, in the shooter test only

After the shooter test builds, add the six steps as named states. This is how one button will eventually shoot a ball. The match program can keep using the hand controls until this has been tried on the robot.

1. **Idle.** Wheel off or slow. Gate closed.
2. **Spin up.** Wheel to the speed constant. Hood at the saved position.
3. **Aim.** Turret to the angle you asked for.
4. **Ready.** Speed has stayed near the target for a short time, and the turret has stopped moving.
5. **Feed.** Gate opens for one ball.
6. **Recover.** Wait until the speed is back, then Ready if more balls are waiting, otherwise Idle.

The "near the target" amount is a constant you will set on the robot. Until then, Ready can stay false, or it can become true when you press a test button. Write which one you chose in a comment.

**Done when** you can walk someone through the six names and say what each one is waiting for.

### 9. Hive camera, if the list above already builds

The camera code is already in `TurretCamera`. Two cleanups make robot day faster:

- `HiveAimTest` should be able to start with only the camera in the config. Today it builds a full `Robot`, so it also demands the drive and the Pinpoint.
- The alliance should be a choice you can change, not a red alliance fixed in the test.

**Done when** those two are true and the project builds. If the webcam is on the bench, try it. If the webcam is on the robot, this step is still done when it builds. Checking the picture waits for the camera.

### 10. Pedro leave-and-park

Pedro is on the `Pedropathing` branch. The hive camera is on `master`. 

What you can write this week is the auto program: three poses, named start, leave, and park. Ask a coach for those spots on the field map. Until you have the numbers, the constants stay 0 with a comment. No shooting in auto.

**Done when** the auto program builds on the branch your mentor pointed you at, and the three poses are named. Proving that it parks is robot day.

## Robot day checklist

Do this in order. Each line is a measurement or a direction fix, typed into a constant you already made. Stop on a line if the device hits something or a servo starts buzzing. Buzzing means it is pushing against metal.

1. Type the config with the same names as `HardwareNames`. Use Test Hardware, one device at a time.
2. Drive in TestDriveOpMode. Fix directions. A driver can take it around the field.
3. Intake test: set the power, fix the direction, in and out.
4. Transfer test: same.
5. Read the pollen motor sticker. Fill in ticks per revolution. Find one speed and one hood position that score a pollen ball from one spot. Fill in the gate positions. Gate stays closed until the wheel is already up to speed.
6. Pollen turret: nudge until you are near a stop, back off, write min and max. Do not drive into the stop.
7. Only then run the scrimmage TeleOp and practice pickup plus one shot.
8. Push the robot and fix the Pinpoint offsets with the Pedro tuner. Then run leave-and-park twice.
9. Repeat the shooter and turret numbers for nectar. If time runs out, the scrimmage robot is pollen only.

## Not this week

Field-centric drive, the flower arm, color sensors, ball counting, the Limelight, and a second turret that aims itself. Drivers count balls out loud and stop at 4. Do not shoot nectar into flowers until the last minute of a match.
