Projeto Feito no Lab 404 do Inpser
Colaboradores: Rogério e Lícia

Material Utilizado:
- Raspbery PI 4 
- TurtleBot 3 Waffle_Pi
    - RaspBerry PI 4
    - OpenCR
    - OpenMannipulator

Sistemas
Computador: 
- Ubuntu 20.04 LTS
- ROS2 Foxy Fitzroy
Rasp 4:
- Ubuntu 20.04 para o Turtlebot3
- ROS2 Foxy Fitzroy

Sites:
Robotis (E-Manual): https://emanual.robotis.com/docs/en/platform/turtlebot3/sbc_setup/
ROS2 Foxy: https://docs.ros.org/en/foxy/Installation.html


Configuração do RaspBerry PI 4
- Imagem baixado do site da robotis.com: https://www.robotis.com/service/download.php?no=2064
- Gravar Imagem no cartão com o Disk (Ubuntu 20.04) ou Imager (Rasp)
- Iniciar a RaspBerry com o sistema gravado no cartão
    - Usuário: ubuntu
    - Senha Padrão: turtlebot
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
Por ultimo vamos dizer qual o modelo do Turtlebot3 ele eh. Voce deve substituir o waffle_pi pelo seu modelo (burger, waffle ou waffle_pi):
```
echo 'export TURTLEBOT3_MODEL=waffle_pi' >> ~/.bashrc
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
tar -xjf ./opencr_update.tar.bz2
```
Realizar a atualizacao do firmware do OpenCR:
```
cd ~/opencr_update
./update.sh $OPENCR_PORT $OPENCR_MODEL.opencr
```


- Configuracoes do PC REMOTO
    - 
