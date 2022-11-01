# Turtlebot3 com ROS2 (Dashing no Robo e Foxy no Computador)
Projeto Feito no Lab 404 do Inpser
Colaboradores: Rogério e Lícia

A escolha de utilizar o Ros2 Dashing no Robo se deu pela melhor aceitação dos drivers da RealSense D415i e a escolha do Ros2 Foxy no computador foi para manter a compatibilidade dos atuais robôs com Ros1, sendo assim um SSD híbrido com ROS1, ROS2 e Docker do Bebop2.

Material Utilizado:
- Raspbery PI 4 
- TurtleBot 3 Burger
    - RaspBerry PI 4
    - OpenCR
- Realsense D400

Sistemas
Computador: 
- Ubuntu 20.04 LTS
- ROS2 Foxy Fitzroy
Rasp 4:
- Ubuntu 18.04 server para RaspBerry Pi4
- ROS2 Dashing Diademata

Sites:
Robotis (E-Manual): https://emanual.robotis.com/docs/en/platform/turtlebot3/sbc_setup/
Ubuntu 18.04 para Raspi4 arm64: http://old-releases.ubuntu.com/releases/18.04.4/ubuntu-18.04.4-preinstalled-server-arm64+raspi4.img.xz
ROS2 Dashing: https://docs.ros.org/en/dashing/Installation.html
Ubunu 20.04 LTS Desktop: https://releases.ubuntu.com/focal/
ROS2 Foxy: https://docs.ros.org/en/foxy/Installation.html
ROS2 Wraper Intel Realsense: https://github.com/intel/ros2_intel_realsense



Configuração do RaspBerry PI 4
- Utilizar o "Disks" do linux para abrir o arquivo de imagem pré-instalada do Ubuntu server 18.04 para Raspi4
- Iniciar a RaspBerry com o sistema gravado no cartão
    - Usuário: ubuntu
    - Senha Padrão: ubuntu (ele pedirá para repetir a senha e escolher uma nova)
- Configurar a rede por NETPLAN:
Deixarei aqui duas configurações, sendo a primeira para rede TTLS com acesso corporativo e o segundo para redes "domésticas".
Configuração para Rede com TTLS, usuário e senha (Sem CA).
```
# Exemplo de configuracao NETPLAN com rede "tuledada" TTLS e SEM Certificado.
# Editar o arquivo yaml dentro da pasta /etc/netplan e colar este texto substituindo as configuracoes de rede.
# Extremamente importante manter a tabulacao como esta aqui.
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
```
# Configuracao padrao para redes "domesticas" (Configuracao padrao WPA(2)-PSK com senha e ssid)
# Editar o arquivo yaml dentro da pasta /etc/netplan e colar este texto substituindo as configuracoes de rede.
# Extremamente importante manter a tabulacao como esta aqui.
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
- Desabilitar a atualização automática
    É interessante não permitir a atualização automática, então para desabilitar utilize o comando abaixo e altere o conteúdo como mostrado a seguir:

```
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```
```
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
```

- Desabilitar o PowerSave do Wlan0 (caso nao esteja desabilitado a rede "morre" apos alguns minutos)
    - Para desabilitar voce precisa editar o arquivo de configuracao do servico com o comando abaixo: 
```
sudo systemctl --full --force edit wifi_powersave@.service
```
Na tela vazia do editor,  coloque o texto abaixo:
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
Agora ajuste como quer que o PowerSave do WiFi inicie no boot. Se quiser que o PowerSave esteja LIGADO use o codigo:
```
sudo systemctl disable wifi_powersave@off.service
sudo systemctl enable wifi_powersave@on.service
```
Se quiser que o PowerSave esteja DESLIGADO (exatamente o que queremos) use estes comandos:
```
sudo systemctl disable wifi_powersave@on.service
sudo systemctl enable wifi_powersave@off.service
```
Lembrando que nao sera necessario repetir estes comandos, apenas serao executados uma vez ou se quiser mudar o status do PowerSave.
Agora basta reiniciar
```
sudo reboot
```
Para verificar se tudo esta como deveria estar, use o seguinte comando:
```
iw dev wlan0 get power_save
```
A resposat esperada e: 
Power save: off
Se acontecer se aparecer: "Poser save: ON" basta repetir os comandos abaixo e verificar novamente.
```
sudo systemctl disable wifi_powersave@on.service
sudo systemctl enable wifi_powersave@off.service
```

Agora chegou a hora de instalar ou configurar o LDS. Os TurtleBots3 comprados desde o inicio de 2022 estao com o LDS-02 os anteriores com o LDS-01. Caso voce use o LDS-01 nao precisa fazer a instalacao do driver e atualizacao do pacote do TurtleBot3. Entao so execute os comandos abaixo se voce usa o LDS-02!!!
```
sudo apt update
sudo apt install libudev-dev
cd ~/turtlebot3_ws/src
git clone -b ros2-devel https://github.com/ROBOTIS-GIT/ld08_driver.git
cd ~/turtlebot3_ws/src/turtlebot3 && git pull
rm -r turtlebot3_cartographer turtlebot3_navigation2
cd ~/turtlebot3_ws && colcon build --symlink-install
```
Agora vamos exportar para o .bashrc o modelo do LDS que voce usa. Se voce usa o modelo LDS-02 substitua, no texto abaixo, o LDS-01 pelo LDS-02:
```
echo 'export LDS_MODEL=LDS-01' >> ~/.bashrc
source ~/.bashrc
```
Por ultimo vamos dizer qual o modelo do Turtlebot3 ele eh. Voce deve substituir o burger pelo seu modelo (burger, waffle ou waffle_pi):
```
echo 'export TURTLEBOT3_MODEL=burger' >> ~/.bashrc
source ~/.bashrc
```

- Configuracao do OpenCR
Neste momento voce deve conectar o OpenCR ao Rasp com o cabo USB
Vamos ao procecimento para atualizar o OpenCR

Instalacao dos pacotes para atualizacao do firmware:
```
sudo dpkg --add-architecture armhf
sudo apt update
sudo apt install libc6:armhf
```
Agora troque o modelo do seu TurtleBot3 antes de rodar o codigo:
```
export OPENCR_PORT=/dev/ttyACM0
export OPENCR_MODEL=burger
rm -rf ./opencr_update.tar.bz2
```
Fazer o Download do pacote do firmware atualizado:
```
wget https://github.com/ROBOTIS-GIT/OpenCR-Binaries/raw/master/turtlebot3/ROS2/latest/opencr_update.tar.bz2
tar -xvf ./opencr_update.tar.bz2
```
Realizar a atualizacao do firmware do OpenCR:
```
cd ~/opencr_update
./update.sh $OPENCR_PORT $OPENCR_MODEL.opencr
```
Voce pode verificar se tudo esta ok realizando, no RASP (no Turtlebot3) o teleop (teleoperacao) e mover as rodas com o teclado.

Veja qual o IP do seu turtlebot (ifconig wlan0) e acesse via ssh.
ssh ubuntu@SEU_ENDERECO_IP
Lembrando que o usuario eh ubuntu e a senha eh turtlebot.

Dentro do terminal realize o bringup do robo:
``` 
ros2 launch turtlebot3_bringup robot.launch.py
```
Se tudo deu certo, a ultima frase terminara com: Run!

Agora em um novo terminal execute o Teleop:
``` 
ros2 run turtlebot3_teleop teleop_keyboard
```
Neste ponto voce deve estar feliz com os motores funcioando.

Agora vamos para o desafio maior, fazer a Intel RealSense funcionar no Foxy!!!!
- Instalar o PIP:
```
sudo apt-get install python3-pip
```
- Instlar NumPy
```
pip install numpy
``` 
- Instalar OpenCV
```
sudo apt-get install libopencv-dev
pip install opencv-contrib-python
```
Para instar os drivers da RealSense para o ROS2 Dashing, basta seguir este git:
```
https://github.com/intel/ros2_intel_realsense
```

# Configuracoes do PC REMOTO
    Para a preparação do PC Remoto, não há muitos procedimentos, basta o computador ter instalado um Ubuntu 20.04 (no caso do Ros2 Foxy) e iniciar a instlação do ROS2 Foxy com os seguinte comandos:
    
```    
wget https://raw.githubusercontent.com/ROBOTIS-GIT/robotis_tools/master/install_ros2_foxy.sh
sudo chmod 755 ./install_ros2_foxy.sh
bash ./install_ros2_foxy.sh
```

Agora a instalação das dependências (Gazebo 11, Cartographer e Navigation2):
```
sudo apt-get install ros-foxy-gazebo-*
sudo apt install ros-foxy-cartographer
sudo apt install ros-foxy-cartographer-ros
sudo apt install ros-foxy-navigation2
sudo apt install ros-foxy-nav2-bringup
```
Instalação dos pacotes do Turtlebot:

```
source ~/.bashrc
sudo apt install ros-foxy-dynamixel-sdk
sudo apt install ros-foxy-turtlebot3-msgs
sudo apt install ros-foxy-turtlebot3
```
Preparando o ambiente:
```
echo 'export ROS_DOMAIN_ID=30 #TURTLEBOT3' >> ~/.bashrc
source ~/.bashrc
```

Com este tutorial o Turtlebot3 Burger com a Realsense funcionaram perfeitamente.

Segue abaixo um link para migrar códigos do ROS1 para o ROS2.



# Links para portar do ROS1 para ROS2
https://docs.ros.org/en/foxy/Contributing/Migration-Guide-Python.html


