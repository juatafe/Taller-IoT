# Taller-IoT 
Introducción a IoT con un simulador Rasbian y una placa NodeMCU Lolin V3
-
![Setup ](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/TallerIoT.png)
 Con este taller se pretende que el alumnado sea capaz de preparar un entorno de desarrollo simulando una Raspberry Pi conectada a un hardware real como el NodeMCU mediante una red WiFi. Mediante Node-Red se programa de forma visual los diferentes módulos que permiten la comunicación. Mediante el IDE de Arduino se carga en la tarjeta NodeMCU un programa capaz de comunicarse con la Raspbian. 
 En primer lugar se prepara el entorno tanto en la Raspbian como en la tarjeta NodeMCU. Se realizan dos ejecercicios sencillos por separado y finalmente se implementa la comunicación entre ambos. 
 
## Material necesario
* Máquina virtual Raspbian con Virtual Box
  * 1024 MB de RAM
  * Disco 25GB
* Placa NodeMCU Lolin v3 ESP8288 12E v0.9
* Cable micro-USB a USB-A
* Tarjeta WiFi o USB WiFi. Para este caso USB EDIMAX EW-7811Un V2
* PC con un mínimo 4G de RAM i puertos USB-2.0
* Router WiFi (El própio teléfono móvil)
## Raspbian
### Descargar e instalar ISO 
A la [página oficial de Raspberry encontramos la ISO](https://www.raspberrypi.com/software/raspberry-pi-desktop/)  en concreto de la versión [Debian 10 Buster con el kernel 4.19](https://downloads.raspberrypi.org/rpd_x86/images/) que es con la que se ha preparado este taller. Para ello se prepara una máquina virtual en Virtual Box con Extension Pack del tipo Debian 64 bits. La instalación se realiza en modo gráfico y usando toda la partición dinámica de 25GB.
Tras la instalación se actualiza mediante los comandos:
```
sudo apt update && sudo apt full-upgrade
```
Más adelante se requiere de conectividad con el puerto USB así que se instala Guest Additions. Se debe insertar el CD mediante el menú Devices--> Insert Guest Additions CD image..

![Devices--> Insert Guest Additions CD image](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/InsertGuestAdditionsCDImage.png)

Tras esta acció aparecerá montado el CD. Se procede a lanzae el script de instalación mediante el comando:
```
bash media/cdrom0/VBoxLinuxAdditions.run
```
### 
