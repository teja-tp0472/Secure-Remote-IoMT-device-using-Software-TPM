# Secure Remote IoMT Device Handshaking for Smart Healthcare - Doctors' Dilemma Problem

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

Once after entering in file, copy paste below code to start the server
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


