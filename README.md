# Migration from ROS1 to ROS2 on Turtlebot3 named as InsperBot

Hi! I hope that with this step-by-step guide, I can assist in updating your educational or domestic robot from ROS1 to ROS2. Within this repository, you will find what has been developed to meet the needs of our institution's robotics laboratory.

This project was carried out at the Robotics Laboratory of INSPER, for the Computational Robotics course. I had the assistance of Lícia Sales, a skilled programmer and deep connoisseur of this framework.

Contributors: Rogério and Lícia

A Escolha de utilizar 


Materials Used:

- TurtleBot 3 Burger
    - Raspberry Pi 4
    - OpenCR
    - Realsense D435i (or Raspi-Cam)


Systems Used:

Computer:
    - Ubuntu 22.04 LTS
    - ROS2 Humble Hawksbill
Raspberry Pi 4:
    - Ubuntu 22.04 Server for Raspberry Pi 4
    - ROS2 Humble Hawksbill

Sites:
[Robotis (E-Manual)](https://emanual.robotis.com/docs/en/platform/turtlebot3/sbc_setup/)
[Ubuntu 12.04 for Raspi4](https://ubuntu.com/download/raspberry-pi) 
[ROS2 Humble](https://docs.ros.org/en/humble/index.html)
[ROS2 Wraper Intel Realsense](https://github.com/IntelRealSense/realsense-ros)



## Part #1 - Raspberry Pi 4 SD Card Preparation

Installing the Ubuntu 22.04 Server:
    - Run Raspberry Pi Imager - [LINK](https://www.raspberrypi.com/documentation/computers/getting-started.html#raspberry-pi-imager)
    - Click CHOOSE OS.
    - Select Other gerneral-purpose OS.
    - Select Ubuntu.
    - Select Ubuntu Server 22.04.5 LTS (64-bit) that support RPi 3/4/400.
    - Click CHOOSE STORAGE and select the micro SD card.
    - Click WRITE to install the Ubuntu.

Before removing the memory card, let's set up network access. One way to do this is by using netplan, which is already enabled by default.

To edit the file, navigate to the location where the SD card is mounted (for example /media/USERNAME) and use the following command:
```
sudo nano /writable/etc/netplan/50-cloud-init.yaml
```

I'll provide two examples of configuration for this file. The first one is a way to configure using TTLS tunneling with NO certificate, and the second one for a "home" connection.

Example 1: TTLS tunneling with NO certificate:
```
# Example of NETPLAN configuration for TTLS network and NO certificate.
# Edit the yaml file inside the /etc/netplan folder and paste this text, replacing the network configurations.
# It's extremely important to maintain the indentation as it is here.
network:
    ethernets:
        eth0:
            dhcp4: true
            optional: true
    version: 2
    wifis:
        wlan0:
            dhcp4: yes
            dhcp6: yes
            access-points:
                 SSID_Name:
                     auth:
                       key-management: eap
                       method: ttls
                       identity: "username"
                       password: "password"
```

Example 2: "Home" connection:
```
# Default configuration for "home" networks (Default WPA(2)-PSK configuration with password and SSID)
# Edit the yaml file inside the /etc/netplan folder and paste this text, replacing the network configurations.
# It's extremely important to maintain the indentation as it is here.
network:
    ethernets:
        eth0:
            dhcp4: true
            optional: true
    version: 2
    wifis:
        wlan0:
            dhcp4: yes
            dhcp6: yes
            access-points:
                "ssid_name":
                     password: "ssid_password"
```


Configuração do RaspBerry PI 4
- Iniciar a RaspBerry com o sistema gravado no cartão
    - Usuário: ubuntu
    - Senha Padrão: ubuntu (ele pedirá para repetir a senha e escolher uma nova)


- Desabilitar a atualização automática
    É interessante não permitir a atualização automática, então para desabilitar utilize o comando abaixo e altere o conteúdo como mostrado a seguir:

Raspberry Pi 4 Configuration

- Start the Raspberry Pi with the system recorded on the card
    - Username: ubuntu
    - Default Password: ubuntu (it will ask to repeat the password and choose a new one)
- Disable automatic updates
    It's advisable to disable automatic updates. To do so, use the command below and change the content as shown:
```
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```
```
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
```

- Disable PowerSave for Wlan0 (if not disabled, the network "dies" after a few minutes)
  To disable it, you need to edit the service configuration file with the following command:
```
sudo systemctl --full --force edit wifi_powersave@.service
```
On the blank screen of the editor, insert the following text:
```
[Unit]
Description=Set WiFi power save %i
After=sys-subsystem-net-devices-wlan0.device

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/sbin/iw dev wlan0 set power_save %i

[Install]
WantedBy=sys-subsystem-net-devices-wlan0.device
```

Now adjust how you want the WiFi PowerSave to start at boot. If you want PowerSave to be ON, use the code:
```
sudo systemctl disable wifi_powersave@off.service
sudo systemctl enable wifi_powersave@on.service
```

If you want PowerSave to be OFF (exactly what we want), use these commands:
```
sudo systemctl disable wifi_powersave@on.service
sudo systemctl enable wifi_powersave@off.service
```

Remember that it won't be necessary to repeat these commands; they will only be executed once or if you want to change the PowerSave status. Now simply restart.
```
sudo reboot
```
Para verificar se tudo esta como deveria estar, use o seguinte comando:
```
iw dev wlan0 get power_save
```

The expected response is:
Power save: off
If "Power save: ON" appears, simply repeat the commands below and check again.
```
sudo systemctl disable wifi_powersave@on.service
sudo systemctl enable wifi_powersave@off.service
```

Now you'll install ROS2 Humble Hawksbill from the instructions in [the official ROS2 Humble instalation guide](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html).

After installing ROS2 and restarting the Raspberry Pi, let's install the specific packages for TurtleBot:
```
sudo apt install python3-argcomplete python3-colcon-common-extensions libboost-system-dev build-essential
sudo apt install ros-humble-hls-lfcd-lds-driver
sudo apt install ros-humble-turtlebot3-msgs
sudo apt install ros-humble-dynamixel-sdk
sudo apt install libudev-dev
mkdir -p ~/turtlebot3_ws/src && cd ~/turtlebot3_ws/src
git clone -b humble-devel https://github.com/ROBOTIS-GIT/turtlebot3.git
git clone -b ros2-devel https://github.com/ROBOTIS-GIT/ld08_driver.git
cd ~/turtlebot3_ws/src/turtlebot3
rm -r turtlebot3_cartographer turtlebot3_navigation2
cd ~/turtlebot3_ws/
echo 'source /opt/ros/humble/setup.bash' >> ~/.bashrc
source ~/.bashrc
colcon build --symlink-install --parallel-workers 1
echo 'source ~/turtlebot3_ws/install/setup.bash' >> ~/.bashrc
source ~/.bashrc
```

Now it's time to install or configure the LDS. TurtleBots3 purchased since the beginning of 2022 come with the LDS-02, while earlier versions come with the LDS-01. If you use the LDS-01, you don't need to install the driver or update the TurtleBot3 package. So, only execute the commands below if you use the LDS-02!

```
sudo apt update
sudo apt install libudev-dev
cd ~/turtlebot3_ws/src
git clone -b ros2-devel https://github.com/ROBOTIS-GIT/ld08_driver.git
cd ~/turtlebot3_ws/src/turtlebot3 && git pull
rm -r turtlebot3_cartographer turtlebot3_navigation2
cd ~/turtlebot3_ws && colcon build --symlink-install
```

USB Port Setting for OpenCR
```
sudo cp `ros2 pkg prefix turtlebot3_bringup`/share/turtlebot3_bringup/script/99-turtlebot3-cdc.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Now let's export the model of the LDS you use to the .bashrc. If you use the LDS-02 model, replace LDS-01 with LDS-02 in the text below:
```
echo 'export LDS_MODEL=LDS-01' >> ~/.bashrc
source ~/.bashrc
```

Finally, let's specify the model of the Turtlebot3. You should replace "burger" with your model (burger, waffle, or waffle_pi) and set the ROS_DOMAIN_ID:
```
echo 'export TURTLEBOT3_MODEL=burger' >> ~/.bashrc
echo 'export ROS_DOMAIN_ID=30 #TURTLEBOT3' >> ~/.bashrc
source ~/.bashrc
```

Before configuring the OpenCR, I perform a system update, and there's an interesting fact: either during the update or if automatic hibernation is disabled, the USBs will stop working. So, you just need to switch the kernel to the one I'll specify in the codes below or a newer one:
```
apt list linux-image*raspi
```

This command above will list all kernels for the Raspberry Pi. Now, you just need to choose one and install it with the command below:

```
sudo apt install --install-recommends linux-image-5.4.0-1047-raspi
sudo apt update
sudo apt upgrade
```

- OpenCR Configuration
At this moment, you should connect the OpenCR to the Raspberry Pi using the USB cable.
Let's proceed with the procedure to update the OpenCR.

Installation of packages for firmware update:
```
sudo dpkg --add-architecture armhf
sudo apt update
sudo apt install libc6:armhf
```

Now change the model of your TurtleBot3 before running the code:
```
export OPENCR_PORT=/dev/ttyACM0
export OPENCR_MODEL=burger
rm -rf ./opencr_update.tar.bz2
```

Download the updated firmware package:
```
wget https://github.com/ROBOTIS-GIT/OpenCR-Binaries/raw/master/turtlebot3/ROS2/latest/opencr_update.tar.bz2
tar -xvf ./opencr_update.tar.bz2
```

Perform the firmware update of the OpenCR:
```
cd ~/opencr_update
./update.sh $OPENCR_PORT $OPENCR_MODEL.opencr
```
You can verify if everything is okay by performing teleoperation on the Raspberry Pi (Turtlebot3) and moving the wheels using the keyboard.

First, find out the IP address of your Turtlebot (using ifconfig wlan0) and access it via SSH:
ssh ubuntu@YOUR_IP_ADDRESS

Inside the terminal, perform the robot bringup:
``` 
ros2 launch turtlebot3_bringup robot.launch.py
```

If everything went well, the last sentence will end with: Run!

Now, in a new terminal, execute the Teleop:
``` 
ros2 run turtlebot3_teleop teleop_keyboard
```

At this point, you should be happy with the motors working.


Now, let's move on to the bigger challenge, getting the Intel RealSense to work on Humble!!!!
- Install PIP:
```
sudo apt-get install python3-pip
```
- Install NumPy
```
pip install numpy
``` 
- Install OpenCV
```
sudo apt-get install libopencv-dev
pip install opencv-contrib-python
```
To install the RealSense drivers for ROS2 Humble, simply follow this GitHub repository:
```
[https://github.com/intel/ros2_intel_realsense](https://github.com/IntelRealSense/realsense-ros)
```

# Remote PC Setup
For the preparation of the Remote PC, there aren't many procedures. You just need a computer with Ubuntu 22.04 installed (in the case of Ros2 Humble) and start the installation of ROS2 Humble with the following [the oficioal instalation guide.](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html)


Install Dependent ROS 2 Packages:
```
sudo apt install ros-humble-gazebo-*
sudo apt install ros-humble-cartographer
sudo apt install ros-humble-cartographer-ros
sudo apt install ros-humble-navigation2
sudo apt install ros-humble-nav2-bringup
```

Install TurtleBot3 Packages:
```
source ~/.bashrc
sudo apt install ros-humble-dynamixel-sdk
sudo apt install ros-humble-turtlebot3-msgs
sudo apt install ros-humble-turtlebot33
```
Environment Configuration:
```
echo 'export ROS_DOMAIN_ID=30 #TURTLEBOT3' >> ~/.bashrc
source ~/.bashrc
```


I hope this step-by-step guide helps you to perform the ROS1 to ROS2 upgrade on your robot or TurtleBot.

Best Regards,
Rogerio B. Cuenca.


