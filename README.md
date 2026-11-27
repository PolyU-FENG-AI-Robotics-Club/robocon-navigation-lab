# Robocon 2027 Navigation Lab

Practice mapping, restarting and driving a simulated robot around an obstacle. Then adapt the same workflow to the Builder Robot (BR) route to the handover area and two tower positions.

## Student downloads

### Week 1 — Navigation workshop

1. **Start here.** Download the [main course ZIP][main-course-zip] before class and follow the Ubuntu setup below.
2. [English course slides (PowerPoint)][english-slides] walk through the main lab and the homework.
3. **After class.** Download the [vision replay ZIP][vision-replay-zip] for the optional ORB-SLAM3 and cuVSLAM homework.

### Week 2 — Learning materials and assignment

Practise RGB-D target localisation, A* detouring and updating the stopping point while the robot is moving. For the assignment, run the lab and then use the competition field to visit two box stops and the cube-flip zone.

- [English handout and assignment (Word)](https://github.com/PolyU-FENG-AI-Robotics-Club/robocon-navigation-lab/releases/download/v1.0.0/Robocon_2027_Vision_and_Navigation_Lab_English_Rolling_Vision.docx)
- [Pixel navigation add-on v2 (ZIP)](https://github.com/PolyU-FENG-AI-Robotics-Club/robocon-navigation-lab/releases/download/v1.0.0/Robocon_2027_Pixel_Navigation_Lab_Addon_v2.zip). Install this first for the manual pixel exercises and as a prerequisite for rolling vision.
- [Rolling vision add-on (ZIP)](https://github.com/PolyU-FENG-AI-Robotics-Club/robocon-navigation-lab/releases/download/v1.0.0/Robocon_2027_Rolling_Vision_Lab_Addon.zip). Use this for the moving recognition exercise.

Materials for later weeks will be added below in teaching order.

[main-course-zip]: https://github.com/PolyU-FENG-AI-Robotics-Club/robocon-navigation-lab/releases/download/v1.0.0/Robocon_Navigation_Lab_Student_Files.zip
[english-slides]: https://github.com/PolyU-FENG-AI-Robotics-Club/robocon-navigation-lab/releases/download/v1.0.0/Robocon_2027_Navigation_Lab_English_Student_Release.pptx
[vision-replay-zip]: https://github.com/PolyU-FENG-AI-Robotics-Club/robocon-navigation-lab/releases/download/v1.0.0/Robocon_Vision_Replay_Starter.zip

## 1. Get the course code and start Ubuntu

Extract the [main course ZIP][main-course-zip] into your Ubuntu home folder. The project folder is `~/robocon-navigation-lab`. Use Ubuntu 24.04 x86-64 with Docker and internet access for the first build. The container uses ROS 2 Jazzy and Gazebo Harmonic.

Run these commands in the **Ubuntu host** terminal. If Docker is already installed, start with `sudo docker run --rm hello-world`.

```bash
sudo apt update
sudo apt install -y docker.io git unzip
sudo systemctl enable --now docker
sudo docker run --rm hello-world
```

Check that you see `Hello from Docker`. Start the lab from the project folder.

```bash
cd ~/robocon-navigation-lab
bash start_lab.sh
```

Use the same command for every session. The launcher creates the container on first use, builds missing course dependencies, starts the virtual display and opens an Ubuntu shell inside `/work`. If an older project already uses the name `robocon-intro`, the launcher creates a separate container for this project. Complete the first build before class and wait for `Lab ready in /work`. Type `exit` to return to your Ubuntu terminal.

Use an interactive Ubuntu terminal to start the launcher. If a previous command left Bash at a `>` prompt, press Ctrl+C to cancel that unfinished command. If setup stops, save the first error and retry the launcher after resolving it. Use one simulation at a time. Run the remaining commands **inside the container**.

### Upstream code you download

Setup runs `scripts/download_upstream.sh` after installing Git. Read that script to see the exact repositories, checkout directories and commit IDs. The full developer repositories are downloaded into `jazzy_ws/` on your computer.

| Upstream project | What it provides | Revision |
| --- | --- | --- |
| [FASTLIO2_ROS2](https://github.com/liangheming/FASTLIO2_ROS2) | LiDAR/IMU mapping, pose graph and ICP localizer | `7baa59f7339ba88ce3b646fe35b23f61031c5d3f` |
| [livox_gz_sim](https://github.com/mathaimp/livox_gz_sim) | Simulated MID-360 LiDAR and IMU | `3f06f1ac4d3f6854f7275ca9aa097484791a1abd` |
| [Sophus](https://github.com/strasdat/Sophus) | Pose mathematics | `de0f8d3d92bf776271e16de56d1803940ebccab9` |
| [GTSAM](https://github.com/borglab/gtsam) | Pose graph optimization | `c57988fe554e7213c77fe379c1d7c483de26ad33` |
| [Livox ROS Driver 2](https://github.com/Livox-SDK/livox_ros_driver2) | Two ROS message definitions for simulated scans | `21445540f0d100dc86a7e6df312dd70bbdb4afdf` |

The course includes Gazebo scene generation, the A* grid and position controller, the simulation runner, the dashboard, the saved teaching map and compatibility scripts. The Livox simulation helper builds the two downloaded messages and retains the upstream license. It uses no physical LiDAR in this lab. See the [Gazebo Harmonic and ROS compatibility guide](https://gazebosim.org/docs/harmonic/ros_installation/) for the platform pairing.

## 2. Run the first navigation test with the course map

Use the original field and the original configuration for the first run. This restores the map pointer to the included teaching map.

```bash
source /work/env.sh
export ROS_DOMAIN_ID=79
export GZ_PARTITION=robocon_bias_video
export COMPLEX_CAMERA_HZ=2 LP_NUM_THREADS=4
cd /work/initial_guess_video_20261116
printf '%s\n' /work/course_map/map.pcd > latest_map.txt
python3 run.py navigation --record
```

The robot starts 0.20 m away from S on both axes, turned 15° clockwise. ICP receives a zero guess relative to S and aligns the scan with the existing map. Then A* plans and position feedback moves the robot. Watch for `RETURN 0`.

## 3. Build your own map and navigate on it

```bash
source /work/env.sh
export ROS_DOMAIN_ID=76
export GZ_PARTITION=robocon_complex
export COMPLEX_CAMERA_HZ=2 LP_NUM_THREADS=4
cd /work/complex_demo_20261113
python3 run.py mapping --record
```

When you see `RETURN 0`, run `cat latest_map.txt`. The returned path should end in `map.pcd`. The next block copies this pointer to the navigation demo and runs the same ICP → A* → controller sequence.

```bash
source /work/env.sh
export ROS_DOMAIN_ID=79
export GZ_PARTITION=robocon_bias_video
export COMPLEX_CAMERA_HZ=2 LP_NUM_THREADS=4
cd /work/initial_guess_video_20261116
cp /work/complex_demo_20261113/latest_map.txt .
python3 run.py navigation --record
```

## 4. Check the results

After the run, evaluate it while using the same scene settings.

```bash
cd /work/initial_guess_video_20261116
run_dir=$(cat latest_navigation.txt)
printf '%s\n' "$run_dir"
python3 evaluate.py "$run_dir"
```

- `status: COMPLETE` means the script completed the route. Check `truth_final_error_m` for the distance to the goal and `minimum_physical_clearance_m` for the evaluated clearance. Simulated ground truth measures the result. The controller uses the estimated pose.
- Open `recording.mp4` and find the orange plan and blue trail. On the Ubuntu host, replace `/work` in the printed path with `~/robocon-navigation-lab`.
- If a run stops, read the terminal error and `task_result.json` if available. The Gazebo and localization logs are in the same run folder. Record the first error with the run path.

Next use [Change the field and plan the BR exercise](docs/change-field.md) for the target, obstacle and checkpoint activities.

## 5. Optional visual localisation homework

Follow [RTAB-Map in Gazebo](extras/RTABMAP.md) for the RGB-D exercise. For ORB-SLAM3 or cuVSLAM, download the [vision replay ZIP][vision-replay-zip]. It includes the Gazebo stereo images, IMU measurements and starter scripts. See the [replay README](extras/vision-replay/README.md) for the setup and result files.
