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
. Dado que en la Raspberry Pi real se requiere de su instalación (aquí se puede encontrar cómo) y en la virtual no existe versión para i386 se procede a crear un scrip con el mismo nombre del comando que no recibe parámetros y que devuelve un valor de temperatura. Este escrip se situa en el directorio /bin de la Rasbian con el contenido siguiente:
```
#!/bin/bash
echo “temp=60.7'C”
```
Así es posible configurar el módulo de igual forma que ser haría con el hardware real. 

![vcgencmd](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/vcgencmd.png)

* Para que el contenido sea válido se requiere cortar el mensaje con el dato de temperatura. Para ello se requiere del módulo function.

![function](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/function.png) ![function2](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/function2.png)

* Para representar el dato se require del módulo gauge contenido en el paquete node-red-dashboard. Para instalarlo es posible mediante comandos con ```npm install node-red-dashboard```  o mediante Node-RED en el menú Manage palette(alt+mays+p)-->Install . Se recomienda este último que no requiere reiniciar Node-RED.
* Con el módulo gauge en el tablero se modifican sus propiedades. Se requiere un tab, se requiere un group (desmarca Display group name) y selecciona Add. Finalmente se configura el gauge node con una temperatura de 0 a 120 y sectores de 40 y 80 grados. 
 
 ![gauge](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/gauge.png)
 
 * Se conectan las salidas con las entradas como en la siguiente figura:
 ![Flow1](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/Flow1.png)
 
 * Mediante Deploy se guarda el tablero(Flow) y empieza la acción. 
 * Para acceder al Dashboard creado y ver el gauge se requiere acceder al socket IP-DE-LA-RASPBIAN:1880/ui
 
 ![Home](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/Home.png)
 
 # Configurar NodeMCU
 ## Instalar IDE de Arduino
 * Se descarga mediante el comando ``` wget https://downloads.arduino.cc/arduino-1.8.19-linux32.tar.xz ```
 * Se descomprime y se instala mediante los siguientes comandos:
 ```
tar -xvf arduino-1.8.19-linux32.tar.xz
cd arduino-1.8.19
sudo bash arduino-linux-setup.sh root
sudo su
bash install.sh
exit
sudo usermod -a -G dialout pi
echo "SUBSYSTEM==\"usb\", MODE=\"0660\", GROUP=\"$(id -gn)\"" | sudo tee /etc/udev/rules.d/00-usb-permissions.rules udevadm control--reload-rules
reboot

 ```
 ### Para evitar problemas en detectar el dispositivo NodeMCU
 --- 
 * En caso de que el anfitrión sea Windows se requiere de instalar el driver. Los siguientes enlaces puden resultar de utilidad para ello:
   [CH341SER_Windows](https://github.com/nodemcu/nodemcu-devkit/blob/master/Drivers/CH341SER_WINDOWS.zip
) y [CH340-Drivers-tutorial] (https://learn.sparkfun.com/tutorials/how-to-install-ch340-drivers/all
)
* En el caso de Linux se requiere el abconnector. Para la versión i386 se obtienen con el comando ```wget http://www.arduinoblocks.com/web/site/abconnectordownload/2 ``` y se instalan con el comando ```sudo dpkg -i abconnector_4_x86.deb ```
* Para arrancar se requiere /opt/abconnector/abconnector se arranca una vez y ya no se necesita. Recomienda reiniciar Raspbian.
--- 
## Abrir el IDE de Arduino
![Arduino](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/Arduino.png)
* Poner la librería ESP8266 en el Ide de arduino en File--->Preferences
![Preferences]()
[https://arduino.esp8266.com/stable/package_esp8266com_index.json](https://arduino.esp8266.com/stable/package_esp8266com_index.json
)

![Preferences-link](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/Preferences-link.png)

* Entrando en Tools-->Board-->Board Manager

![toolsBoard](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/ToolsBoard.png)

* Se busca e instala la placa ESP8266

![esp8266](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/esp8266.png)

* Se selecciona la placa NodeMCU 1.0 (ESP-12E Module)

![ESp-12E](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/ESP-12E.png)

* Se introduce el micro-USB en la placa NoceMCU y el USB al puerto del PC, con ello se conecta la placa NodeMCU al PC anfitrión.

![conectaNode](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/conectaNode.jpg)

*  Para conectar la placa NodeMCU a la máquina virtual se selecciona Devices-->USB-->QinHeng ELectronics USB serial.

![qingen](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/qingen.jpg)

* Tras la conexión se requiere seleccionar el puerto en Tools -->Ports.

![port](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/port.jpg)



 ## Segundo Ejercicio 
### Hacer que el LED del NodeMCU parpadee
Este ejerccio pone de manifiesto que se tiene la posibilidad de programar el NodeMCU correctamente y todo está preparado para realizar el ejercicio final. 
* Con el NodeMCU conectado por USB al PC y a la máquina virtual Raspbian arrancar el IDE de Arduino y abrir un ejemplo en File-->Examples-->ESP8266-->Blink
* Mediante el icono Upload se carga el firmware en el NodeMCU. El led incorporado debería parpadear tras la descarga del programa.

![blink](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/blink.jpg)

## Tercer Ejercicio
### Encender el LED de la Placa NodeMCU con Node-RED en Raspbian
* Para el envio de mensajes entre el NodeRED y el NodeMCU se requiere un Broker MQTT(Mosca) que se instala con el menú Manage palette o con el comando ```npm install node-red-contrib-mqtt-broker```
* El escenario consiste en el gestor de mensajes, un interruptor y una salida de mensajería MQTT. Como puede verse en la siguiente imagen el módulo Aedes MQTT broker(gestor de mensajes) está conectado a dos módulos debugers para mostrar por terminal mensajes de depuración. El resto són el módulo del panel dashboard switch y el módulo del panel network mqtt out. 
![flow2](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/flow2.png)

* La configuración de los módulos resulta bastante intuitiva y se omite. El switch se configura de forma que se envia un string On Payload "off" y "on" Off Payload. El tópic, que es la cabecera del mensaje que se envía se establece en room/lamp simulando que es la luz de una habitación. 
![LED](https://github.com/juatafe/Taller-IoT/blob/main/imagenes/LED.png)
