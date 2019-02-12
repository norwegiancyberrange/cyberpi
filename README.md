# Automate Chromium kiosk mode for Raspberry Pi with Ansible

This workbook will automate the setup of a Raspberry Pi for use as a simple and robust digital signage client. It is tweaked for use with Kitteh, but should be suitable for any web based system. This is a fork of [Pikiosk](https://github.com/chriso0710/pikiosk/), with a lot of things changed and/or removed. This was in turn forked from [5Minutes - Server Security Essentials](https://github.com/chhantyal/5minutes).

## Ansible automation

Start with a default Raspbian image with SSH enabled and the password set to something you know. The first step will lock down the built in pi user, and secure the installation. 

### Basic requirements

- Raspbian Lite
- Ansible
- Micro SD card reader/writer
- Some sort of website to present your content

### 1. Setup SD card image

Burn the Raspbian Jessie image on to your SD card. [Etcher](https://etcher.io/) can do this, unless you know your way around dd.
Connect your Pi to the network via ethernet cable, power it on and boot.

If you create a valid `wpa_supplicant.conf` file and place in your boot partition the Pi copies it over using stock images (like Raspbian Jessie). The file disappears after it copies, but activates network on first boot so you should be able to find your Pi via nmap/ssh in your Wifi network.

### 2. Find the IP adresses of your Pis and change Ansible hosts file

Set the Pi addresses in your Ansible hosts file.
Add the host variables `pi_name` and `chrome_url` to a file named `hosts`. They are used by the playbooks for setting the hostname and the target URL. See `hosts-example` for the correct syntax.

### 3. Set more variables

Please set some required variables in vars.yml, which you might want to change. You should at least set the Ansible management user and the key. See also vars.yml-example.

```
server_user_name: kiosk-admin
server_user_password: raspberry
user_public_keys:
  - ~/.ssh/id_rsa.pub
```

### 4. First run, bootstrap and secure your Pis

Use the user "pi" with the default password that you set during setup (the default password is "raspberry").

You may want to check that your Pis are responding on the ssh port:

```
$ ansible all -m ping -u pi -k
```

Execute the secure.yml playbook to secure your Pis:
- sets hostname and timezone
- creates a new sudo user with key login
- locks down the pi account
- disables ssh root access and password authentication
- installs some packages
- sets up ufw firewall

```
$ ansible-playbook secure.yml -u pi -k
```

After running the secure.yml playbook you can omit the username for Ansible and you won't be able to login with a password.

### 5. Update Raspbian (optional)

Use the apt.yml playbook to update the OS:

```
$ ansible-playbook apt.yml
```

This does only a safe upgrade, no dist upgrade. The playbook checks if the server needs to be restarted.

### 6. Setup the kiosk

This is the main part. Use the kitteh.yml playbook to set up Chromium in kiosk mode and make it autostart:

```
$ ansible-playbook kitteh.yml
```

This does the following:
- set HDMI modes
- install unclutter, MS core fonts, chromium-browser and various other required packages
- disable screensaver
- configure Chromium autostart

The playbook checks if X environment needs to be restarted if you changed the chrome_url variable.
After changing HDMI modes the Pi will reboot.

### 7. Get info (optional)

Use the info.yml playbook to get some info about your Pi flock:

```
$ ansible-playbook info.yml
```

## License

See [MIT License](LICENSE.txt)