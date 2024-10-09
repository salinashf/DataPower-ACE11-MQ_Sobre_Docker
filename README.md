# DataPower-ACE11-MQ_Sobre_Docker
Este repositorio contiene la implementación de Datapower ACE11 y MQ tecnologías de IBM sobre Docker

¡¡Buen día, con todos!!

Para iniciar se buscará tener los tres servidores(aplicaciones) sobre una misma red (misma LAN)

- DataPower
- ACE11/IBM App Connect Enterprise
- IBM WebSphere MQ

Se debe de crear una red LAN propia para poder comunicar  los CONTENEDORES 
>Los datos de nombre de red e ip son libres de cambiarse 
```sh
docker network create --driver bridge --subnet 192.20.0.0/24 --gateway 192.20.0.1 networkDevicesIBM_Apps
```


Como siguiente paso  procedemos a bajar las  imágenes bases de  [Docke Hub](https://hub.docker.com/)

| Contenedor | Comando  |
| ------ | ------ |
| Datapower | [docker pull sankeerth520/datapower] |
| Datapower Alternativo | [docker pull juliopari/datapower:10.0] |
| IBM MQ y ACE | [docker pull ibmcom/ace-mq:latest] |


>Para las imágenes de Datapower se pudieses bajar de los repositorios
>oficiales de IBM, las imágenes propuestas son imágenes que en 
>su momento fueron publicadas en docker hub y posteriormente retiraron.
>Son imágenes que se bajaron , las empaquetaron  con otro nombre y se volvió a subir a docker hub, en docker hub existen más  copias !! 

> Para las app  de IBM MQ y ACE esta en una sola imagen


# CONTENEDORES
## IBM MQ y ACE

Se iniciara creando las carpetas para que se persistan los datos de IBM ACE y IBM MQ 

Crear las carpetas siguientes, 
sobre las  misma  ruta donde está el archivo docker-compose.yml



		
	└───docker-compose.yml
	└───volumens
		├───ace
		│   └───aceuser
		│   	├───ace-server
		│   	├───initial-config
		│   	└───log	
		└───mqm
			└───mnt

El docker-compose.yml para IBM ACE-MQ


```sh
version: "3.9"
networks:
    networkDevicesIBM_Apps:
        external: true
services:
  ACE_MQ-ServerLocal:
    container_name: ace_mq_local
    environment:
      ACE_SERVER_NAME: ACE_MQ_LOCAL
      USE_QMGR: 'true'
      MQ_QMGR_NAME: QMGR
      LOG_FORMAT: 'json'
      LICENSE: accept
      LANG: es
      MQ_ENABLE_METRICS: 'true'
      ACE_ENABLE_METRICS: 'true'
    image: ibmcom/ace-mq:latest
    privileged: true
    ports:
    - 7600:7600
    - 7688:7688
    - 7800:7800
    - 7843:7843
    - 1414:1414
    - 2800:2800
    - 2801:2801
    - 2803:2803
    volumes:
      - ./volumens/ace/aceuser/initial-config:/home/aceuser/initial-config
      - ./volumens/ace/aceuser/ace-server:/home/aceuser/ace-server
      - ./volumens/ace/aceuser/log:/home/aceuser/log
      - ./volumens/mqm/mnt:/mnt/mqm 
    hostname: device_ACE_MQ_Local
    networks:
      networkDevicesIBM_Apps:
        ipv4_address: 192.20.0.40

```     
>Se utiliza la red creada inicialmente  [ networkDevicesIBM_Apps ] 

>Se le asigna una ip que este en el mismo segmento de red

>Se carga las carpetas creadas para que persista sobre las mismas

>Los demás nombre son arbitrarios de acuerdo a su necesidad




## IBM DataPower

Se iniciara creando las carpetas para que se persistan los datos de IBM DataPower

Crear las carpetas siguientes, sobre las  misma  ruta donde está el archivo docker-compose.yml

		
	└───docker-compose.yml
	└───volumens
		├───apply
		│   ├───config
		│   └───local
		└───default
			├───config
			└───local

El docker-compose.yml para IBM DataPower
```sh
version: "3.9"
networks:
    networkDevicesIBM_Apps:
        external: true
services:
  datapowerDevLocalServer:
    container_name: datapowerDevLocal
    image: ibmcom/datapower
    privileged: true
    environment:
      DATAPOWER_ACCEPT_LICENSE: 'true'
      DATAPOWER_INTERACTIVE: 'true'
      DATAPOWER_LOG_STDOUT: 'true'
      DATAPOWER_FAST_STARTUP: 'true'
      DATAPOWER_WORKER_THREADS: 4
      DP_WEB_MGMT: 'true'
    ports:
      - 9099:9099   
      - 8447:8447
      - 8088:8088
    volumes:
      - ./volumens/default/config:/drouter/config
      - ./volumens/default/local:/drouter/local     
      - ./volumens/apply/config:/opt/ibm/datapower/drouter/config
      - ./volumens/apply/local:/opt/ibm/datapower/drouter/local
    tty: true
    stdin_open: true   
    deploy:
      resources:
        limits:
          cpus: "10"
    hostname: deviceDataPowerLocal
    networks:
      networkDevicesIBM_Apps:
        ipv4_address: 192.20.0.25



```
>Se utiliza la red creada inicialmente  [ networkDevicesIBM_Apps ] 

>Se le asigna una ip que este en el mismo segmento de red

>Se carga las carpetas creadas para que persista sobre las mismas

>Se carga los cpu que pudiese consumir, esto se hace para poder un poco  acelerar el datapower

>Los demás nombre son arbitrarios de acuerdo a su necesidad



## Habilitar dominio  
### Se habilitara el dominio una sola vez 

La primera vez que se ejecutó el contenedor con el comando   
>[docker compose up]

se debe ingresar y ejecutar   y habilitar el  dominio.

En otra terminal ingresar al datapower  con el comando  
>docker attach datapowerDevLocal	



| Descripción | Comando |
| ------ | ------ |
|Ingrese la palabra admin, está ingresando el nombre de usuario  |admin |
|Ingrese la palabra admin, está ingresado la contraseña del usuario  |admin  |
|Ingrese a modo de configuración de terminal |configure terminal|
|Ingrese el comando, para entrar habilitar consola web | web-mgmt|
|ingrese el comando, para habilitar la consola web |admin-state "enabled"|
|Ingrese el comando, para configurar el puerto del consola web | local-address "eth0_ipv4_1" "9099"|
|ingrese el comando, para guardar cambios  | idle 0|
|ingrese el comando , para salir | exit|


>Recuerde que el habilitar el dominio y la consola web es una sola vez

>Una ves terminado  ya se puede acceder a datapower desde consola  web 
```sh
	https://localhost:9099/dp/login.xml
```