# Setting up simulation environment (Windows — WSL2)

The simulation environment tools only work in Linux. For Windows users, WSL2 (Windows Subsystem for Linux 2) is the recommended approach. It runs a real Ubuntu 22.04 environment directly inside Windows without needing a separate VM.

> **WSL2 vs Dual Boot:** Dual boot gives better performance but requires partitioning your drive. WSL2 is easier to set up and works well for this project. If you already dual boot Ubuntu 22.04, skip straight to [Install ROS 2](#install-ros-2).

---

## Important: Two types of terminals

You will use **two different terminals** throughout this guide. Never mix them up — this is the most common reason installation fails.

| Terminal | When to use |
|---|---|
| **Windows Terminal / PowerShell** | Only for WSL2 setup steps (Section 1) |
| **WSL2 Ubuntu terminal** | Everything else — all ROS 2, Gazebo, ArduPilot commands |

I will use the term **WSL terminal** for the Ubuntu shell inside WSL2, and **Windows terminal** for PowerShell or Windows Terminal. Once WSL2 is set up, almost everything runs in the WSL terminal.

---

## Step 1 — Install WSL2

Open **PowerShell as Administrator** (right click Start → "Windows Terminal (Admin)" or search PowerShell → Run as Administrator) and run:

```powershell
wsl --install -d Ubuntu-22.04
```

This installs WSL2 and Ubuntu 22.04 automatically. It will ask you to **restart your computer**. Do so.

After restart, Ubuntu will open and ask you to create a username and password. Choose anything you like — this is your Linux account inside WSL2.

> If you already have WSL but an older version, upgrade it first:
> ```powershell
> wsl --update
> wsl --set-default-version 2
> ```

Verify WSL2 is installed correctly by running in PowerShell:

```powershell
wsl --list --verbose
```

You should see `Ubuntu-22.04` with **VERSION 2**. If it shows version 1, run:

```powershell
wsl --set-version Ubuntu-22.04 2
```

---

## Step 2 — Open the WSL Terminal

From now on, open your Ubuntu WSL2 shell by either:
- Searching **"Ubuntu"** in the Start menu and opening it, or
- Opening Windows Terminal and clicking the **Ubuntu-22.04** tab from the dropdown

Your prompt should look like:
```
yourname@DESKTOP-XXXXX:~$
```

All commands from this point forward go in this WSL terminal unless stated otherwise.

---

## Install ROS 2

Paste the following lines in the **WSL terminal**:

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y wget git sudo python3-pip python3-dev software-properties-common curl tzdata keyboard-configuration

# Locale
sudo apt update && sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

# ROS 2 repo
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository universe -y
sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update
sudo apt install ros-humble-desktop ros-dev-tools -y
source /opt/ros/humble/setup.bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

This will take around **10–15 minutes**. If it pauses asking about keyboard layout or timezone, just press Enter to accept defaults.

Verify:
```bash
ros2 --help
```
You should see a list of commands like `topic`, `node`, `launch`, etc.

---

## Install Gazebo

Paste the following in the **WSL terminal**:

```bash
sudo curl https://packages.osrfoundation.org/gazebo.gpg \
  --output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] \
  http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" \
  | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null

sudo apt update
sudo apt install gz-harmonic -y
```

This takes around **10–20 minutes**.

Verify:
```bash
gz sim --version
```
You should see `Gazebo Sim, version 8.x.x`.

> **WSL2 display note:** Gazebo's 3D GUI requires a display. WSL2 on Windows 11 supports this natively via WSLg (built-in). On Windows 10, you need to install an X server like [VcXsrv](https://sourceforge.net/projects/vcxsrv/) and set `export DISPLAY=:0` in your `~/.bashrc`. Windows 11 is strongly recommended.

---

## ROS Gazebo Bridge

Run the following in the **WSL terminal**:

```bash
sudo rosdep init
sudo wget https://raw.githubusercontent.com/osrf/osrf-rosdep/master/gz/00-gazebo.list \
  -O /etc/ros/rosdep/sources.list.d/00-gazebo.list
rosdep update

mkdir -p ~/ros_gz_ws/src
cd ~/ros_gz_ws/src
git clone https://github.com/gazebosim/ros_gz.git -b humble

cd ~/ros_gz_ws
export GZ_VERSION=harmonic

rosdep install -r --from-paths src -i -y --rosdistro humble

# Limit parallelism to avoid running out of memory
export MAKEFLAGS="-j 1"
colcon build --parallel-workers 1
source ~/ros_gz_ws/install/setup.bash
echo "source ~/ros_gz_ws/install/setup.bash" >> ~/.bashrc
```

This will take around **10 minutes** to compile. You will see lines like `Starting >>> ros_gz_bridge` and `Finished <<<` — that is normal.

> **If the build runs out of memory:** WSL2 by default uses 50% of your RAM. If your PC has less than 8GB RAM, create a file at `C:\Users\YourName\.wslconfig` and add:
> ```
> [wsl2]
> memory=6GB
> processors=2
> ```
> Then restart WSL: run `wsl --shutdown` in PowerShell, then reopen Ubuntu.

---

## ArduPilot

Run the following in the **WSL terminal**:

```bash
cd ~
git clone --recurse-submodules https://github.com/ArduPilot/ardupilot.git
cd ardupilot
Tools/environment_install/install-prereqs-ubuntu.sh -y
```

The clone may take a few minutes due to submodules. After the install script finishes, reload your shell:

```bash
source ~/.bashrc
```

Verify:
```bash
which sim_vehicle.py
```

You should see a path like `/home/yourname/.local/bin/sim_vehicle.py`. If there is no output, run:

```bash
echo 'export PATH=$PATH:$HOME/ardupilot/Tools/autotest' >> ~/.bashrc
source ~/.bashrc
which sim_vehicle.py
```

---

## ArduPilot Gazebo Plugin

Run in the **WSL terminal**:

```bash
sudo apt update
sudo apt install libgz-sim8-dev rapidjson-dev -y
sudo apt install libopencv-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl -y
sudo apt update
sudo apt install libdebuginfod1 libdebuginfod-dev -y

mkdir -p ~/gz_ws/src && cd ~/gz_ws/src
git clone https://github.com/ArduPilot/ardupilot_gazebo
export GZ_VERSION=harmonic
cd ardupilot_gazebo
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo
make -j4
```

After the build completes (you should see `[100%] Built target GstCameraPlugin`), set the environment variables:

```bash
export GZ_SIM_SYSTEM_PLUGIN_PATH=$HOME/gz_ws/src/ardupilot_gazebo/build
export GZ_SIM_RESOURCE_PATH=$HOME/gz_ws/src/ardupilot_gazebo/models
echo 'export GZ_SIM_SYSTEM_PLUGIN_PATH=$HOME/gz_ws/src/ardupilot_gazebo/build' >> ~/.bashrc
echo 'export GZ_SIM_RESOURCE_PATH=$HOME/gz_ws/src/ardupilot_gazebo/models' >> ~/.bashrc
source ~/.bashrc
```

Verify:
```bash
grep "GZ_SIM" ~/.bashrc
```

You should see both `GZ_SIM_SYSTEM_PLUGIN_PATH` and `GZ_SIM_RESOURCE_PATH` lines.

---

## Final Test

Install QGroundControl on your Windows machine [https://qgroundcontrol.com/](https://qgroundcontrol.com/) and open it.

Clone the SLAM repo and run the simulation. Open **two WSL terminals** side by side.

In the **first WSL terminal** run:
```bash
cd ~
git clone https://github.com/abhishekjain1612006/AeroClub-SLAM.git
cd AeroClub-SLAM
gz sim -r -v4 world.sdf
```

In the **second WSL terminal** run:
```bash
sim_vehicle.py -v ArduCopter -f gazebo-iris --frame JSON
```

> This command will compile the ArduPilot codebase the first time you run it — it takes **5–10 minutes**. Subsequent runs are instant.

You should see **"Ready to Fly"** on QGroundControl after a while, and a Gazebo window with a house model and a drone sitting in it. You can try sending takeoff and land commands to the drone using QGC.

---

## Daily Startup (after first install)

Once everything is installed, here is all you need to do each day:

Open **two WSL terminals** and run:

**Terminal 1:**
```bash
cd ~/AeroClub-SLAM
gz sim -r -v4 world.sdf
```

**Terminal 2:**
```bash
sim_vehicle.py -v ArduCopter -f gazebo-iris --frame JSON
```

Then open QGroundControl on Windows. That's it.

---

## Verification Commands (run anytime to check your install)

```bash
ros2 --help                          # ROS 2 installed?
gz sim --version                     # Gazebo installed?
grep "ros" ~/.bashrc                 # ROS 2 sourced?
grep "ros_gz_ws" ~/.bashrc           # Bridge sourced?
which sim_vehicle.py                 # ArduPilot in PATH?
ls ~/gz_ws/src/ardupilot_gazebo/build/libArduPilotPlugin.so   # Plugin built?
grep "GZ_SIM" ~/.bashrc              # Plugin env vars set?
ls ~/AeroClub-SLAM/world.sdf         # SLAM repo cloned?
```

---

## Common Issues

**`wsl: command not found` in PowerShell**
→ You need Windows 10 version 2004 or higher, or Windows 11. Update Windows first.

**Gazebo window doesn't open on Windows 10**
→ WSLg (built-in display support) is Windows 11 only. On Windows 10, install VcXsrv, launch it, then add `export DISPLAY=:0` to your `~/.bashrc`.

**`colcon build` crashes or hangs**
→ WSL2 ran out of memory. Create `.wslconfig` as described in the ROS Gazebo Bridge section and restart WSL.

**`sim_vehicle.py: command not found`**
→ Run `echo 'export PATH=$PATH:$HOME/ardupilot/Tools/autotest' >> ~/.bashrc && source ~/.bashrc`

**QGroundControl doesn't connect**
→ Make sure both WSL terminals are running (Gazebo and ArduPilot). QGC connects automatically on UDP — give it 30 seconds after ArduPilot starts.

**ArduPilot clone fails mid-way**
→ Run `rm -rf ~/ardupilot` then clone again with `git clone --recurse-submodules --shallow-submodules https://github.com/ArduPilot/ardupilot.git`
