This guide provides step-by-step instructions on how to deploy an end-to-end 5G SA network using open-source software solutions and COTS hardware.
# Table of Contents 

- [Table of Contents](#table-of-contents)
- [1. Deployment Scenario](#1-deployment-scenario)
  - [1.1 OAI minimum requirements](#11-oai-minimum-requirements)
  - [1.2. Our UWaterloo testbed configuration](#12-our-uwaterloo-testbed-configuration)
- [2.OAI CN5G](#2oai-cn5g)
  - [2.1 OAI CN5G pre-requisites](#21-oai-cn5g-pre-requisites)
  - [2.2 OAI CN5G configuration files](#22-oai-cn5g-configuration-files)
  - [2.3 Pull OAI CN5G docker images](#23-pull-oai-cn5g-docker-images)
  - [2.4 Start OAI CN5G](#24-start-oai-cn5g)
  - [2.5 Stop OAI CN5G](#25-stop-oai-cn5g)
  - [2.6 OAI CN5G configuration](#26-oai-cn5g-configuration)
  - [2.7 Add an ISIM to the OAI CN5G database](#27-add-an-isim-to-the-oai-cn5g-database)
- [3. ISIM configuration](#3-isim-configuration)
  - [3.1 SIM programming and configuration](#31-sim-programming-and-configuration)
  - [3.2 SUPI concealment](#32-supi-concealment)
- [4. OAI 5G SA gNB](#4-oai-5g-sa-gnb)
  - [4.1 Build UHD from source](#41-build-uhd-from-source)
  - [4.2 USRP x310 Setup](#42-usrp-x310-setup)
    - [4.2.1 Prerequisites](#421-prerequisites)
    - [4.2.2 Check Python installation](#422-check-python-installation)
    - [4.2.3 Configure firewall to allow communication with USRP](#423-configure-firewall-to-allow-communication-with-usrp)
  - [4.3 Installing GNUradio from source (Optional/not required for OAI)](#43-installing-gnuradio-from-source-optionalnot-required-for-oai)
    - [4.3.1. Enabling python support for GNUradio](#431-enabling-python-support-for-gnuradio)
  - [4.4 Starting and testing the USRP](#44-starting-and-testing-the-usrp)
- [5. OAI 5G SA gNB](#5-oai-5g-sa-gnb)
  - [5.1 Build OAI gNB](#51-build-oai-gnb)
  - [5.2 OAI gNB configuration](#52-oai-gnb-configuration)
- [6. Setup and configure COTS UE](#6-setup-and-configure-cots-ue)
  - [6.1 Setup COTS UE](#61-setup-cots-ue)
  - [6.2 Setup OAI UE](#62-setup-oai-ue)
- [7. Deploy the 5G SA end-to-end network](#7-deploy-the-5g-sa-end-to-end-network)
  - [7.1 Run OAI CN5G](#71-run-oai-cn5g)
  - [7.2 Run OAI gNB](#72-run-oai-gnb)
    - [7.2.1 USRP X310](#721-usrp-x310)
    - [7.2.2 RFSIMULATOR](#722-rfsimulator)
  - [7.3 Connect COTS UE](#73-connect-cots-ue)
    - [7.3.1 Connect COTS UE](#731-connect-cots-ue)
    - [7.3.2 Connect Simulated UE](#732-connect-sim-ue)
- [8. Troubleshooting, debugging and advanced configurations](#8-troubleshooting-debugging-and-advanced-configurations)
  - [8.1 Debugging UHD](#81-debugging-uhd)
  - [8.2 Network not visible on the UE](#82-network-not-visible-on-the-ue)
  - [8.3 No internet on UE (masq does not work)](#83-no-internet-on-ue-masq-does-not-work)
  - [8.4 USRP N300 and X300 Ethernet Tuning](#84-usrp-n300-and-x300-ethernet-tuning)
  - [Authors](#authors)
  - [License](#license)
  - [Acknowledgments](#acknowledgments)


This guide provides step-by-step instructions on how to deploy an end-to-end 5G SA network using open-source software solutions and COTS hardware

# <a name='Deployment Scenario'></a>1. Deployment Scenario
In this tutorial we describe how to configure and run a 5G end-to-end setup with OAI CN5G, OAI gNB and COTS UE.

## <a name='OAIminimumrequirements'></a>1.1 OAI minimum requirements
- Laptop/Desktop/Server for OAI CN5G and OAI gNB
    - Operating System: [Ubuntu 22.04 LTS](https://releases.ubuntu.com/22.04/ubuntu-22.04.1-desktop-amd64.iso)
    - CPU: 8 cores x86_64 @ 3.5 GHz
    - RAM: 32 GB
- Laptop for UE
    - Operating System: Microsoft Windows 10 x64
    - CPU: 4 cores x86_64
    - RAM: 8 GB
    - Windows driver for Quectel MUST be equal or higher than version **2.2.4**
- [USRP B210](https://www.ettus.com/all-products/ub210-kit/), [USRP N300](https://www.ettus.com/all-products/USRP-N300/) or [USRP X300](https://www.ettus.com/all-products/x300-kit/)
    - Please identify the network interface(s) on which the USRP is connected and update the gNB configuration file
- Quectel RM500Q
    - Module, M.2 to USB adapter, antennas and SIM card
    - Firmware version of Quectel MUST be equal or higher than **RM500QGLABR11A06M4G**


## <a name='OurUWaterlootestbedconfiguration'></a>1.2. Our UWaterloo testbed configuration
- 3 Desktops for OAI CN5G, OAI gNB and simulated UEs
    - Operating System: [Ubuntu 22.04 LTS](https://releases.ubuntu.com/22.04/ubuntu-22.04.1-desktop-amd64.iso)
    - CPU: i9-10980XE CPU @ 3.5 GHz
    - RAM: 32 GB
- [USRP X310](https://www.ettus.com/all-products/x310-kit/)
    - Please identify the network interface(s) on which the USRP is connected and update the gNB configuration file   
- Google Pixel 7 PRO UE
    - Android 13 Tiramisu
    - 5G Sub-6: Bands n1/2/3/5/7/8/12/20/25/28/30/38/40/41/48/66/71/77/78
    - Sysmocomm sysmoISIM-SJA2 programmable SIM

# <a name='OAI CN5G'></a>2.OAI CN5G

## <a name='OAICN5Gpre-requisites'></a>2.1 OAI CN5G pre-requisites

Install and configure the OAI 5G core pre-requisites as follows

```bash
sudo apt install -y git net-tools putty

# https://docs.docker.com/engine/install/ubuntu/
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add your username to the docker group, otherwise you will have to run in sudo mode.
sudo usermod -a -G docker $(whoami)
reboot
```

## <a name='OAICN5Gconfigurationfiles'></a>2.2 OAI CN5G configuration files

Download and copy configuration files:

```bash
wget -O ~/oai-cn5g.zip https://gitlab.eurecom.fr/oai/openairinterface5g/-/archive/develop/openairinterface5g-develop.zip?path=doc/tutorial_resources/oai-cn5g
unzip ~/oai-cn5g.zip
mv ~/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g/doc/tutorial_resources/oai-cn5g ~/oai-cn5g
rm -r ~/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g ~/oai-cn5g.zip
```

## <a name='PullOAICN5Gdockerimages'></a>2.3 Pull OAI CN5G docker images

```bash
docker pull mysql:8.0
docker pull oaisoftwarealliance/oai-amf:develop
docker pull oaisoftwarealliance/oai-nrf:develop
docker pull oaisoftwarealliance/oai-smf:develop
docker pull oaisoftwarealliance/oai-udr:develop
docker pull oaisoftwarealliance/oai-udm:develop
docker pull oaisoftwarealliance/oai-ausf:develop
docker pull oaisoftwarealliance/oai-spgwu-tiny:develop
docker pull oaisoftwarealliance/trf-gen-cn5g:latest
docker build --target ims --tag asterisk-ims:latest --file ~/oai-cn5g/Dockerfile .
```

## <a name='StartOAICN5G'></a>2.4 Start OAI CN5G
```bash
cd ~/oai-cn5g
docker compose up -d
```

## <a name='StopOAICN5G'></a>2.5 Stop OAI CN5G
```bash
cd ~/oai-cn5g
docker compose down
```
## <a name='OAICN5Gconfiguration'></a>2.6 OAI CN5G configuration

```bash
# Access the core configuration files
cd ~/oai-cn5g
cd docker-compose/conf
sudo nano <conf_file>
```
To configure the core correctly the following steps need to be taken:

 * In the AMF configuration file, the NGAP addr, MCC, MNC, TAC and S-NSSAI (SST and SD) should be the same as the one seen in the gNB configuration file.
 * In the rest of the core functions configuration files, the correct AMF addr needs to be configured and should be the same as the one seen in the gNB configuration file.
 * The APN/DNN needs to be configured so that it is the same as the one present in the UE and set it to IPv4, not IPv4/6 or IPv6.
 * The ISIM credentials need to be added to the list of subscribers through the oai_db2.sql database file.

## <a name='AddanISIMtotheOAICN5Gdatabase'></a>2.7 Add an ISIM to the OAI CN5G database

After configuring the ISIM, you need to register its ISIM credentials to the list of subscribers through the oai_db2.sql database:

```bash
# Access the database file
cd ~/oai-cn5g
cd docker-compose/database/
sudo nano oai_db2.sql
```

Example command of an OAI UE ENTRY in oai_db2.sql: 

```bash
INSERT INTO `AuthenticationSubscription` (`ueid`, `authenticationMethod`, `encPermanentKey`, `protectionParameterId`, `sequenceNumber`, `authenticationManagementField`, `algorithmId`, `encOpcKey`, `encTopcKey`, `vectorGenerationInHss`, `n5gcAuthMethod`, `rgAuthenticationInd`, `supi`) VALUES
('001010000060591', '5G_AKA', '18E8DE5BA04A8D910B010D298672BCE0', '18E8DE5BA04A8D910B010D298672BCE0', '{\"sqn\": \"000000000020\", \"sqnScheme\": \"NON_TIME_BASED\", \"lastIndexes\": {\"ausf\": 0}}', '8000', 'milenage', 'B71C0607B0A5AE6E3657FEE846579991', NULL, NULL, NULL, NULL, '001010000060591');
```

# <a name='ISIM'></a>3. ISIM configuration
We are using a sysmoISIM-SJA2 ISIM (5G-enabled) in this example tutorial.
Sysmocom SIM user manual: https://sysmocom.de/manuals/sysmousim-manual.pdf

## <a name='SIMprogrammingandconfiguration'></a>3.1 SIM programming and configuration 

Download `pySim <https://github.com/osmocom/pysim>`_ : 

```bash

    git clone https://github.com/osmocom/pysim
    cd pysim
    sudo apt-get install --no-install-recommends \
    	pcscd libpcsclite-dev \
    	python3 \
    	python3-setuptools \
    	python3-pyscard \
    	python3-pip
    pip3 install -r requirements.txt
```
You can then run the following commands from within the ``pysim`` directory. 
Make sure your card reader with the sim in it is inserted into your pc.
Go into the pysim folder and run the `pcsc_scan` command, make sure that the output mentions “card inserted”.
Check the current ISIM configuration: 

```bash

    ./pySim-read.py -p0
```

Reconfigure the ISIM: 

.. code-block:: bash

   ./pySim-prog.py -p0 -s <ICCID> --mcc=<MCC> --mnc=<MNC> -a <ADM-KEY> --imsi=<IMSI> -k <KI> --opc=<OPC> 

You need to set the PLMN to 00101, optionally you can also reconfigure other aspects of the ISIM. The following is an example of how the command should look like: 

```bash

   ./pySim-prog.py -p0 -s 8988211000000689694 --mcc=001 --mnc=01 -a 77190612 --imsi=001010000060591  -k 18E8DE5BA04A8D910B010D298672BCE0 --opc=B71C0607B0A5AE6E3657FEE846579991
```

You will need to get the ICCID, ADM-KEY and other security information from the SIM provider.

## <a name='SUPIconcealment'></a>3.2 SUPI concealment

You will need to modify the 5G-related fields of the sim card. In particular you need to configure or disable SUPI concealment (SUCI).

SUPI concealment can be disabled using the following commands. You should replace ``<ADM-KEY>`` with the ADM key of the respective SIM card. 

.. note::
   ``verify_adm`` does not print any output on success. If you see something like `"SW Mismatch: Expected 9000 and got 6982"` the ADM key is not correct. Keep in mind that after 
   3 failed write attempts due to a wrong ADM key the SIM is blocked and cannot be rewritten again.

```bash

    pySIM-shell (MF)> select MF
    pySIM-shell (MF)> select ADF.USIM
    pySIM-shell (MF/ADF.USIM)> select EF.UST
    pySIM-shell (MF)> verify_adm <ADM-KEY>
    pySIM-shell (MF/ADF.USIM/EF.UST)> ust_service_deactivate 124
    pySIM-shell (MF/ADF.USIM/EF.UST)> ust_service_deactivate 125
```
After these steps **UST service 124** and **125** should be disabled. You can verify the ISIM configuration using the following command:

```bash

    ./pySim-read.py -p0
```

More information on pySim and SUCI configuration can be found in `this guide <https://gist.github.com/mrlnc/01d6300f1904f154d969ff205136b753>`_, written by Merlin Chlosta. 

# <a name='OAIgNBpre-requisites'></a>4. OAI 5G SA gNB


Install and configure the OAI gNB pre-requisites as follows

## <a name='BuildUHDfromsource'></a>4.1 Build UHD from source
```bash
sudo apt install -y libboost-all-dev libusb-1.0-0-dev doxygen python3-docutils python3-mako python3-numpy python3-requests python3-ruamel.yaml python3-setuptools cmake build-essential

git clone https://github.com/EttusResearch/uhd.git ~/uhd
cd ~/uhd
git checkout v4.4.0.0
cd host
mkdir build
cd build
cmake ../
make -j $(nproc)
make test # This step is optional
sudo make install
sudo ldconfig
sudo uhd_images_downloader
```
## <a name='USRPx310Setup'></a>4.2 USRP x310 Setup

Install the x310 daughterboard according to the [Ettus Wiki](https://kb.ettus.com/USRP_X_Series_Quick_Start_(Daughterboard_Installation)).

### <a name='Prerequisites'></a>4.2.1 Prerequisites
Make sure you have the latest distribution packages.

```bash
sudo apt-get update && apt-get upgrade
```
### <a name='CheckPythoninstallation'></a>4.2.2 Check Python installation  
Ensure you have `python3 (3.6+)` installed. Install `pip3`.

```bash
sudo apt-get install python3-pip
```

Set the pythonpath in `~/.bashrc`.

```bash
export PYTHONPATH=/usr/local/lib/python3/dist-packages/:$PYTHONPATH
```
### <a name='ConfigurefirewalltoallowcommunicationwithUSRP'></a>4.2.3 Configure firewall to allow communication with USRP

Add an iptables rule to allow data from udp port 49152.

```bash
sudo iptables -A INPUT -p udp --sport 49152 -j ACCEPT
```

Make the iptables rule persistent across reboots

```bash
sudo apt install iptables-persistent

sudo su

iptables-save > /etc/iptables/rules.v4

exit
```

## <a name='InstallingGNUradiofromsourceOptionalnotrequiredforOAI'></a>4.3 Installing GNUradio from source (Optional/not required for OAI)

Follow the instructions from [the wiki](https://wiki.gnuradio.org/index.php/InstallingGR#From_Source). Additional notes are given below.

* We used the maint-3.9 branch.
* The maint-3.10 branch throws errors related to numpy.    
* Don't forget to install volk separately.  
* Don't forget to install dependencies from [here](https://wiki.gnuradio.org/index.php?title=UbuntuInstall#Focal_Fossa_.2820.04.29_through_Impish_Indri_.2821.10.29) before starting. we installed all the dependencies Bionic Beaver through Focal Fossa.


Make gives us error about gcc-7 being too old. Thus, install `gcc-8` as follows.

```bash
sudo apt-get install gcc-8

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 700 --slave /usr/bin/g++ g++ /usr/bin/g++-7

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 800 --slave /usr/bin/g++ g++ /usr/bin/g++-8
```

Check gcc configuration to make sure gcc-8 is default.

```bash
sudo update-alternatives --config gcc
```

### <a name='EnablingpythonsupportforGNUradio'></a>4.3.1. Enabling python support for GNUradio 

Install `pygccxml` for python3

```bash
sudo apt-get install -y python3-pygccxml
```

`pybind11` is needed for gnuradio python support.
Install pybind11 from source. Clone the pybind11 library from [Github](https://github.com/pybind/pybind11).
During the install, you will run into error regarding pytest.
To solve, run the following.

```bash
/usr/bin/python3.6 -m pip install pytest ====> if this doesnt work try: sudo apt-get install gnuradio python3-packaging and skip the next step.
```
Then do the standard make process as follows:

```bash
mkdir build
cd build
cmake ../
make
sudo make install
sudo ldconfig
```

Reboot to get pybind11 working. 

```bash
sudo reboot
```

Install pyqtgraph then update shared libraries.

```bash
sudo pip3 install pyqtgraph
sudo ldconfig
```
Check gnuradio install as follows:

```bash
gnuradio-config-info --version
```
## <a name='StartingandtestingtheUSRP'></a>4.4 Starting and testing the USRP 

Follow the guide below, if the usrp and daughterboards are assembled already, start at step 12

```bash
https://kb.ettus.com/USRP_X_Series_Quick_Start_(Daughterboard_Installation)
```
# <a name='OAI 5G SA gNB'></a>5. OAI 5G SA gNB

Install and deploy the OAI 5G SA gNB as follows

## <a name='BuildOAIgNB'></a>5.1 Build OAI gNB

```bash
# Get openairinterface5g source code
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git ~/openairinterface5g
cd ~/openairinterface5g
git checkout develop

# Install OAI dependencies
cd ~/openairinterface5g/cmake_targets
./build_oai -I

# Build OAI gNB
cd ~/openairinterface5g
source oaienv
cd cmake_targets
./build_oai -w USRP --ninja --gNB -c

```
## <a name='OAIgNBconfiguration'></a>5.2 OAI gNB configuration

You will need to adjust the gNB configuration file depending on the desired setup prior to running the gNB in the CONF folder. It contains multiple files for different USRP devices, bandwidth and frequency bands that can be adjusted to your specific setup. We include some of our custom configuration files in the conf-files folder for reference.

```bash
# Access the gNB configuration files
cd ~/openairinterface5g
cd targets/PROJECTS/GENERIC-NR-5GC/CONF/
sudo nano <conf_file>
```
* The MCC, MNC, TAC (tracking_area_code) and S-NSSAI (SST and SD) parameters should be the same as the values seen in the core configuration files.
* To configure the connection between the core and the gNB, you need to set the correct AMF parameters (amf_ip_address) to the address of the AMF and the correct network interfaces (NETWORK_INTERFACES).
* The selected frequency band and configuration should be supported by the UE, we used TDD band 78 with our Google Pixel 7 PRO UE.

# <a name='SetupandconfigureUE'></a>6. Setup and configure UE

## <a name='SetupCOTSUE'></a>6.1 Setup COTS UE

You should make sure your UE device is capable of operating in 5G SA mode and that it operates in the bands supported by the OAI Project gNB. As mentioned previously, we have used a Google Pixel 7 PRO in this turotial.

To configure the phone UE to connect to the 5G network the following steps must be taken:

* In the mobile network options, make sure that the SIM and the use of a 5G NR carrier are enabled.
* Set 5G as the preferred network type. If you cannot see the 5G option in Preferred network type, then you may need to activate it. This can be enabled under the Developer Options, if you do not have access to Developer Options see [this guide](https://developer.android.com/studio/debug/dev-options). In Developer Options go to Networking and enable 5G, you may also need to set 5G network mode to NSA + SA Mode
* The APN needs to be set to the same as the DNN/APN option as set in the OAI CN5G core subscriber registration.
* In some phones there may also be an option to configure VoNR and/or VoLTE, it is important to make sure that this is disabled.

## <a name='SetupOAIUE'></a>6.2 Setup OAI UE

You will need to adjust the ue configuration file depending on the desired setup prior to running the ue in the CONF folder.

```bash
# Access the gNB configuration files
cd ~/openairinterface5g
cd targets/PROJECTS/GENERIC-NR-5GC/CONF/
sudo nano ue.conf
```
* The APN needs to be set to the same as the DNN/APN option as set in the OAI CN5G core subscriber registration.
* S-NSSAI (SST and SD) parameters should be the same as the values seen in the core configuration files.

# <a name='Deploythe5GSAend-to-endnetwork'></a>7. Deploy the 5G SA end-to-end network

## <a name='RunOAICN5G'></a>7.1 Run OAI CN5G

```bash
cd ~/oai-cn5g
docker compose up -d
```

## <a name='RunOAIgNB'></a>7.2 Run OAI gNB

### <a name='USRPX310'></a>7.2.1 USRP X310
```bash
cd ~/openairinterface5g
source oaienv
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/5gnb.band78.fr1.106PRB.sa.usrpx310.conf --sa --tune-offset 20000000
```
### <a name='RFSIMULATOR'></a>7.2.2 RFSIMULATOR Mode
```bash
cd ~/openairinterface5g
source oaienv
cd cmake_targets/ran_build/build
sudo RFSIMULATOR=server ./nr-softmodem --rfsim --sa -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf
```
## <a name='ConnectUE'></a>7.3 Connect UE

Prior to connecting the UE, make sure that the gNB is running and is properly connected to the core.

### <a name='ConnectCOTSUE'></a>7.3.1 Connect COTS UE

Go into the network and cellular data parameters in the device and perform the following:

* Disable automatic network selection if it is enabled.
* Manually select and connect to the network. It will be displayed as the core name or PLMN value.
* Monitor the AMF and gNB logs and make sure that the UE has successfully attached to the network.
* Test connectivity and bandwidth by running streaming data or running a speedtest.

### <a name='ConnectSimulatedUE'></a>7.3.2 Connect Simulated UE

If you are using RFsimulator, you can deploy and connect the OAI simulated UE as follows:

```bash
cd ~/openairinterface5g
source oaienv
cd cmake_targets/ran_build/build
sudo RFSIMULATOR=127.0.0.1 ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000  --rfsim --sa -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf
```
* The command-line switch paramters need to match the values and configuration of the gNB.

# <a name='Troubleshootingdebuggingadvancedconfiguration'></a>8. Troubleshooting, debugging and advanced configurations

## <a name='DebuggingUHD'></a>8.1 Debugging UHD

* If you get `" No devices found for ----->"` error when trying to run `uhd_find_devices`, ensure that you have the [iptables rule](#NointernetonUEmasqdoesnotwork) in place.

* If you are following the [Ettus daughterboard installation wiki](https://kb.ettus.com/USRP_X_Series_Quick_Start_(Daughterboard_Installation)), note that the `uhd_fft` utility is **not** installed by `UHD`, but rather by `GNUradio`.

## <a name='NetworknotvisibleontheUE'></a>8.2 Network not visible on the UE
* Verify the APN and make sure that it is the same as the value present in the AMF configuration.
* Verify the ISIM information in the OAI subscriber database (oai_db2.sql) and make sure that all the parameters are inputed correctly in the right format.
* Verify the frequency band in the gNB configuration and make sure that the UE supports it.
* In the gNB configuration file, experiment with different rx_gain values.


## <a name='NointernetonUEmasqdoesnotwork'></a>8.3 No internet on UE (masq does not work)

Docker changed the default policy of the `FORWARD` chain to drop, as well as added a bunch of rules in the `POSTROUTING` chain in the `nat` table.

To fix this, we need to add an `ACCEPT` rule at the beginning of the `FORWARD` chain and add the masquerade rule at the beginning of the `POSTROUTING` chain. We add them at the beginning so that the packets from the UE’s don’t go and match the docker rules.

```bash
sudo iptables -I FORWARD 1 -s 172.16.0.0/24 -j ACCEPT
sudo iptables -t nat -I POSTROUTING 1 -s 172.16.0.0/24 -o enx2c16dbab4418 -j MASQUERADE
```

In the rules above, I’ve specified the source IP (IP assigned to UE by the OAI 5G core) to make the rules more specific. This makes sure that these rules don’t conflict with the docker rules.

## <a name='USRPN300andX300EthernetTuning'></a>8.4 USRP N300 and X300 Ethernet Tuning

Please also refer to the official [USRP Host Performance Tuning Tips and Tricks](https://kb.ettus.com/USRP_Host_Performance_Tuning_Tips_and_Tricks) tuning guide.

The following steps are recommended. Please change the network interface(s) as required. Also, you should have 10Gbps interface(s): if you use two cables, you should have the XG interface. Refer to the [N300 Getting Started Guide](https://kb.ettus.com/USRP_N300/N310/N320/N321_Getting_Started_Guide) for more information.

* Use an MTU of 9000: how to change this depends on the network management tool. In the case of Network Manager, this can be done from the GUI.
* Increase the kernel socket buffer (also done by the USRP driver in OAI)
* Increase Ethernet Ring Buffers: `sudo ethtool -G <ifname> rx 4096 tx 4096`
* Disable hyper-threading in the BIOS (This step is optional)

Example code to run:
```
for ((i=0;i<$(nproc);i++)); do sudo cpufreq-set -c $i -r -g performance; done
sudo sysctl -w net.core.wmem_max=62500000
sudo sysctl -w net.core.rmem_max=62500000
sudo sysctl -w net.core.wmem_default=62500000
sudo sysctl -w net.core.rmem_default=62500000
sudo ethtool -G enp1s0f0 tx 4096 rx 4096
```
* Enable Performance Mode `sudo cpupower idle-set -D 0`
* If you get real-time problems on heavy UL traffic, reduce the maximum UL MCS using an additional command-line switch: `--MACRLCs.[0].ul_max_mcs 14`.
* There is noise on the DC carriers on N300 and especially the X300 in UL. To avoid their use or shift them away from the center to use more UL spectrum, we used the `--tune-offset <Hz>` command line switch, where `<Hz>` is ideally half the bandwidth, or possibly less. For Example `--tune-offset 20000000` for 40Mhz bandwidth.

## <a name='Authors'></a>Authors

* **Mohamed Rouili** - [mrouili](https://github.com/mrouili)
* **Niloy Saha** - [niloysh](https://github.com/niloysh)

## <a name='License'></a>License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## <a name='Acknowledgments'></a>Acknowledgments

* The tutorials and deployment instructions described in this guide are based on the guides and documentation provided by the [OpenAirInterface](https://gitlab.eurecom.fr/oai/openairinterface5g/-/tree/develop/doc) and [srsRAN](https://github.com/srsran/srsRAN_Project_docs/tree/main/docs/source/tutorials/source) projects. All credits go to their respective authors.
