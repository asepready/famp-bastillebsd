# famp-bastillebsd
FreeBSD, Apache2, MariaDB, and PHP(PHPMyAdmin)

## Requirements
Before you can install FAMP using this template you need some initial configurations

#### Create a loopback interface
We can create it add some lines to /etc/rc.conf
```sh
sysrc cloned_interfaces+=lo1
sysrc ifconfig_lo1_name="bastille0"
service netif cloneup
```
#### Enable Packet filter
We need add somes lines to /etc/rc.conf

```sh
sysrc pf_enable="YES"
```
Edit /etc/pf.conf and modify like you want. Minimal working configuration could look at the following

```sh
ext_if="vtnet0"

set block-policy return
scrub in on $ext_if all fragment reassemble
set skip on lo

table <jails> persist
nat on $ext_if from <jails> to any -> ($ext_if:0)
rdr-anchor "rdr/*"

block in all
pass out quick keep state
antispoof for $ext_if inet
pass in inet proto tcp from any to any port ssh flags S/SA keep state
```
rdr-anchor section is necessary for use dynamic redirect from jails

Start packet filter

```sh
service pf start
```

#### Bootstrap a FreeBSD version
Before you can begin creating containers, Bastille needs to "bootstrap" a release. It must be a version equal or lesser than your host version. In this example we will create a 14.1-RELEASE bootstrap

```sh
bastille bootstrap 14.1-RELEASE
```
#### Create a lightweight container system
Create a container named FAMP with a private IP address 10.0.0.10

```sh
bastille create FAMP 14.1-RELEASE 10.0.0.10
bastille sysrc FAMP sshd_enable=YES
bastille service FAMP sshd start
# Redirect Port Conatiner to Host
#CMD RDR Container Protocol HostPort ContainerPort
bastille rdr FAMP tcp 80 80
```
Now apply FAMP template to container

```sh
bastille bootstrap https://github.com/asepready/famp-bastillebsd
bastille template FAMP asepready/famp-bastillebsd
```
if you want to apply template using another private IP address, you can do the following

```sh
bastille template FAMP asepready/famp-bastillebsd --arg server_ip=11.0.0.2
```
## License
This project is licensed under the BSD-3-Clause license.
