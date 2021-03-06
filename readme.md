# Wireless Named Data Networking Over LoRa
This repository contains the project and work done to add a LoRa networking face to the Named Data Networking protocol based fowarding software [NFD](https://github.com/named-data/NFD).

## Installation & Usage
### Hardware and Libraries
This code was modified for and tested on the following hardware and software:
- Raspberry Pi 3B+
- Libelium SX1272 LoRa Shield (old) https://www.cooking-hacks.com/sx1272-lora-shield-for-raspberry-pi-900-mhz.html 
- Modtronix inAir9B SX1276 LoRa module (new) https://modtronix.com/product/inair9b/ 

The libraries used are as follows:
- [ndn-cxx](https://github.com/named-data/ndn-cxx) - C++ Library that implements NDN programming functions. Used by NFD.
- [ArduPi](https://www.cooking-hacks.com/documentation/tutorials/extreme-range-lora-sx1272-module-shield-arduino-raspberry-pi-intel-galileo/index.html#step4) SX1272 Library for Raspberry Pi
- [SX1276](https://github.com/ahmedawad1/SX1276-LoRa-driver) Register changes to library for SX1276 LoRa module from SX1272 

Software:
- Raspbian 10 Buster
- Clang 9
- [NFD](https://github.com/named-data/NFD) - NDN forwarding software.
- [NDN-Tools](https://github.com/named-data/ndn-tools) - Simple command-line tools for testing NFD and NDN applications.

### Software Setup
Prior knowledge of Linux, the command line, C++, Python, compilation, and installation is assumed. This setup is based off of [Dr. Ashiqur Rahman's Guide](https://gist.github.com/ashiqopu/547f357345db396fdf3e36e6ed462cbe). See his guide for a more in depth instructions.

Raspbian and Rasbian Lite were both tested working for this project. Raspbian Lite is recommended for the lighter memory usage.

After installing Raspbian Lite on the Raspberry Pi, Clang 9 must be installed. [Here](https://solarianprogrammer.com/2018/04/22/raspberry-pi-raspbian-install-clang-compile-cpp-17-programs/) is an article about installing Clang 9 on the Pi. Newer versions have not been tested with this project (but will likely work).


### Compiling & Installation
#### First Time Compilation
These build and installation instructions are based off of [this tutorial](https://named-data.net/doc/NFD/current/INSTALL.html) from Named-Data.net. See the tutorial for more in depth instructions.

Install the dependencies first:
```
sudo apt-get install build-essential pkg-config libboost-all-dev \
                     libsqlite3-dev libssl-dev libpcap-dev
                     
sudo apt-get install doxygen graphviz python-sphinx

```

Download the git files:
```
git clone --recursive https://github.com/kevinchau94/NDN_over_LoRa.git
```
Open the Directory:
```
cd NDN_over_LoRa
```	

#### Compile the NDN Directory
Navigate to the NDN_over_loRa folder.
Compile and install the modified library in the `ndn-cxx` folder first from the command line:
```
cd ndn-cxx
sudo chmod u+x ./waf
CXX=clang++ ./waf configure
./waf -j2
sudo ./waf install
sudo ldconfig
```

#### Compile the NFD Directory
Next, compile and install the modified library in the `NFD` folder:
```
cd ../NFD
sudo chmod u+x ./waf
CXX=clang++ ./waf configure
```
At this point if it says that something is missing (websocket app) then follow the instructions to add it, the instructions will be:

NOTE - skip to "Copy the modified NFD configuration..." if you do not see that websockets are missing in the configuration process.
```
mkdir -p websocketpp
curl -L https://github.com/cawka/websocketpp/archive/0.8.1-hotfix.tar.gz > websocketpp.tar.gz
tar xf websocketpp.tar.gz -C websocketpp/ --strip 1
```
Then rerun the config command above and continue:
```
./waf -j2
sudo ./waf install
```
Copy the modified NFD configuration file from the root directory to the correct location:
```
sudo cp nfd.conf /usr/local/etc/ndn/nfd.conf
```

#### Compile the NDN-Tools Directory
Finally, compile and install ndn-tools. The version provided in the `ndn-tools` folder has not been modified so a new version may be installed from [https://github.com/named-data/ndn-tools](https://github.com/named-data/ndn-tools) by following the instructions posted there. However the `ndn-tools` folder contains a tested working version and can be compiled and installed as follows:
```
cd ../ndn-tools
sudo chmod u+x ./waf
CXX=clang++ ./waf configure
./waf -j2
sudo ./waf install
```

`CXX=clang++` is used to siginify to [waf](https://waf.io/) (a build system) to use the clang++ compiler.

The -j2 flag is used to limit the number of simultaneous compilations to 2 jobs. This is done to reduce memory usage. Successful compilation has been achieved with 3 jobs, but has also needed to be limited to 1 job depending on what other software is running on the Pi at the time of compilation.

#### LoRa Configuration
The following commands are to compile the lora configuration commands. The LoRa takes its setting values from a text file, and these commands will alter the text file or read it.
```
cd ../NFD/lora_libs/setup/
clang++ getLoraSetup.cpp -o getlora
clang++ setuplora.cpp -o setlora
clang++ resetlora.cpp -o resetlora
sudo cp -r getlora setlora resetlora /usr/local/bin
sudo cp -r 'LoRa Configuration' /usr/local
sudo chmod a+w+r /usr/local/'LoRa Configuration'
sudo chmod a+w+r /usr/local/'LoRa Configuration'/lora_config.txt
```
You can now run the commands from anywhere via the command console.
```
getlora
setlora
resetlora
```
#### Updating Directory
These series of commands allows you to pull the latest updates from Github onto your Pi.
If you have old/local changes on your Pi, you can reset those changes with the following command:
```
git reset --hard 
```
and grab the Github repository's updates with 
```
git pull https://github.com/kevinchau94/NDN_over_LoRa.git
```
this will reset the permission for ./waf so set the permissions back with
```
chmod u+x ./waf
```
and then update the rest of NFD with a recompile and install. (use cd command if you are not in the NFD folder already)
```
cd NFD 
./waf -j2
sudo ./waf install
```
You can do the following for ndn-cxx, ndn-tools, or any other folder as necessary.

#### Testing GPIO (Optional)
The following compiles the test GPIO. 
```
cd test
clang++ -pthread *.cpp -o testGPIO
```
You can move it to the bin directory if you want to be able to run the command from anywhere:
```
sudo cp -r testGPIO /usr/local/bin
```
Run the command with either of the following: (the first one requires you to be in the test folder and the second requires the executable to be in the /bin folder.
DO NOT have anything wired to the Pi while this is being done!
```
sudo ./testGPIO
testGPIO
```
#### Susbsequent Compilations
Once everything has been compiled for the first time and tested working, if any changes are made the compilation and installation will go much faster and requires fewer steps.

If ndn-cxx is modified:
```
cd ndn-cxx
./waf -j2
sudo ./waf install
sudo ldconfig
```

If NFD is modified:
```
cd NFD
./waf -j2
sudo ./waf install
```


### Running and testing the software
Make sure the SX1272/SX1276 module is connected to the Raspberry Pi. Make sure the Pi is off while connecting the module.

Test out NFD locally or with a face other than LoRa first.
Creating a face and connecting to other NFD daemons can be found at [Getting Started with NFD](https://named-data.net/doc/NFD/current/INSTALL.html) & [NDN-tools testing](https://github.com/named-data/ndn-tools).

#### Starting and Stopping NFD
```
sudo NFD -start

sudo NFD -stop
```
#### Creating a LoRa Face
2 or more Raspberry Pis with LoRa modules will need to be set up for this test.

Create a LoRa face on each Pi:

*When creating a face, the format follows the format of: current_node_id-connected_node_id.
The node_id can be any unique number.
Ex. connect node 1 to node 2:

```
nfdc face create lora://1-2
```
Look for the face id
Message should appear as face created id:<faceid>
  
```
nfdc route add /ndn <faceid>
```
The route added (/ndn) can be any route name

Add another face if you want to make a connection to another node.

Follow the same instructions for the next node with the appropriate ids.

To make a broadcast/multicast face use the following command: (face id can be a number other than 1)

```
nfdc face create lora://1
```

#### Testing
Using ndn-tools:

On one Pi, produce a packet:
```
echo "test" | ndnpoke -x 1000 /ndn 
```
-x specifies the freshness of the packet (time is in ms)

/ndn is the name to produce the packet on

On the second Pi,
```
ndnpeek -pf -l 10000
```
The -l flag specifies the timeout period.

producing a packet as such:
```
echo "test2" | ndnpoke -ux 1000 /ndn
```
-u indicates that packet can be sent unsolicited, or without receiving an interest packet. 


### Configuration
The LoRa module hardware is setup in NFD/daemon/face/lora-factory.cpp in the `setup()` function.

## Files & Modifications
Modified/Added files:
- ndn-cxx/ndn-cxx/net/face-uri.cpp
- ndn-cxx/ndn-cxx/net/face-uri.hpp
- ndn-tools/tools/peek/ndnpoke/ndnpoke.cpp
- ndn-tools/tools/peek/ndnpoke/ndnpoke.hpp
- NFD/daemon/face/lora-transport.cpp
- NFD/daemon/face/lora-transport.hpp
- NFD/daemon/face/lora-channel.cpp
- NFD/daemon/face/lora-channel.hpp
- NFD/daemon/face/lora-factory.cpp
- NFD/daemon/face/lora-factory.hpp
- NFD/lora_libs/libraries/arduPiLoRa/arduPiLoRa.cpp
- NFD/lora_libs/libraries/arduPiLoRa/arduPiLoRa.hpp
- NFD/lora_libs/arduPi/arduPi.cpp
- NFD/lora_libs/arduPi/arduPi.h
- ndnScripts/*
- nfd.conf
- NFD/lora_libs/setup/setuplora.cpp
- NFD/lora_libs/setup/getLoraSetup.cpp
- NFD/lora_libs/setup/resetlora.cpp
- NFD/lora_libs/setup/'LoRa Configuration'

## Authors, Contributors, & Resources
[Named Data software and libraries](https://github.com/named-data) were modified by:
- Sean Hammond
- Jake Maschoff
- Bryan Hatasaka
- Katlynne Bills
- Garrick Clegg
- Jason Stauffer
- Thomas Van Hook
- Kevin Chau
- Camden Ewell
- Ye Zhou
- Jack Fahey

Advisors, Mentors, Support, Thanks
- Stephen Dudley
- Mingyue Ji
- Jon Davies
- L3Harris Technologies
- University of Utah Electrical and Computer Engineering department

Resources:
- https://named-data.net/
- https://gist.github.com/ashiqopu/547f357345db396fdf3e36e6ed462cbe
- https://www.cooking-hacks.com/documentation/tutorials/extreme-range-lora-sx1272-module-shield-arduino-raspberry-pi-intel-galileo/index.html
- https://www.raspberrypi.org/
- https://github.com/named-data/NFD
- https://github.com/named-data/ndn-cxx
- https://github.com/named-data/ndn-tools
- https://github.com/ahmedawad1/SX1276-LoRa-driver


Big thanks goes to all contributers of the [Named Data Netwoking Project](https://named-data.net/)!
