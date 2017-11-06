# THIS IS A WORKS IN PROGRESS (WIP) AND NOT READY YET FOR USE. The intention here is to (Docker/Vagrant) automate 'Out of The Box' support for OpenVPN / OpenVZ deployment on an EC2 instance to fully support "Multicast" on AWS for HPC Computing Applications such as GPGPU Raytracing and much more

# openvpn-openvz-multicast
OpenVPN - OpenVZ - Multicast Enabled and Supported for GPGPU Workflows

Using a VPN Server to Connect to Your AWS VPC for Just the Cost of an EC2 Nano Instance

Source: http://computer.howstuffworks.com/vpn3.htm and https://hackernoon.com/using-a-vpn-server-to-connect-to-your-aws-vpc-for-just-the-cost-of-an-ec2-nano-instance-3c81269c71c2
AWS has an awesome firewall built into its core services which can easily be used to make sure that only certain ports are open to the outside world. One extra step that we can take is to run a VPN Server that serves as the gateway to our protected EC2 instances. We can then shutdown direct SSH access to our EC2 instances and also have the freedom to block access to our entire network just by revoking access via our VPN Server. The later is very useful if you need to revoke access for a former employee.
The following tutorial will take you through the steps of setting up an EC2 instance that will run the OpenVPN Server. It will then cover how to grant and revoke access through the VPN Server.
Step 1— Create the VPN Security Group
Overview: security groups allow your servers to communicate with each other in a private cloud while exposing specific ports to the world. We are going to create a security group to allow VPN access to our VPN Server. We will assume that all your other EC2 instances are members of the default security group and that the default security group does not allow access from the outside world.
Log in at https://aws.amazon.com, type EC2 in the search box and click on the target to go to the EC2 Dashboard.
From the EC2 dashboard, click Security Groups

Click Create Security Group

Enter a name and description of vpn and specify inbound rules on ports 22, 443, 943 and 1194. Note: the protocol for port 1194 is UDP.

Note: if the IP addresses that your team uses are static then you can add yet another layer of security by specifying that IP address range in the Source of your rules. However, you’ll want to leave the Source blank if you want your team to be able to connect from different IPs as they may be working from a hotel, home, cafe, etc…
Step 2 — Create the EC2 Instance
Return to the EC2 Dashboard and then click Launch Instance

Select Ubuntu (you can of course select almost any other OS that runs OpenVPN, but this tutorial is tailored for Ubuntu)

Select t2.nano and click Review and Launch

On the next screen, click Edit security groups

Select the vpn and default security groups and click Review and Launch

Click Launch, choose your key pair and then click Launch Instances
Step 3 — Disable Source/Destination Check
From the list of instances, select the VPN instance and then Networking->Change Source/Dest. Check from the drop down menu. Then click Yes, Disable. This is needed as otherwise, your VPN server will not be able to connect to your other EC2 instances.

Step 4— Create an Elastic IP Address
Overview: when an EC2 instance is stopped and restarted, the Public IP address changes. We want the IP address of our VPN Server to remain static so we’ll use an Elastic IP Address.
From the E2c Dashboard, select Elastic IPs

Click Allocate new address

Click Allocate and then Close.
Make a note of your Elastic IP address as this will be the Public IP Address of your VPN Server.
Then select the Elastic IP and click Associate address from the drop down menu.

Select the EC2 instance you just created and click Associate.

Step 5— Install and Configure the OpenVPN Server
SSH into your VPN server:
$ ssh ubuntu@PUBLIC-IP-OF-VPN-SERVER
Download our helper scripts and set up a default config:
$ git clone https://github.com/redgeoff/openvpn-server-vagrant
$ cd openvpn-server-vagrant
$ cp config-default.sh config.sh
Edit config.sh and enter in your configuration. Note: PUBLIC_IP should be equal to the Elastic IP Address that you created above.
$ nano config.sh
Switch to root
$ sudo su -
Update Ubuntu and install OpenVPN. Note: you will be prompted twice and when you do, select Keep the local version currently installed
$ /home/ubuntu/openvpn-server-vagrant/ubuntu.sh 
$ /home/ubuntu/openvpn-server-vagrant/openvpn.sh
At this point, the OpenVPN Server is running.
Step 6 — Add the Route
Routes must be added to the server so that your team’s clients know which traffic to route to the VPN Server.
You can determine the proper subnet by returning to your list of EC2 instances, clicking on a target instance and identifying the Private IP.

Your network will be the first 2 parts of the Private IP appended with zeros, e.g. 172.31.0.0
On the VPN Server edit /etc/openvpn/server.conf and add something like the following:
push "route 172.31.0.0 255.255.0.0"
Then restart the VPN Server with:
$ systemctl restart openvpn@server
Step 7— Grant Access to Your VPN
Note: We assume that you are still SSH’d into the VPN and logged in as root.
Run the following command and be sure to replace client below with a unique name for your user/client.
$ /home/ubuntu/openvpn-server-vagrant/add-client.sh client
You’ll then find a configuration file at
~/client-configs/files/client-name.ovpn
You will want to provide this file to the individual on your team who will be connecting to your VPN. SCP is handy for downloading this .ovpn file from your VPN Server.
Your team can use one of various VPN clients such as Tunnelblick (OS X) and OpenVPN (Linux, iOS, Android and Windows). After installing one of these clients they should be able to set up the VPN config just by double clicking on the .ovpn file.
Note: once connected to the VPN, your users will want to use the Private IPs of your EC2 instances. You’ll probably want to use Route 53 to create subdomain records that route to the Private IPs.
Step 8— Revoke Access to Your VPN
Note: We assume that you are still SSH’d into the VPN and logged in as root.
Run the following command and be sure to replace client below with a unique name for your user/client.
$ /home/ubuntu/openvpn-server-vagrant/revoke-full.sh client

# Phase Two
https://serverfault.com/questions/749909/openvpn-openvz-multicast-and-how-to-enable-it

# NOTE: All of the above already works but is a manual setup that is subject to human error. The WIP to convert the above steps into a Docker deployment to save time and more importantly ensure that setup is configured properly. 
