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
# Create and Setup Clients
###### Use the shell script file i wrote to generate clients keys very easily
###### Download it from this github repo: OpenVPNClientsKeysGenerator.sh

###### to Download it from CentOS you will need to Programs
> yum -y install wget unzip

###### Then you download with wget and unzip
> wget https://github.com/TarikSeyceri/Setup-VPN-Server-OpenVPN-Server-in-Linux-CentOS/archive/master.zip
> unzip -qq master.zip && rm -rf master.zip
> cd Setup-VPN-Server-OpenVPN-Server-in-Linux-CentOS-master

> nano OpenVPNClientsKeysGenerator.sh
###### Modify 'server_static_ip_address' variable to work with your Server's IP Address
###### If easy-rsa version is changed?, make sure you change it in 'path_to_rsa' variable

###### To autherise the file to be executed
> sed -i -e 's/\r$//' OpenVPNClientsKeysGenerator.sh
> sudo chmod +x OpenVPNClientsKeysGenerator.sh

###### Then you can run it with
> ./OpenVPNClientsKeysGenerator.sh

###### Follow the instructions in the Script
###### It will only ask for the client username, make sure it is unique
###### a folder has been created with the client username you wrote in the path: /root/Documents/, provides THE_CLIENT_USERNAME.ovpn and the needed keys and certs to be used for VPN Client Programs, if you want to use OpenVPN Client (Which is recommended), for Windows download it from here: 
> https://openvpn.net/community-downloads/
###### For Other OS OpenVPN or Other VPN Client Programs ( Google it :) )

# Setup OpenVPN Client in Windows
###### Download the THE_CLIENT_USERNAME.ovpn file from the server and send it to the Client Computer
###### The Client Computer should download: https://openvpn.net/community-downloads/ and install Next - Next - Next
###### Open the location of the OpenVPN Client after installation => From Desktop => OpenVPN GUI => right click => Properties => Open File Location
###### Go back one level up, then go to config folder: the path should be something like: C:\Program Files\OpenVPN\config
###### Copy the THE_CLIENT_USERNAME.ovpn inside the config folder then close the window
###### from the Desktop run OpenVPN GUI, from the TaskBar you will see the OpenVPN icon, right click => connect. Done.
###### Enjoy!

