# Setup Guide

## Virtual machine requirements

- 4 GiB RAM
- 2 vCPU's
- Ubuntu 24.04 LTS

Makes sure to install the openssh server during the ubuntu installation.
It's also important to enable **Virtualize Intel VT-x/EPT or AMD-V/RVI in VMware workstation pro** or enable **Nested VT-x/AMD-V in VirtualBox** on the VM.

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

The resulting file should look similar to this.

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

you can check the file with `sudo netplan try` to makes sure the file is valid and you can apply changes with `sudo netplan apply`.
