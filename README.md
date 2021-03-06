# Overview

This project provides a Vagrant machine that connects to a Private Internet Access VPN. Undesirable connections are
Firewalled to protect from dns and other leakage in the event of the VPN becoming disconnected.

# Requirements

* [Vagrant](https://www.vagrantup.com/)
* [Ansible](www.ansible.com)
* This is set up for the VPN provider [Private Internet Access](https://www.privateinternetaccess.com/)  
  if you prefer another VPN provider, you have to edit the `/etc/openvpn` folder and disable the installation of the firewall in `site.yml`.

On Debian like systems just install

    apt-get install vagrant ansible


# Configuration

## Environment Variables

In `Vagrantfile` you must configure these variables:

* PIAUSERNAME: Your Private Internet Access Username
* PIAPASSWORD: Your Private Internet Access Password
* DELUGEWEBPORT: the port opened on this machine; default 8112


# Usage

Bring up the VM using

    vagrant up

This will automatically download a fresh Ubuntu 14.04 vagrant image and then start provisioning the vm with the settings from `Vagrantfile` and the ansible playbook `site.yml`.

NOTE: Software packages are downloaded (i.e. openvpn) from public Ubuntu repositories before the PIA VPN is established.
Use a VPN on the host machine if you would like to prevent this leakage.

## openvpn

The VPN should be available on reboot. If it is not the Firewall rules will block any outgoing connections with
the exception of DNS (to PIA's DNS) and establishing a connection to the default VPN.
    
The openvpn service can be managed with `/etc/init.d/openvpn`. To restart the VPN use:

    sudo /etc/init.d/openvpn restart

## deluge

Once started Deluge UI should be available on [http://localhost:8112](http://localhost:8112).

**Connect to Deluge UI and change the password immediately. The old password is "deluge".**

The configuration for this exists in `Vagrantfile` and can be changed using an environment variable

    config.vm.network "forwarded_port", guest: delugeport, host: 8112

Deluge should move completed files to `/home/deluge/completed`. To share this folder with the host system, 
edit `Vagrantfile` and change `disabled` to `false` as shown.

    config.vm.synced_folder "./files/downloads", "/home/deluge/complete", owner: "deluge", disabled: false

This change will need to be made after the system is provisioned and `/home/deluge/complete` exists. After
the change is made use `vagrant reload` to restart the system.

## git

Git support has been provided to help contributors provide fixes. It can be commented out in `site.yml`

To share local files edit `Vagrantfile` and change `disabled` to `false` as shown

    config.vm.synced_folder "./files/src", "/home/vagrant/src", disabled: false

## test

The test tole provides basic testing support at the command line.

Check your external ip address:

    /opt/bin/testvpn.sh

Hope to add dns leak tests and more later.


# Troubleshooting

## I can't connect to anything

Your VPN might be disconnected.

* Connect to the guest machine
* try to ping google
* if the ping timed out try restarting openvpn

```
vagrant ssh
ping google.com
sudo /etc/init.d/openvpn reset
```

## deluge web isn't working

deluge may not be started

* check processes it should show 3 entries
* check deluge logs
* restart deluged and deluge-web

```
ps aux | grep deluge
sudo -u deluge tail /var/log/deluge/daemon.log
sudo restart deluged
sudo restart deluge-web
```
