# Automate Chromium pseudo-kiosk mode for Raspberry Pi with Ansible

This workbook will automate the setup of a Raspberry Pi for use as a simple and robust digital signage client. It is tweaked for use in Norwegian Cyber Range, but should be suitable for any web based system. This is a fork of [kittehpi](https://github.com/einarjh/kittehpi), which is a fork of [Pikiosk](https://github.com/chriso0710/pikiosk/), with a lot of things changed and/or removed. This was in turn forked from [5Minutes - Server Security Essentials](https://github.com/chhantyal/5minutes).

## Ansible automation

Start with a default Raspbian image with SSH enabled and the password set to something you know. The first step will lock down the built in pi user, and secure the installation. 

### Basic requirements

- Raspberry Pi OS Lite
- Ansible
- Micro SD card reader/writer
- Some sort of website to present your content

### 1. Setup SD card image

Write the Raspberry Pi OS lite image on to your SD card. [Etcher](https://etcher.io/) can do this, unless you know your way around dd.
Connect your Pi to the network via ethernet cable, power it on and boot. Log on with username "pi" and password "raspberry", then run `sudo raspi-config` to change the password, and then go to `interfacing options` and enable the SSH server.

If you create a valid `wpa_supplicant.conf` file and place in your boot partition the Pi copies it over using stock images (like Raspbian Jessie). The file disappears after it copies, but activates network on first boot so you should be able to find your Pi via nmap/ssh in your Wifi network.

### 2. Find the IP adresses of your Pis and change Ansible hosts file

Set the Pi addresses in your Ansible hosts file.
Add the host variables `pi_name` and `chrome_url` to a file named `hosts`. They are used by the playbooks for setting the hostname and the target URL. See `hosts-example` for the correct syntax.

### 3. Set more variables

Please make a copy of group_vars/all-example to group_vars/all and change any variables to suit your needs. You should at least set the Ansible management user and the key.

```
ansible_user: kiosk-admin
ansbile_ssh_pass: raspberry
user_public_keys:
  - ~/.ssh/id_rsa.pub
```

### 4. First run, bootstrap and secure your Pis

Use the user "pi" with the default password that you set during setup.

You may want to check that your Pis are responding on the ssh port:

```
$ ansible all -m ping -e 'ansible_user=pi' -e 'ansible_ssh_pass=<password>'
```

Execute the bootstrap.yml playbook to secure your Pis:
- sets hostname and timezone
- creates a new sudo user with key login
- locks down the pi account
- disables ssh root access and password authentication
- installs some packages
- sets up ufw firewall

```
$ ansible-playbook bootstrap.yml -e 'ansible_ssh_user=pi' -e 'ansible_ssh_pass=<password>'
```

After running the bootstrap.yml playbook you can omit the username for Ansible and you won't be able to login with a password.

### 5. Update Raspbian (optional)

Use the apt.yml playbook to update the OS:

```
$ ansible-playbook apt.yml
```

This does only a safe upgrade, no dist upgrade. The playbook checks if the server needs to be restarted.

### 6. Install the Argon One Power Button & Fan Control

Our CyberPis is encased in the Argon One Pi4 casing. This needs some software for fan-control and power button features. Run the argonone.yml playbook to install this. The playbook will reboot the server if the software is installed:

```
$ ansible-playbook argonone.yml
```

### 7. Setup the pseudo-kiosk

This is the main part. Use the cyberpi.yml playbook to set up Chromium in fullscreen mode and make it autostart:

```
$ ansible-playbook cyberpi.yml
```

This does the following:
- install unclutter, MS core fonts, chromium-browser and various other required packages
- disable screensaver
- install xrdp
- configure Chromium autostart

The playbook checks if X environment needs to be restarted if you changed the chrome_url variable.

### 8. Get info (optional)

Use the info.yml playbook to get some info about your Pi flock:

```
$ ansible-playbook info.yml
```

### 9. Install munin-node (optional)
If you want, run the munin playbook to install and configure munin-node. Remeber to set this variable in your group_vars:
```
munin_server: ^123\.456\.789\.101$ # In the regex format needed for munin-node.conf
```
And the run the munin playbook:
```
$ ansible-playbook munin.yml
```


### 10. Log in with rdp client to inspect status of monitor

You can visually inspect what is being viewed on the screen by logging in with an RDP client, either the built-in client on Linux or for instance with xfreerdp.

If you're using a newer version of xfreerdp, you will have to supply two options to work around some bugs in the current xrdp server:

```
$ xfreerdp /relax-order-checks +glyph-cache /v:<ip-or-hostname>
```

## License

See [MIT License](LICENSE.txt)
