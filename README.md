# cuckoo sandbox
Cuckoo Sandbox is an open-source automated malware analysis system that executes malware in isolated virtual machines. It captures real-time behaviors like file system changes, network traffic, and API calls. With detailed reports and plugin extensibility, Cuckoo is vital for malware research, threat intelligence, and incident response.

# Cuckoo Sandbox Installation Guide

Cuckoo Sandbox is an open-source automated malware analysis system that executes malware in isolated virtual machines, allowing for detailed behavioral analysis.

## Prerequisites

Before starting, ensure your system is updated:

```bash
sudo apt update
sudo apt upgrade
Install Required Packages

# Python 2.x and Pip:
# bash

sudo apt install python2.7 python-pip

# Install Additional Packages:

# bash

    sudo apt-get install python python-pip python-dev libffi-dev libssl-dev -y
    sudo apt-get install python-virtualenv python-setuptools -y
    sudo apt-get install libjpeg-dev zlib1g-dev swig -y
    sudo apt-get install mongodb -y
    sudo apt-get install postgresql libpq-dev -y
    sudo apt install virtualbox -y
    sudo apt-get install tcpdump apparmor-utils -y

# Create Cuckoo User

# bash

## skip this : sudo adduser --disabled-password --gecos "" cuckoo

# Setup TCPDump Permissions

# bash

sudo groupadd pcap
sudo usermod -a -G pcap <current_user_name>
sudo chgrp pcap /usr/sbin/tcpdump
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
sudo aa-disable /usr/sbin/tcpdump

# Create Setup Script

# Create a directory for your setup scripts:

# bash

mkdir opt
cd opt

# Install vim or nano to create files:

# bash

sudo apt-get install vim nano
nano cuckoo-setup-virtualenv.sh

# Copy and paste the following script into cuckoo-setup-virtualenv.sh:

# bash
---------------------------------------------------------------------------------------
#!/usr/bin/env bash

# NOTES: Run this script as: sudo -u <USERNAME> cuckoo-setup-virtualenv.sh

# install virtualenv
sudo apt-get update && sudo apt-get -y install virtualenv

# install virtualenvwrapper
sudo apt-get -y install virtualenvwrapper

echo "source /usr/share/virtualenvwrapper/virtualenvwrapper.sh" >> ~/.bashrc

# install pip for python3
sudo apt-get -y install python3-pip

# turn on bash auto-complete for pip
pip3 completion --bash >> ~/.bashrc

# avoid installing with root
pip3 install --user virtualenvwrapper

echo "export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3" >> ~/.bashrc

echo "source ~/.local/bin/virtualenvwrapper.sh" >> ~/.bashrc

export WORKON_HOME=~/.virtualenvs

echo "export WORKON_HOME=~/.virtualenvs" >> ~/.bashrc

echo "export PIP_VIRTUALENV_BASE=~/.virtualenvs" >> ~/.bashrc 
------------------------------------------------------------------------------------------
#Set Permissions and Execute

# bash

sudo chmod +x cuckoo-setup-virtualenv.sh
sudo -u <your_username> ./cuckoo-setup-virtualenv.sh
source ~/.bashrc

# Create Virtual Environment

# bash

mkvirtualenv -p python2.7 cuckoo-test
pip install -U pip setuptools
pip install -U cuckoo
-------------------------------------------------------------------------------------------
# Setup Virtual Machine
# Download Windows 7 ISO link in requirements and mount it:

# bash

sudo mkdir /mnt/win7
sudo chown cuckoo:cuckoo /mnt/win7/
sudo mount -o ro,loop win7ultimate.iso /mnt/win7

# Install vmcloak:

# bash

    pip install -U vmcloak
    vmcloak-vboxnet0
    vmcloak init --verbose --win7x64 win7x64base --cpus 2 --ramsize 2048
    vmcloak clone win7x64base win7x64cuckoo
    vmcloak install win7x64cuckoo ie11
    vmcloak snapshot --count 1 win7x64cuckoo 192.168.56.101
------------------------------------------------------------------------------------------
# Interacting with Cuckoo

# bash

cuckoo init
cd .cuckoo/
cuckoo community

# In a new terminal, install net-tools:

# bash

sudo apt install net-tools

# Add VMs to Cuckoo:

# bash

while read -r vm ip; do cuckoo machine --add $vm $ip; done < <(vmcloak list vms)

# Configure IP forwarding:

# bash

sudo sysctl -w net.ipv4.conf.vboxnet0.forwarding=1
sudo sysctl -w net.ipv4.conf.ens33.forwarding=1

# Set up IPTables:

# bash

sudo iptables -t nat -A POSTROUTING -o ens33 -s 192.168.56.0/24 -j MASQUERADE
sudo iptables -P FORWARD DROP
sudo iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 192.168.56.0/24 -j ACCEPT

# Final Configuration

# Switch to the Cuckoo test environment:

# bash

cd conf/
nano routing.conf
nano virtualbox.conf

# Modify the necessary configurations.
# Start Cuckoo

# Run the following commands in the sandbox:

# bash

cuckoo rooter --sudo --group <current_user_name>
cuckoo web --host 127.0.0.1 --port 8080

# Conclusion

You have successfully set up Cuckoo Sandbox for automated malware analysis. For further information, refer to the Cuckoo documentation.

markdown


### Notes:
- Replace `<current_user_name>` and `<your_username>` with your actual username.
- Ensure you have the Windows 7 ISO available to mount. You can download it from a reliable source if needed.
- Adjust any paths or commands specific to your setup or preferences.

