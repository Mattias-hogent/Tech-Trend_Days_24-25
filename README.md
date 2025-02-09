# Setup Guide

## Important
During testing we weren't able to get the lab fully working on VirtualBox so we recommend using VMware workstation (pro). If the account creation is acting up you can still download the executable on the update repository <https://softwareupdate.broadcom.com/cds/vmw-desktop/ws/17.6.2/24409262/windows/core/>. You can extract the tar file with [7-Zip](https://www.7-zip.org/).

## Virtual machine requirements

- 4 GiB RAM
- 2 vCPU's
- 20 GB storage
- [Ubuntu 24.04 LTS](https://ubuntu.com/download/server)

Makes sure to install the openssh server during the ubuntu installation.
It's also important to enable **Virtualize Intel VT-x/EPT or AMD-V/RVI in VMware workstation pro** or enable **Nested VT-x/AMD-V in VirtualBox** on the VM. This option can typically be found under the proccessor settings of the VM.

## Installing containerlab

```bash
# Makes sure the system is up to date.
sudo apt update -y && sudo apt full-upgrade -y
# Downloads and runs the containerlab setup script.
curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
```

## Building the Mikrotik Routeros v7 docker image

```bash
# Clones the vrnetlab repo. This builds the routeros image.
git clone https://github.com/hellt/vrnetlab
# Downloads the Mikrotik routers v7.17.2 Cloud Hosted Router vmdk file.
wget https://download.mikrotik.com/routeros/7.17.2/chr-7.17.2.vmdk.zip
# Installs the unzip utility.
sudo apt install unzip
# Unzips the Mikrotik routers v7.17.2 Cloud Hosted Router vmdk file.
unzip chr-7.17.2.vmdk.zip
# Move the Mikrotik routers v7.17.2 Cloud Hosted Router vmdk file to the right location so it can be used to build the docker image.
mv chr-7.17.2.vmdk vrnetlab/routeros/
# Goes to the correct directory to build the Mikrotik routers v7.17.2 Cloud Hosted Router docker image.
cd ./vrnetlab/routeros
# Builds the Mikrotik routers v7.17.2 Cloud Hosted Router docker image.
sudo make docker-image
```

We can check if the image has been build correctly with `sudo docker image ls` the output should be similar to this.

```txt
REPOSITORY                   TAG       IMAGE ID       CREATED          SIZE
vrnetlab/mikrotik_routeros   7.17.2    b22a0571bf79   37 minutes ago   973MB
```

## Setting up a bridge interface for container lab to use

We have to edit the `/etc/netplan/50-cloud-init.yaml` file to add a bridge. You have to add the following code to the end of the file.

```yaml
bridges:
    ixp-net:
        interfaces: []
        optional: true
```

The resulting file should look similar to this. The entries under **ethernets** depends on how many interfaces your vm has so this can vary depending on your setup.

```yaml
network:
    ethernets:
        ens32:
            dhcp4: true
        ens33:
            dhcp4: true
    version: 2

    bridges:
        ixp-net:
            interfaces: []
            optional: true
```

You can check the file with `sudo netplan try` to makes sure the file is valid and you can apply changes with `sudo netplan apply`. After apply the netplan you can check if the bridge interface has been create with `ip -c a` the output should contain something similar to this.

```txt
7: ixp-net: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether aa:7c:79:c5:15:9e brd ff:ff:ff:ff:ff:ff
```
