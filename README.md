# Troubleeshooting-Of-Ec2-
Troubleeshooting Of Ec2 that created using Userdata

What is an EC2 Instance?
EC2 stands for Elastic Cloud Compute. It is a compute service where users can rent virtual machines or instances to run their own applications. With EC2, you can launch as many servers as needed.

What is User Data?
User Data is a metadata field of an EC2 instance that allows custom scripts or commands to run automatically when the instance is launched.

In This Case:

I created an EC2 instance in the US-East region using default settings. In the advanced settings, under User Data, I added a script to run at the time of instance launch.

Troubleshooting Steps:

I encountered two issues with this EC2 instance, which I have since resolved:

Issue: HTTP Server Not Accessible on EC2 Instance

Problem Description: I launched an EC2 instance using a user data script to install and configure an Apache HTTP server. Despite the successful launch,
I couldn’t access the web server using the public IP address of the instance.

Steps to Reproduce:

Launched an Amazon Linux 2 EC2 instance.

Used the following user data script to install and start Apache:

#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd

echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html

Attempted to access the instance via http://<instance-public-ip> but received no response.

Expected Behavior:
When navigating to the public IP of the EC2 instance in a browser, I expected to see a webpage displaying the text: "Hello World from <hostname>."

Actual Behavior:
The browser returned a timeout or connection error, indicating that the HTTP server was unreachable.

Root Cause:
The issue occurred because the security group attached to the EC2 instance did not have the appropriate inbound rules for HTTP (port 80) and HTTPS (port 443) traffic.

Solution:
Navigated to the EC2 security group settings.

Added the following inbound rules:

HTTP (port 80) from anywhere (0.0.0.0/0)
HTTPS (port 443) from anywhere (0.0.0.0/0)
After adding these rules, I was able to access the EC2 instance's web server successfully by visiting http://<instance-public-ip>.

Secondary Issue: HTTPS Not Accessible

Problem Description: After resolving the initial issue with HTTP access, I faced a new problem where I couldn’t access the server via HTTPS (port 443). The browser did not establish a secure connection, and I received an error indicating that the site was not secure.

Troubleshooting Steps:

Connecting to the Instance:

Used PuTTY to connect to the EC2 instance using the public IP address.
Checking Port Statuses:

Executed the following commands to check if ports 22 (SSH) and 443 (HTTPS) were listening:

sudo netstat -tuln | grep 22
sudo netstat -tuln | grep 443
The output confirmed that port 22 was open and listening, but port 443 was not.

Identifying the Issue:
The absence of port 443 listening indicated that the Apache server was not configured to handle HTTPS traffic.

Installing SSL Module:
To enable HTTPS support, I installed the mod_ssl package, which provides SSL/TLS support for Apache:
sudo yum install -y mod_ssl

Restarting Apache:
After installing mod_ssl, I restarted the Apache HTTP server to apply the changes:
sudo systemctl restart httpd

Verifying HTTPS Access:

Checked the server again using https://<instance-public-ip>.
The server was now accessible over HTTPS, confirming that the SSL module was correctly installed and configured.

Limitations:

Although HTTPS access was now functional, the connection was not fully secure because a valid TLS certificate had not been configured. Without a TLS certificate, browsers will show a warning that the connection is not secure.

Next Steps:

To ensure a fully secure connection, obtain and configure a valid TLS certificate for the server. This will prevent browser warnings and provide a secure HTTPS connection.
