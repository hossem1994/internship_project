# internship_project 

# Used HW SW 

- dev and test server : Docker on Vmware ESi
- Operation server : 4 cores 16G with public ip address with Docker
- Gateway: Raspberry Pi 3 with RFM95 @868.1 MHz
- devices : Arduino Uno R3 DHT22 humidity and temperature sensor 

# Getting started with LoraWAN, loraserver.io
    
  ## Device setup
  .Loading the existing code RedMineAtIpnet/Fiware/cayennlpp.ino into the DHT22 sensor. In order to transmit date to the fiware iot        agent the payload has to be encoded with the Cayenne Low Power Payload (Cayenne LPP) \ 
  .To learn more about the Cayennelpp data model see https://mydevices.com/cayenne/docs/lora/#lora-cayenne-low-power-payload\ \
  .To see similar project visit https://github.com/sabas1080/CayenneLPP/blob/master/examples/WeatherStation_LMIC/WeatherStation_LMIC.ino 
  ## Gateway setup RPI Gateway
  .activation of SPI see reference\
  .command line : $ sudo raspi-config Navigate to Interface Options -> SPI -> enable it\
  .Next comes downloading the WiringPi library: $ sudo apt-get install wiringpi\
  .We need git, too: $ sudo apt-get install git\
  .On the LoRa software We use the single-channel packet redirector, tftelkamp / single_chan_pkt_fwd. It's a little c / cpp program to      listen to in order to communicate with a transceiver Semtech SX1276, listing  LoRa paquets and transmitting it\
  $ git clone https://github.com/tftelkamp/single_chan_pkt_fwd.git \
  $ cd single_chan_pkt_fwd \
  $ nano main.cpp (//changing the server address from the TTN address to the lora gateway bridge host address)\
  $ make \
  $ ./single_chan_pkt_fwd 

  # Loraserver's implementation
  ## LoRa gateway bridge
### Requirements
•	Mosquitto MQTT broker: In order to install it, I executed the following command: sudo apt-get install mosquitto\
•	PostgreSQL database: LoRa Server persists the gateway data into a PostgreSQL database. To install the latest PostgreSQL:\
wget –quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - sudo echo "deb http://apt.postgresql.org/pub/repos/apt/ jessie, trusty or xenial-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list sudo apt-get update sudo apt-get install postgresql-9.6\\
•	Redis database: LoRa Server stores all non-persistent data into a Redis datastore. Installation on Debian / Ubuntu: sudo apt-get\ install redis-server
### Installation of LoRa Gateway Bridge:
 It’s done simply by executing the following command that creates a configuration file is located at /etc/lora-gateway-bridge/lora-gatewaybridge.toml.\
sudo apt-get install lora-gateway-bridge
### Configuration
•	Gateway: Modify the packet-forwarder of the gateway so that it will send its data to the LoRa Gateway Bridge. serveraddress to the IP address / hostname of the LoRa Gateway Bridge servportup to 1700 (the default port that LoRa Gateway Bridge is using) servportdown to 1700 (same)\
•	The configuration file: entering the packet forwarder’s address which is the same as gateway’s.
## LoRa network server
Having the same requirements as the gateway bridge, LoRa Server needs its own database.
### Postgres database creation
1.	Starting the prompt: sudo -u postgres psql
2.	Creating a user: create role loraserverNs with login password ’dbpassword’;
3.	Creating a database create database loraserverNs with owner loraserverNs;
### Installing the Network Server
sudo apt-get install loraserver 

## LoRa application server
Creating a user, database and a PostgreSQL pqtrgm extension After starting the prompt we entered the following commands:
1.	Creating a user: create role loraserverAs with login password ’dbpassword’;
2.	Creating a database create database loraserverAs with owner loraserverAs;
3.	Changing to the LoRa App Server database: l,oraserverAs
4.	Enabling the extension: create extension pgtrgm;
### Installing the Application Server
sudo apt-get install lora-app-server\
After installation, modify the configuration file which is located at /etc/lora-app-server/loraapp-server.toml
![image](https://user-images.githubusercontent.com/40475940/450751-58465500-b0df-11e8-8ea1-e66ab24ee17c.png)\
Given that the password that I used when creating the PostgreSQL database is ’dbpassword’, the config variable postgresql.dsn had to be changed into:\
 ![image](https://user-images.githubusercontent.com/40475940/45075556-62685380-b0df-11e8-89ac-cdb7233a2713.png)\
Another thing that has to be changed in the name of the node in the uplink topic subsciption. In pour case the node is called 'node' so the code in the configuration file has to look like the screenshot below\
![image](https://user-images.githubusercontent.com/40475940/45075613-9a6f9680-b0df-11e8-922a-f68276c667a7.png)\

## Security in a LoRaWAN network

In terms of security, the LoRANWAN standard specifies three AES-128 keys:\
NwkSKey, network session key, which uses exchanges between the terminal and the core network. It ensures the authenticity of devices in calculation and by checking the integrity code of the message, MIC, from the Header cloth and the Payload.\
AppSKey, Application Session Key, specific to end-device is used to encrypt and decrypt the payload \
AppKey, application key known only by the application and the final device and which allows to deduce the two previous keys.

![1](https://user-images.githubusercontent.com/42407959/45169205-61c5df00-b1fd-11e8-94fd-2f88fa602e37.png)

The application session key is only known by the application provider. It is impossible for a third party, including the operator, to consult the data. \

=> We have generated DevEUI ,NwKSKey , AppSKey in Loraserver by opening this link https://34.243.110.137:8080/ then login and password admin admin :\

<img width="791" alt="default" src="https://user-images.githubusercontent.com/42407959/45393475-56731900-b62c-11e8-9efd-56839a91e757.PNG">

<img width="782" alt="uu" src="https://user-images.githubusercontent.com/42407959/45393476-570baf80-b62c-11e8-9ad2-7c833d89eba5.PNG">


A frame counter is also used to protect against replay attacks, which would consist of repeating a maliciously intercepted transmission. The counter is incremented with each transmission. The gateway and the end devices reject transmissions whose counter value is less than that present.

 =>   To be part of a LoRaWAN network, each device must obtain both session keys. This activation step can be done in two ways: Over-The-Air Activation (OTAA) or Activation By Personalization (APB).
 


## Securing Lora by generating certificates

Configuration to generate certificates for the LoRa Server project : \
Go installation \
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz \
export PATH=$PATH:/usr/local/go/bin \

CFSSL installation  \
For generating the certificates, cfssl is being used \
go get -u github.com/cloudflare/cfssl/cmd/cfssl \
go get -u github.com/cloudflare/cfssl/cmd/cfssljson \
go get -u github.com/cloudflare/cfssl/cmd/... \ 

![image](https://user-images.githubusercontent.com/42407959/45265383-50bddd80-b44a-11e8-8aab-e4998dd4d65c.png)

![image](https://user-images.githubusercontent.com/42407959/45265387-603d2680-b44a-11e8-8419-7dd7ae024839.png)

![image](https://user-images.githubusercontent.com/42407959/45265390-66330780-b44a-11e8-95be-d78185ac7f55.png)

#### LoRa Server
LoRa Server is an open-source network-server, part of the LoRa Server project\
 Configuration file : loraserver.toml \
(addition of the generated certives + modification of the ports + generation of random bytes for the jwt_secret) \

![image](https://user-images.githubusercontent.com/42407959/45265400-9b3f5a00-b44a-11e8-864b-a448c71cc351.png)
#### LoRa App Server
LoRa App Server is an open-source application-server, part of the LoRa Server project
Configuration file : lora-app-server.toml 

![image](https://user-images.githubusercontent.com/42407959/45265414-c629ae00-b44a-11e8-9019-e72d459a6704.png)

![image](https://user-images.githubusercontent.com/42407959/45265425-d0e44300-b44a-11e8-8fce-5126cb8e2a96.png)


#### LoRa Gateway Bridge
LoRa Gateway Bridge abstracts the packet_forwarder protocol into JSON over MQTT \
Configuration file : lora-gateway-bridge.toml \

![image](https://user-images.githubusercontent.com/42407959/45265433-ebb6b780-b44a-11e8-9461-537637bbc378.png)

![image](https://user-images.githubusercontent.com/42407959/45265435-f2ddc580-b44a-11e8-92f5-a2e27a3656c7.png)


# Fiware installation 
## Prerequisites :
### DOCKER AND DOCKER COMPOSE

To keep things simple all components will be run using Docker. Docker is a container technology which allows to different components isolated into their respective environments

We used it on “Linux” environment

•	First of all, we have to install Docker on Linux. follow the instructions here : https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce-1


It is to note that Fiware is an existant platform and we are not the ones that developed it. Fiware is known by a unique and complex architecture that is based on its own components. We have tried different approached in Fiware’s installation and decided to use a project that has been worked on by Fiware’s partner ATOS. The installation and configuration steps are presented below:\
1.	Clone the repository with the following command:

git clone https://github.com/Atos-Research-and-Innovation/IoTagent-LoRaWAN.git

2.	Once the repository is cloned, you have to download the dependencies for the project. To do so, from the root folder of the project execute:

npm install

3.	If you ever want to test the IoT Agent or make sure that it runs correctly, you can run it with the default configuration by executing the following command

node bin/iotagent-lora

The bootstrap process should finish with:

info: Loading devices from registry\
info: LoRaWAN IoT Agent started

4.	Check that the IoTA is running correctly:

curl -v http://localhost:4061/iot/about

The result must be similar to:

{"libVersion":"2.6.0-next","port":4061,"baseRoot":"/"}

5.	To run orion manually for test purposes, you can run the following commands

o	Manually, run MongoDB on another container for the data to be stored in it. Then, run orion while linking it to the already running MongoDB docker:

sudo docker run --name mongodb -d mongo:3.2\
sudo docker build -t orion .\
sudo docker run -d --name orion1 --link mongodb:mongodb -p 1026:1026 orion -dbhost mongodb.

o	Specify where to find your MongoDB host:

sudo docker build -t orion .\
sudo docker run -d --name orion1 -p 1026:1026 orion -dbhost <MongoDB Host>.\
Check that everything is correctly working with the command below
 
            curl localhost:1026/version
            
PS: The parameter -t orion in the docker build command gives the image a name.

### Docker-compose
For simplicity reasons, you can also run all the dockers in one go by using docker-compose. \
You can configure as many containers as you want, how they should be built and connected, and where data should be stored. \
A docker-compose.yml file\
A docker-compose.yml file is a YAML file that defines how Docker containers should behave in production.

#### docker-compose.yml

You can run a single command to build, run, and configure all of the containers.This will be done by running the following command:\
docker-compose -f docker/docker-compose.yml up


The following figure presents the docker-compose.yml file with which we can run all the dockers containing fiware’s components with a single command:

![doker-compose1](https://user-images.githubusercontent.com/42373973/45123462-43ad9f80-b167-11e8-8a6a-5540c3d2a7a4.png)
![doker-compose2](https://user-images.githubusercontent.com/42373973/45123640-da7a5c00-b167-11e8-944c-edba7d381227.png)

This docker-compose.yml file tells Docker to do the following:

•	Pull the image mongodb from mongo 3.2, the image orion from fiware/orion:latest and the image iotagent-lora from ioeari/iotagent-lora.\
•	Immediately restart containers if one fails.\
•	Map port 1026 on the host to web’s port 1026.\
•	Map port 4061 on the host to web’s port 4061.



### The application server’s web interface
LoRa application server comes with a user-web-interface that enables the management of the different components of the network and that allows the decoding and the visualization of the received data. The following steps lead to the configuration of this interface.
1. Configuring the bind address TLS certificates and the authentication key
 ![image](https://user-images.githubusercontent.com/40475940/45075625-a5c2c200-b0df-11e8-805f-cc11b345d4bc.png)

2. Launching the gateway bridge\
![image](https://user-images.githubusercontent.com/40475940/45075637-af4c2a00-b0df-11e8-82fd-da18d0967bed.png)
 
3.	Launching the loRa network server
 ![image](https://user-images.githubusercontent.com/40475940/45075651-b96e2880-b0df-11e8-84d7-f0e7f515852b.png)

4.	Launching LoRa application server \
![image](https://user-images.githubusercontent.com/40475940/45075660-c428bd80-b0df-11e8-8d71-d42141ab525f.png)

5.	Access the configured bind address, in our case it’s: https://52.211.159.35:8080
 ![image](https://user-images.githubusercontent.com/40475940/45075667-cc80f880-b0df-11e8-8422-65cfbbba8e78.png)

6.	In there, the used gateway’s as well as the device’s information have to be provided, respectively in the ’gateway profiles’ and ’device profiles’ sections showed in the figure above. The used network server with which the application server will deal must be entered in the ’network servers’ section. And in the ’Applications’ section, an application using the mentioned gateway and devices can be made. In the application we can choose to decode using a self developed javascript code or using CayenneLPP schema. In our case the latter one was chosen.
## Device provisioning and data’s circulation visualization
1.	Launching the packet forwarder\
![image](https://user-images.githubusercontent.com/40475940/45075779-2681be00-b0e0-11e8-957e-5156ca38856f.png) \

And here are the values that are being sent by the node\
 ![image](https://user-images.githubusercontent.com/40475940/45075797-36010700-b0e0-11e8-8ca3-761139ec500c.png)

2.	Encoded data will be then forwarded to the gateway bridge\
![image](https://user-images.githubusercontent.com/40475940/45075813-40230580-b0e0-11e8-9ddb-e83fe50a05ae.png)

3.	In the application server’s interface, we can visualize frames that are being received by the gateway\
![image](https://user-images.githubusercontent.com/40475940/45075856-58932000-b0e0-11e8-91ba-69f091aaabc6.png)

4.	Device provisioning: for the data to be forwarded to fiware’s IoT agent through an MQTT broker, the device has to be provisioned and that is by: entering the attributes which, in our case, are ’temperature’ and ’humidity’ as well as providing loraserver’s information, the device’s eui, application eui, id , key and the data model which is Cayennelpp.\
 ![image](https://user-images.githubusercontent.com/40475940/45075865-621c8800-b0e0-11e8-98b7-011fbf402e1c.png)\

In the next figure, is the IoT agent processing the provisioning, connecting to the MQTT broker, starting the application and then subscribing to the application’s topic\
 ![image](https://user-images.githubusercontent.com/40475940/45075891-6fd20d80-b0e0-11e8-84b9-5b523ae94c08.png)\

In the figure below, the received and decoded data in the application server is shown\
 ![image](https://user-images.githubusercontent.com/40475940/45075907-7ceefc80-b0e0-11e8-82ab-66aead4702f9.png)\
 
In the following figure, data that has been forwarded to the IoT agent is shown\
![image](https://user-images.githubusercontent.com/40475940/45075928-89735500-b0e0-11e8-983d-e452b3452483.png)\



And below, is data stored in to context broker’s database after being translated to NGSI by the IoT agent\
![image](https://user-images.githubusercontent.com/40475940/45075935-942dea00-b0e0-11e8-81a8-e1613486f5a7.png)

 



# DATA VISUALISATION
In order to control Data integrity we should visualise them in different steps using MongoDB to know if the totality of the data are sent 

=> we enter the IP address 34.254.184.237


![image](https://user-images.githubusercontent.com/42407959/45264951-b0b18580-b444-11e8-8961-08c93862b3ad.png)

and the file ppk

![image](https://user-images.githubusercontent.com/42407959/45264952-c030ce80-b444-11e8-8229-a97cc536af13.png)

![image](https://user-images.githubusercontent.com/42407959/45264954-cc1c9080-b444-11e8-908c-6c8d54afb380.png)


![image](https://user-images.githubusercontent.com/42407959/45264959-d8085280-b444-11e8-8e91-3eba68ade9c3.png)

![image](https://user-images.githubusercontent.com/42407959/45264960-e2c2e780-b444-11e8-84cb-93458574ef66.png)

![image](https://user-images.githubusercontent.com/42407959/45265318-39322500-b449-11e8-85af-0d83f8b4cbcd.png)

Open another terminal\
=> Monitor Iotagentlora database

![image](https://user-images.githubusercontent.com/42407959/45265326-5ebf2e80-b449-11e8-9658-9d2d99dde3f4.png)

monitor Orion database (orion-test)

![image](https://user-images.githubusercontent.com/42407959/45265334-93cb8100-b449-11e8-9520-d5a3a74c6dfc.png)
![image](https://user-images.githubusercontent.com/42407959/45265336-a5148d80-b449-11e8-8587-a26f84c2d38b.png)

## References:
 .LoRa: https://www.lora-alliance.org \
 .LoRa Semtech: http://www.semtech.com/wireless-rf/internet-of-things/ \
 .LoRaWAN Tutorial: http://www.instructables.com/id/Use-Lora-Shield-and-RPi-to-Build-a-LoRaWAN-Gateway/ \
 .Raspbian OS: https://www.raspberrypi.org/downloads/raspbian/ \
 .Single Channel Package Forwarder: https://github.com/tftelkamp/single_chan_pkt_fwd/ \
 .Duty Cycle of LoRa: https://www.thethingsnetwork.org/wiki/LoRaWAN/Duty-Cycle \
 .RFM95 assembly with lora gateway: https://learn.adafruit.com/adafruit-rfm69hcw-and-rfm96-rfm95-rfm98-lora-packet-padio-breakouts/assembly \
 .See also http://cpham.perso.univ-pau.fr/LORA/RPIgateway.html


 
