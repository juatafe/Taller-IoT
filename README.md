# Taller-IoT 
Introducción a IoT con un simulador Rasbian y una placa NodeMCU Lolin V3
-
![Setup ](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/TallerIoT.png)
 Con este taller se pretende que el alumnado sea capaz de preparar un entorno de desarrollo simulando una Raspberry Pi conectada a un hardware real como el NodeMCU mediante una red WiFi. Mediante Node-RED se programa de forma visual los diferentes módulos que permiten la comunicación. Mediante el IDE de Arduino se carga en la tarjeta NodeMCU un programa capaz de comunicarse con la Raspbian. 
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
En la [página oficial de Raspberry encontramos la ISO](https://www.raspberrypi.com/software/raspberry-pi-desktop/)  en concreto de la versión [Debian 10 Buster con el kernel 4.19](https://downloads.raspberrypi.org/rpd_x86/images/) que es con la que se ha preparado este taller. Para ello se prepara una máquina virtual en Virtual Box con Extension Pack del tipo Debian 64 bits. La instalación se realiza en modo gráfico y usando toda la partición dinámica de 25GB.
Tras la instalación se actualiza mediante los comandos:
```
sudo apt update && sudo apt full-upgrade
```
Más adelante se requiere de conectividad con el puerto USB así que se instala Guest Additions. Se debe insertar el CD mediante el menú Devices--> Insert Guest Additions CD image..

![Devices--> Insert Guest Additions CD image](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/InsertGuestAdditionsCDImage.png)

Tras esta acción aparecerá montado el CD. Se procede a lanzar el script de instalación mediante el comando:
```
bash media/cdrom0/VBoxLinuxAdditions.run
```
### Instalación de Node-RED
Aquí surge la problemática que estamos en un entorno virtual para i386. Raspberry pi lleva ARMs y los paquetes disponibles para Debian Buster de nodejs no pasan de la versión 10.24. El procedimiento pasa por tratar de engañar al instalador para que instale una versión superior. Para ello se requieren una serie de dependencias. Como aptitude gestiona mejor las dependencias que apt se procede a su instalación con el comando:
```
sudo apt install aptitude
```
* Se instala con aptitude las siguientes dependencias:
```
sudo aptitude install libc6:amd64
sudo aptitude install libgcc1:amd64
sudo aptitude install libstdc++6:amd64
```
* Para la siguiente y última dependencia se requiere una de las soluciones que presenta aptitude que instale la dependencia a pesar que desinstala muchos paquetes de python3. A priori para este taller no se requieren por tanto se procede con la opción numérica que instale el paquete "r XX" donde XX corresponde a dicha opción. 
```
sudo aptitude install python3-minimal:amd64
```
* Una vez instaladas la dependencias se procede a comentar todos los repositorios del fichero /etc/apt/sources.list
* Se realiza un update con ``` sudo apt update ```
* Se procede a descargar un scrip que permite añadir la fuente para la instalación de nodejs con el siguiente comando:
```
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo bash -
```
* Se procede a instalar nodejs mediante el comando:
```
sudo apt-get  install -y nodejs
```
* En este punto se debe comprobar las versiones de nodejs y nmp mediante ``` node -v ``` y ``` npm -v ``` que para este taller se han instalado v16.14.2 y 8.5.0 respectivamente. 
* Se procede a descomentar el fichero /etc/apt/sources.list y se ejecuta el comando ``` sudo apt update```
* Se instala Node-red mediante el comando:
```
npm install node-red
```
* Ahora es posible arrancar Node-RED mediante el comando:
```
sudo node ~/node_modules/node-red/red.js --max-old-space-size=256
```
* En el PC anfitrión se abre el navegador y se introduce el socket formado con la IP-DE-LA-RASPBIAN:1880 que permite ver el panel de control de Node-RED

![Wellcome to Node-RED 2.2](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/wellcomeNodeRed.png)

## Primer Ejercicio 
### Obtener temperatura de la CPU y mostrarla en un panel de control
* En el apartado common se encuentra el módulo inject. Este módulo permite introducir una especie de reloj que con el que ocurren los sucesos. Con doble clik se abre la pestaña de propiedades y se seleccionan los siguientes parámetros:
 * Inject once after 1 second
 * Repeat interval
 * every 1 minutes
 
![inject](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/inject.png)

* En el aparatado function se encuentra el módulo exec donde permite introducir comandos.

![exec](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/exec.png)

* Para emular que se trabaja con una Raspberry Pi real se utiliza el mismo comando que permite medir la temperaura de la CPU. ```vcgencmd measure_temp```
. Dado que en la Raspberry Pi real se requiere de su instalación (aquí se puede encontrar cómo) y en la virtual no existe versión para i386 se procede a crear un scrip con el mismo nombre del comando que no recibe parámetros y que devuelve un valor de temperatura. 
