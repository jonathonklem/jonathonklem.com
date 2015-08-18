---
layout: post
title: OpenVPN Adventures
date: 2014-11-28 03:50:11.000000000 -06:00
categories:
- Networking
tags: []
status: publish
type: post
published: true
---
I recently had the pleasure of configuring a VPN between a data center in Virginia, one in California, and an Amazon VPC.  The fun part was that we had to do this with no physical access to the machines and without having admin access to the switches that the servers in the data centers were behind.  With the limitations we had, we decided to use OpenVPN.  OpenVPN is open source and uses a custom protocol that uses SSL to share keys and is actually pretty easy to work with.  This article will focus on setting up a VPC and a VPN server that will allow you to connect to that VPC.

## Setting Up The Network

First, we have to create our VPC.  Inside the AWS management console, navigate to Services-&gt;VPC and inside the VPC dashboard select "Your VPCs".  Next we'll click "Create VPC": 

![Create VPC](https://jonathonklem.com/assets/images/create-vpc-e1417043038127.png)

The first option, "Name tag" is just used as a reference and can be anything you want.  The CIDR block will define our network.  In the example that they give: 10.0.0.0/16, this will create a network with valid IP addresses ranging from 10.0.0.1 - 10.0.255.255 (a 16 bit subnet, or a network mask of 255.255.0.0).  For this example we're going to go with '10.8.0.0/16' as our CIDR block.

Next, we're going to have to add a gateway to our VPC so our packets can reach the outside world.  We do this by selecting "Internet Gateways" in the left panel of the VPC Management Console and selecting "Create Internet Gateway".  Here, we're only presented with one option, and that's the name of our VPC:

![Create Gateway](https://jonathonklem.com/assets/images/create-gateway-e1417043077318.png)

In this example we'll call our Gateway "Test VPN Gateway".  After it's created we'll have to attach it to our VPC.  To do this, we'll right click on the gateway and select "Attach To VPC".  In this next screen there's a drop down with a list of our active VPCs:

![Attach To VPC](https://jonathonklem.com/assets/images/attach-to-vpc-e1417042970382.png)

Lastly we'll create a subnet within our VPC.  We have a 16 bit network that we could divide up however we see fit, for this example though we'll make the subnet be the entire network, so 10.8.0.0/16.  We'll name it "TestVPN Subnet".

![Create Subnet](https://jonathonklem.com/assets/images/create-subnet-e1417042889555.png)

Now we must associate our route with our subnet.  When you created the VPC a route was automatically created for you, but when you created your subnet it wasn't automatically associated with the route table.  In the VPC dashboard click "Route Tables", select the "Subnet Association" tab, click "edit" and then make sure your subnet is checked:

![Attach your route table](https://jonathonklem.com/assets/images/route-table.png)

Next we need to make sure that we're routing to our gateway.  Make sure that "0.0.0.0/0" is being routed to our internet gateway.  This can be modified in the 'Routes' tab:

![Create Gateway Route](https://jonathonklem.com/assets/images/create-gateway-route.png)

Now we have our VPC all set up.  Next we'll create two EC2 instances inside of our VPC.  From the EC2 dashboard, select "Launch Instance" and go through the setup just like any other instance.  You can select whichever instance type you like, I'm going with micro for this article.  The main part to pay attention to is when you get to stage 3.  You want to make sure that you've selected the proper VPC and Subnet.  Also for the line that says "Auto-assign Public IP", be sure to set this to 'Enabled' for the VPN server instance and 'Disabled' for the other server:

![EC2 Setup](https://jonathonklem.com/assets/images/ec2-setup.png)

**Of the utmost importance is to disable source/destination checking on the VPN Server and enable IPv4 Forwarding.**  To enable IPv4 Forwarding edit /etc/sysctl.conf and uncomment the line that says "net.ipv4.ip_forward=1" and issue the following command:

```
/sbin/sysctl -w net.ipv4.ip_forward=1
```

To disable source/destination checking right click the EC2 instance, select "Change Source/Dest. Checking" and make sure to disable.  This will allow the VPN server to actually route the packets, and will let you avoid any hairpin NATing issues.

![Disable Source and Destination Checking](https://jonathonklem.com/assets/images/disable-source-dest-check-e1417042744126.png)

## Setting Up The OpenVPN Server

First, we'll connect to the ec2 instance we've designated as our VPN server and install OpenVPN and easy-rsa (a lot of this information was adapted from <a title="OpenVPN Guide" href="https://help.ubuntu.com/community/OpenVPN">this guide</a>):

```
sudo apt-get install openvpn easy-rsa
```

Next we'll copy the Easy RSA files into our OpenVPN directory:

```
sudo apt-get install easy-rsa
sudo cp -R /usr/share/easy-rsa/ /etc/openvpn/
```

After that, we'll edit our vars file to set up some default settings for easy-rsa:

```
sudo vim /etc/openvpn/easy-rsa/vars
```

We're looking for the following lines (change the values to what you need):

```
export KEY_COUNTRY="US"
export KEY_PROVINCE="IN"
export KEY_CITY="Evansville"
export KEY_ORG="JonathonKlem"
export KEY_EMAIL="jonathonklem@gmail.com"
export KEY_OU="MyOrgUnit"
```

Next we need to generate our Diffie Hellman key, our certificate authority, and our server key.  If you would like a more detailed explanation of these commands refer to the aforementioned article:

```
sudo -s
cd /etc/openvpn/easy-rsa
source ./vars
./clean-all
./build-dh
./build-ca
./build-key-server server
cp keys/server.key keys/server.crt keys/ca.crt keys/dh2048.pem ..
```

Next we'll edit our server.conf which will be located in /etc/openvpn.  Here is a simplified configuration file.  Edit the appropriate values but if you've been using the same commands used throughout this article there won't be much to edit.

```
# We want to be in server mode
mode server
# We're going to use a tunnel, rather than a tap interface
dev tun
# Use UDP not TCP
proto udp
# Try to preserve some state across restarts.
persist-key
persist-tun
# Point OpenVPN to all of our necessary key files
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
# Enable compression on the VPN link.
comp-lzo
#specify our server's address
server 10.9.0.0 255.255.0.0

#make sure clients can access our internal network
push "route 10.8.0.0 255.255.0.0"
#set log verbosity to 5 for easier debugging
verb 5
```

Next we'll start the server:

```
/etc/init.d/openvpn start
```

When you run `ifconfig` you should see there is a new tun0 interface.  If you do not see this new interface, run `cat /var/log/syslog | grep vpn` to see any error messages OpenVPN may have spat out.  Most likely it won't start because a certificate file name was misspelled or not copied to the proper location.  You'll want to make sure that UDP port 1194 is accessible.  So go back to the ec2 dashboard and edit the VPN server's security group to open that port:

![Allow UDP Port 1194](https://jonathonklem.com/assets/images/udp-rule.png)

You can create whatever size subnet you like for your VPN configuration.  I chose to go with a 16 bit subnet for this article.  The most important part is that you do not want the networks to overlap, so our VPC is 10.8.0.0/16 and our VPN is 10.9.0.0/16.  Next we'll want to edit our routing table, and make sure that all packets headed to our VPN network go through our VPN server.  To do that we'll go back to our VPC dashboard and edit the route table just like when we added the default route to the gateway like so (i-6abfd58b is the instance ID of our VPN server):

![Add Your VPN Route](https://jonathonklem.com/assets/images/add-vpn-route.png)

While we're in AWS, we want to make sure to enable ICMP traffic to our other host in our VPC (the one that's not the VPN server) to test that our connection works, do this the same way we enabled UDP port 1194 for the VPN server.

## Connecting To Network

To connect to the network we'll need to generate a key for each client.  In this example we'll just set up one.  Make sure that you're in the director /etc/openvpn/easy-rsa and run the following

```
source ./vars
./build-key client
```

You'll be presented with a series of prompts.  You can chose to use the default values and reply 'y' for yes where appropriate.  Afterwards you'll have a client.crt and a client.key in the /etc/openvpn/easy-rsa/keys directory.  Every client that you have connect to your network will have to have ca.crt, and their crt and key file pair to authenticate.

When we have everything set up we can connect to the network.  This is a very easy process for both Windows and Linux systems.  For both Operating Systems, the process involves creating a configuration file and then running the OpenVPN software.

First you must download the software.  In Ubuntu this can be done very simply with the same command we ran on the server:

```
apt get install openvpn
```

For Windows you can visit the [OpenVpn Website](http://openvpn.net/index.php/download/58-open-source/downloads.html) to download the installer.  The installer is very straight forward and doesn't require any complex configuration to get going.

In both Windows and Linux the setup is going to be identical, for both systems we'll make sure to get the client.crt, client.key, ca.crt to the proper directory and we'll also include a configuration file.  For windows this will be named client.ovpn and in linux it will be named client.conf.  The default directory to place these files in a linux system is /etc/openvpn and for windows it's C:\Windows\Program Files\OpenVPN (x86)\config.

For both systems the configuration file will look like the following:

```
# Specify that we are a client and that we
# will be pulling certain config file directives
# from the server.
client

# Use the same setting as you are using on
# the server.
# On most systems, the VPN will not function
# unless you partially or fully disable
# the firewall for the TUN/TAP interface.
dev tun

# Are we connecting to a TCP or
# UDP server?  Use the same setting as
# on the server.
;proto tcp
proto udp

# The hostname/IP and port of the server.  You'll have to change this to the public IP address of your VPN server
remote X.X.X.X 1194


# Keep trying indefinitely to resolve the
# host name of the OpenVPN server.  Very useful
# on machines which are not permanently connected
# to the internet such as laptops.
resolv-retry infinite

# Most clients don't need to bind to
# a specific local port number.
nobind

# Try to preserve some state across restarts.
persist-key
persist-tun

# Our key files
ca ca.crt
cert client.crt
key client.key

# Enable compression on the VPN link.
# Don't enable this unless it is also
# enabled in the server config file.
comp-lzo
```

Once we have the necessary files in place, to start your VPN client in Windows you'll run the OpenVPN client, then right click on the system tray icon and click 'connect'. On a Linux system we'll do the following:

```
/etc/init.d/openvpn start
```

Once we've started the VPN client, you'll be notified in windows once it connects successfully. On a linux system you'll want to run `ifconfig` to make sure the interface comes up. If there are any issues, the most common problems are permissions issues. The client.key file should not be group or world readable. You can verify that the connection works by pinging the VPC host.

You now have a VPC that you can VPN in to! If you have any questions or comments you can email me at [jonathonklem@gmail.com](mailto:jonathonklem@gmail.com).
