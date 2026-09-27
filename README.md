# Slip-Aware-Autonomous-Navigation-System-for-modular-robots-on-unstructured-terrain
Most terrain-aware navigation frameworks assume robots that do not slip. A low-cost skid-steer robot has no steering mechanism and turns only by scrubbing its wheels sideways across the ground — exactly what a no-slip model cannot predict. A friction-based kinematic model that does predict this slip existed in the literature, but its authors left its integration into a real-time controller as future work.This project closes that gap.
We present a four-wheel, two-motor skid-steer robot whose navigation controller reasons about wheel slip. A friction-based kinematic model is identified from the robot's own driving data on deformable, leaf-littered terrain and embedded as the rollout dynamics of a Hybrid A*-guided Model Predictive Path Integral (MPPI) controller. A low-cost stereo depth camera replaces the reference 3D LiDAR. The entire system is designed, implemented, and evaluated in simulation on a procedurally generated Ghanaian cocoa-farm world.

## Key Results
Metric	Value
Yaw-rate prediction error(friction model) =	0.10 rad/s
Yaw-rate prediction error (no-slip baseline)= 	0.84 rad/s
Improvement in yaw-rate prediction- 87.9%
Online solver evaluation time	=2.44 µs per prediction
Control-loop cycle time-16–19 ms within a 100 ms period
Goals reached , friction rollout (original seed) = 4 / 6 vs. 2 / 6 for no-slip
Goals reached, both rollouts (adopted seed)-6 / 6
Tree trunks mapped , stereo camera vs. LiDAR= −40.8 pp (perception deficit)
Task completion , stereo vs. LiDAR =	Statistically equivalent
Obstacle clearance ,stereo vs. LiDAR= No measurable loss
Survey mission: trees visited / images captured =	5 trees / 20 images, no intervention
Camera standoff spread to trunk= ≤ 10 mm

## Friction based kinematic Model
Standard navigation frameworks model skid-steer robots with a no-slip kinematic equation:
ẋ = v · cos(θ)
ẏ = v · sin(θ)
θ̇ = ω
This fails for a skid-steer robot because every turn involves lateral wheel scrub. We adopt the friction-based kinematic model of Rabiee & Biswas (2019), which derives slip from the wheel–ground friction relationship rather than fitting an empirical curve, giving physically interpretable parameters that can be re-identified for any platform from driving data.

Open-loop validation result:
Yaw rate RMSE: 0.10 rad/s (friction model) vs. 0.84 rad/s (no-slip)
Forward velocity: friction model less accurate — acknowledged limitation
The fitted model is then wrapped in a bounded solver that evaluates each MPPI rollout sample in 2.44 µs, placing a previously offline predictor inside the real-time control loop

1. Slip-Aware MPPI
The key contribution is embedding the friction model as the rollout dynamics of the MPPI controller rather than treating the model and the controller as separate offline / online tools. This means every sampled trajectory the controller evaluates already accounts for how the wheels will actually slip during that manoeuvre.

What this buys in closed loop:
With the navigation framework's original seed settings, the friction-based rollout reaches 4/6 goals vs. 2/6 for the no-slip rollout — doubling goal success at the same hyperparameter setting. With the settings adopted here, both reach 6/6, but the friction rollout does so across a strictly wider range of tuning configurations, meaning the system is more robust to parameter variation.

2. LiDAR → Stereo Camera Substitution
The reference navigation framework (Yang et al. 2021) assumes a 3D LiDAR, which costs USD 5,000–15,000 and is out of reach for the research groups this robot is designed to serve. We substitute an Intel RealSense D435 stereo depth camera.

The stereo camera maps 40.8 percentage points fewer tree trunks due to its shorter range and narrower field of view — a real perception deficit. Despite this, it achieves statistically equivalent task-completion rate, time-to-goal, and path efficiency, with no measurable loss of obstacle clearance. The perception deficit does not propagate to task failure.

3. End-to-End Survey Mission
The complete system was demonstrated on the applied task it exists for: an autonomous tree-image survey of five cocoa trees. The robot selected camera standoff stations geometrically, drove to each, and captured 20 images without operator intervention. Camera distance to trunk was held within a 10 mm spread across all stations.

## limitations
This work is carried out and evaluated entirely in simulation. No physical prototype was built.
Specific limitations:
-Robot pose is taken from the simulator throughout  navigation results are unaffected by localisation error. Localisation under dense canopy (where GPS-RTK fails) is unresolved and identified as primary future work.
-Only two range sensors are compared: 3D LiDAR and one stereo depth camera.
The survey mission demonstrates data collection, not crop diagnosis. No disease model is trained or evaluated.
These limitations define the sim-to-real gap and motivate the graduate research direction this project opens.

## future work
The natural next step is sim-to-real transfer: building the physical robot, running the parameter-identification protocol on real terrain, and evaluating whether the navigation gains observed in simulation hold on a working farm. Specific open problems:
Localisation under canopy — GPS-RTK is unreliable on cocoa farms; a visual-inertial or LiDAR-based odometry solution is needed to close the loop on a real platform
Multi-terrain generalisation — re-identification of friction parameters on wet soil, hard-packed laterite, and root-covered ground
Forward-velocity model — the friction model's weaker accuracy on forward velocity (vs. yaw rate) is an open modelling problem
Dataset release — a public dataset of annotated cocoa-farm imagery collected by this robot would directly address the data gap the project was designed to close
