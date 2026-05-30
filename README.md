# Raspberry Pi 5 Initial Setup

## 1. Update Package Lists

```bash
sudo apt update
```

Refreshes the package list from repositories.

---

## 2. Upgrade Installed Packages

```bash
sudo apt upgrade -y
```

Updates installed packages automatically without confirmation.
**Note:** This may Take some time ...

---

## 3. Install Basic Dependencies

```bash
sudo apt install -y curl git wget vim htop unzip build-essential
```

Installs essential tools for downloading files, version control, editing and monitoring.

---

## 4. Install code-server

```bash
curl -fsSL https://code-server.dev/install.sh | sh

sudo systemctl enable --now code-server@$USER
```

### Configure code-server

```bash
nano ~/.config/code-server/config.yaml
```

Verify:

```yaml
bind-addr: 127.0.0.1:8080
```

Optional: change the password.

```yaml
password: your_password
```

Apply changes:

```bash
sudo systemctl restart code-server@$USER
```

Check status:

```bash
systemctl status code-server@$USER
```

### Remote Access

On the client machine:

```bash
ssh -L 8080:127.0.0.1:8080 pi@<raspi-ip>
```

Open in browser:

```text
http://127.0.0.1:8080
```

Access code-server through a secure SSH tunnel.

---

## 5. Set Hostname and Enable mDNS

```bash
sudo hostnamectl set-hostname raspi

sudo apt install -y avahi-daemon

sudo systemctl enable --now avahi-daemon
```

Connect using:

```bash
ssh raspi@raspi.local
```

Now no matter what is the ip, instead of raspi ip you can use `raspi.local` in any network.

---

## 6. ROS 2 Jazzy Build (Bookworm)

### Install Dependencies

```bash
sudo apt install -y \
  build-essential \
  cmake \
  git \
  curl \
  wget \
  locales \
  python3-dev \
  python3-pip \
  python3-venv \
  python3-numpy \
  libasio-dev \
  libtinyxml2-dev \
  libcunit1-dev \
  libssl-dev \
  libcurl4-openssl-dev \
  libacl1-dev \
  libzstd-dev \
  liblz4-dev \
  libyaml-dev \
  libconsole-bridge-dev \
  libpoco-dev \
  libeigen3-dev \
  libsqlite3-dev \
  libfmt-dev \
  libspdlog-dev \
  pkg-config \
  liblttng-ust-dev
```

### Configure Locale

```bash
sudo sed -i 's/^# *\(en_US.UTF-8 UTF-8\)/\1/' /etc/locale.gen

sudo locale-gen

sudo update-locale LANG=en_US.UTF-8

export LANG=en_US.UTF-8

echo 'export LANG=en_US.UTF-8' >> ~/.bashrc
```

### Create Python Build Environment

```bash
python3 -m venv ~/ros2_build_venv

source ~/ros2_build_venv/bin/activate
```

```bash
python -m pip install --upgrade pip

python -m pip install \
  "setuptools<81" \
  wheel \
  colcon-common-extensions \
  vcstool \
  rosdep \
  rosinstall_generator \
  catkin_pkg \
  rosdistro \
  rospkg \
  empy \
  lark \
  numpy
```

### Initialize rosdep

```bash
sudo ~/ros2_build_venv/bin/rosdep init || true

rosdep update
```

### Create Workspace

```bash
mkdir -p ~/ros2_jazzy_base/src

cd ~/ros2_jazzy_base
```

### Download ROS 2 Sources

```bash
rosinstall_generator ros_base \
  --rosdistro jazzy \
  --deps \
  --format repos > jazzy-ros-base.repos
```

```bash
vcs import src < jazzy-ros-base.repos
```

### Install ROS Dependencies

```bash
rosdep install \
  --from-paths src \
  --ignore-src \
  --rosdistro jazzy \
  --os=debian:bookworm \
  -y \
  --skip-keys "rti-connext-dds-6.0.1"
```

If `rosdep` tries to install a ROS binary package (e.g. `ros-jazzy-ros2param`), add it to `--skip-keys`.

### Build ROS 2

```bash
cd ~/ros2_jazzy_base
source ~/ros2_build_venv/bin/activate

MAKEFLAGS="-j2" colcon build \
  --symlink-install \
  --packages-up-to \
    ros2cli \
    ros2node \
    ros2topic \
    ros2service \
    ros2launch \
    ros2param \
    ros2pkg \
    ros2run \
    ros2interface \
    rclcpp \
    std_msgs \
  --cmake-args -DCMAKE_BUILD_TYPE=Release
```

### Source ROS 2

```bash
source ~/ros2_jazzy_base/install/setup.bash
```

Permanent:

```bash
echo "source ~/ros2_jazzy_base/install/setup.bash" >> ~/.bashrc
```

### Test

```bash
ros2 --help
```

```bash
python3 -c "import rclpy; print('rclpy ok')"
```
