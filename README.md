# Secure Remote IoMT Device using Software TPM

## Overview  
This project focuses on securely accessing patient monitoring devices in the **Internet of Medical Things (IoMT)** ecosystem. The key objective is to ensure **secure remote authentication and verification** of these devices using **Trusted Platform Module (TPM) and Physical Unclonable Functions (PUF)**.  

A **real-world scenario** considered is a patient with a pacemaker traveling internationally—where a doctor in another country needs **secure access** to the device’s health data. To ensure the security of these communications, **TPM-based attestation and cryptographic techniques** are implemented.

## Features  
- **TPM-based security** for IoMT devices  
- **Physical Unclonable Function (PUF)** for authentication  
- **SHA-256 cryptographic integrity verification**  
- **TPM quote generation and verification** for remote attestation  
- **Software TPM (IBM TPM Simulator) support** for testing  
- **Replay attack prevention** using nonce values  

## Technologies Used  
- **IBM TPM simulator**
- **TPM2-tools**
- **Openssl version 1.1**
- **Virtual box 7.1.4**
- **Ubuntu 24.04.1 version**

## Installation  

### Steps  
1. **Install Dependencies**  
   ```bash
   sudo apt-get install lcov 
   sudo apt-get install lcov autoconf-archive liburiparser-dev
   sudo apt install libdbus-1-dev libglib2.0-dev dbus-x11
   sudo apt install libssl-dev autoconf automake
   sudo apt install libtool-bin pkg-config gcc 
   sudo apt install libcurl4-gnutls-dev libgcrypt20-dev libcmocka-dev uthash-dev


2. **Install TPM Simulator**
   ```bash
   wget https://jaist.dl.sourceforge.net/project/ibmswtpm2/ibmtpm1661.tar.gz
   mkdir ibmtpm1661
   cd ibmtpm1661
   tar -xzvf ../ibmtpm1661.tar.gz
   cd src/
   sudo make

3. **Install OPENSSL 1.1 version**
   ```bash
   sudo apt update sudo apt install build-essential checkinstall zlib1g-dev
   wget https://www.openssl.org/source/openssl-1.1.1.tar.gz
   tar -xvzf openssl-1.1.1.tar.gz
   cd openssl-1.1.1 ./config
   make sudo
   make install
   sudo mv /usr/bin/openssl /usr/bin/openssl.backup
   sudo ln -s /usr/local/bin/openssl /usr/bin/openssl
   sudo ldconfig
   
4. **Setup TPM Server**
   ```bash
   sudo vim /lib/systemd/system/tpm-server.service
- Once after entering in file, copy paste below code to start the server
  ```bash
  [Unit]
  Description=TPM2.0 Simulator Server daemon 
  Before=tpm2-abrmd.service 
  [Service] 
  ExecStart=/usr/local/bin/tpm-server 
  Restart=always 
  Envi- ronment=PATH=/usr/bin:/usr/local/bin 
  [Install] 
  WantedBy=multi-user.target

5. **Start the daemon - tpm server**
   ```bash
   systemctl daemon-reload
   systemctl start tpm-server.service
   service tpm-server status
   
6. **Install CA certificates**
   ```bash
   sudo apt install ca-certificates
   
- Once server start it looks like this
  
  ![image alt](https://github.com/teja-tp0472/Secure-Remote-IoMT-device-using-Software-TPM/blob/abeb2e12daa63adce6dc2dd8a56076d636288114/Screenshot%202025-01-28%20215606.png)

7. **Install TPM2-TSS**
   ```bash
   sudo apt-get install libjson-c-dev
   wget https://github.com/tpm2-software/tpm2-tss/releases/download/3.1.0/tpm2-tss-3.1.0.tar.gz
   tar -xzvf tpm2-tss-3.1.0.tar.gz cd tpm2-tss-3.1.0/
   ./configure
   sudo make install
   sudo ldconfig

8. **Install TPM2-ABRMD**
   ```bash
   wget https://github.com/tpm2-software/tpm2-abrmd/releases/download/2.3.1/tpm2- abrmd-2.3.1.tar.gz
   sudo useradd –system –user-group tss
   tar -xzvf tpm2-abrmd-2.3.1.tar.gz
   cd tpm2-abrmd-2.3.1
   sudo ldconfig
   ./configure –with-dbuspolicydir=/etc/dbus-1/system.d –with systemdsystemunitdir=/usr/lib/systemd/system
   sudo make install
   sudo cp /usr/local/share/dbus-1/system-services/com.intel.tss2.Tabrmd.service /usr/share/dbus- 1/system-services/
   sudo pkill -HUP dbus-daemon
   sudo vim /lib/systemd/system/tpm2-abrmd.service
   
- Once the file is open overwrite the text with the below text
  ```bash
  [Unit]
  Descript=TPM2 Access Broker and Resource Management Daemon
  [Service]
  Type=dbus
  Restart=always
  RestartSec=5
  BusName=com.intel.tss2.Tabrmd
  StandardOutput=syslog
  ExecStart=/usr/local/sbin/tpm2-abrmd –tcti=”libtss2-tcti-mssim.so.0:host=127.0.0.1,port=232
  User=tss [Install]
  WantedBy=multi-user.target 

- Check if server is running or not, if not running, restart it by using the below commands
  ```bash
  sudo apt-get install ibmswtpm2
  sudo tpm-server &
  sudo nano /usr/local/lib/systemd/system/tpm2-abrmd.service
  Sudo systemctl daemon-reload
  Sudo systemctl start tpm2-abrmd.service
  Sudo service tpm2-abrmd status

9. **Install TPM2-TSS Engine**
    ```bash
    wget https://github.com/tpm2-software/tpm2-tss-engine/releases/download/v1.1.0/tpm2-tss-engine-1.1.0.tar.gz
    ll /usr/lib/x8664 − linux − gnu/engines − 1.1/
    opensslengine − t – c

10. **Install TPM2-TOOLS**
    ```bash
    Wget https://github.com/tpm2-software/tpm2-tools.git
    cd tpm2-tools
    git checkout 5.4
    ./bootstrap
    ./configure
    make -j$(nproc)
    sudo make install
    sudo ldconfig

11. **Random Number Generation**
- Now the whole set is ready and even the servers are running, so let's check for the random numbers generation
  ```bash
  openssl rand -engine tpm2tss -hex 10

12. **Generating Initial PCR values**
    ``bash
    tpm2_pcrread

- You can see SHA1,SHA256,SHA384,SHA512 PCR values.
  <image>
- Now let's generate the quote and validate it by using below commands

13. **Generate the Quote and validate it without PCR Extension**
    ```bash
    tpm2_createprimary -C e -c primary.ctx
    tpm2_createek -c ek.ctx -G rsa -u ek.pub
    tpm2_create -C primary.ctx -G rsa -u ak.pub -r ak.priv -c ak.ctx.
    tpm2_readpublic -c ak.ctx -o ak.pub
    tpm2_pcrread sha256:0,1,2,3
    tpm2_quote -c ak.ctx -l sha256:0,1,2,3 -q <nounce_value> -m quote.msg -s quote.sig -o pcr_values.out

- Here you can specify any values as you like eg:1231231231231231
- Now the Quote is generated, need to validate it
  
   ```bash
   tpm2_checkquote -u ak.pub -m quote.msg -s quote.sig -f pcr_values.out -q <nounce_value>

- Now lets try with PCR extension

14. **Generate the Quote and validate it PCR Extension**
     ```bash
     tpm2_createprimary -C e -c primary.ctx
     tpm2_createek -c ek.ctx -G rsa -u ek.pub
     tpm2_create -C primary.ctx -G rsa -u ak.pub -r ak.priv -c ak.ctx.
     tpm2_readpublic -c ak.ctx -o ak.pub
     
- Here we generate the PCR Extension
     ```bash
  echo -n "example" | sha256sum | awk '{print $1}'
  tpm2_pcrextend 2:sha256=50d858c8b18cbff2d5c74e4dba7d3b8b30e41dc35d0026b9c5e26b48d386c78f
  tpm2_pcrread sha256:0,1,2,3
  tpm2_quote -c ak.ctx -l sha256:0,1,2,3 -q <nounce_value> -m quote.msg -s quote.sig -o pcr_values.out
  
- (here you can specify any values as you like eg:1231231231231231 )
- Now the Quote is generated, need to validate it
   ```bash
   tpm2_checkquote -u ak.pub -m quote.msg -s quote.sig -f pcr_values.out -q <nounce_value>


  

