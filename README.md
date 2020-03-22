# Setup VPN Server (OpenVPN Server) in Linux CentOS

###### This tutorial works great on CentOS 7.x, if it somehow didn't work on CentOS 8.x ( or above if you are coming from the future :) ), you will have to do some workarounds.

###### Commands:
###### First installation of Needed Libraries and Programs

> yum -y install epel-release

> yum -y install openvpn easy-rsa

> yum -y install nano

###### Copying and editing openvpn config file
> cp /usr/share/doc/openvpn-*/sample/sample-config-files/server.conf /etc/openvpn/

> nano /etc/openvpn/server.conf

###### Using Ctrl+W search short key: look for these and uncomment them (by removing ; semicolon)
###### #uncomment bellow
> topology subnet

> push "dhcp-option DNS 208.67.222.222" # change dns to whatever you want

> push "dhcp-option DNS 208.67.222.222" # change dns to whatever you want

> user nobody

> group nobody

###### #comment this
> ;tls-auth ta.key 0

###### #optional uncomment # if you want your clients to be able to see each other, useful for offices or companies
> client-to-client

###### Then Ctrl+X to Exit nano, Press Y to save then enter to overwrite
###### Now

> cd /usr/share/easy-rsa/

> ls

###### check which version exists, for this tutorial, easy-rsa version is 3.0.6 if it is changed (updated, got higher version, you can use the higher version)
> cd 3.0.6

> ./easyrsa init-pki

> ./easyrsa build-ca nopass

> // Leave blank, press enter

> ./easyrsa gen-req server nopass

> // Leave blank, press enter

> ./easyrsa gen-req client nopass

> // Leave blank, press enter

> ./easyrsa sign-req server server nopass

> yes

> ./easyrsa sign-req client client nopass

> yes

> ./easyrsa gen-dh

###### Then you wait for awhile, depends on the Computer Hardware Specs
> cd pki

> pwd

###### copy the path to use it afterwards: /usr/share/easy-rsa/3.0.6/pki
> nano /etc/openvpn/server.conf

###### Using Ctrl+W search short key: look for these and change them:
> ca ca.crt

> cert server.crt

> key server.key


###### Change them to:
> ca /usr/share/easy-rsa/3.0.6/pki/ca.crt

> cert /usr/share/easy-rsa/3.0.6/pki/issued/server.crt

> key /usr/share/easy-rsa/3.0.6/pki/private/server.key


###### Using Ctrl+W search short key: look for "dh20" and change:
> dh dh2048.pem

to

> dh /usr/share/easy-rsa/3.0.6/pki/dh.pem

###### Make sure the 3.0.6 version is modified if "easy-rsa" version is changed in the future.

###### Then Ctrl+X to Exit nano, Press Y to save then enter to overwrite

###### Then we enable ip forwarding
> sysctl -w net.ipv4.ip_forward=1

> echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf

###### We install firewall if not already installed, then we configure it
> sudo yum -y install firewalld

> systemctl start firewalld

> systemctl status firewalld

> sudo firewall-cmd - set-default=trusted

> firewall-cmd - permanent - zone=trusted - add-masquerade

> firewall-cmd - permanent - add-service openvpn

> firewall-cmd - reload

> firewall-cmd - list-all


###### Sometimes openvpn asks for service.conf so we:
> cp /etc/openvpn/server.conf /etc/openvpn/service.conf

###### We start openvpn
> systemctl start openvpn@server

> systemctl enable openvpn@server

###### Then to create Clients
###### Use the Script i wrote to generate client keys very easily

