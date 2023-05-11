# Run Minecraft server on EC2 instance in AWS

This tutorial describes one of possible approaches on how to run own Minecraft server in AWS on EC2 instance.

## Infrastructure in AWS

### Log in to AWS

Use the following link to open AWS console [aws.amazon.com/console/](https://aws.amazon.com/console/) and click on `Sign In to the Console`

<img src="/images/aws_login_1.png" width="50%" height="50%">

Select `IAM user`, enter `AWS account alias` `ehu-esde-lr` and click on `Next`

<img src="/images/aws_login_2.png" width="389" height="468">

Enter `IAM user name` and `Password`. The user name format is `firstname.lastname`. Click on `Sign In`

<img src="/images/aws_login_3.png" width="382" height="473">


After successful login, first of all select an AWS region where you are planning to create infrastructure for your Minecraft server, e.g. `us-east-1 (N. Virginia)`

<img src="/images/aws_region.png" width="264" height="47"> 

Then you need to select EC2 console to proceed with EC2 stuff

<img src="/images/select_ec2_console.png" width="50%" height="50%">

Your EC2 console should look similar to this image

<img src="/images/ec2_console.png" width="50%" height="50%">

Before you can go ahead and launch our EC2 server for Minecraft you need to create a SSH key pair that will be used for authentication by accessing to your via SSH and a Security Group that will allow you to connect to your EC2 instance by SSH(TCP 22 port) and Minecraft server (TCP 25565 port)

### SSH Key pair

In the left panel of the EC2 console find `Network & Security` -> `Key pairs` and click on it.

<img src="/images/find_key_pairs.png" width="234" height="248">

In the upper right corner of the central panel you can find `Create key pair` and click on it.

<img src="/images/create_key_pair.png" width="339" height="85"> 339 × 85

If you use Linux or MacOS on your workstation select Private key file format `.pem` and if you use Putty on Windows select `.ppk`

<img src="/images/key_format.png" width="181" height="125">

Enter `Name`  `minecraft` and click on `Create key pair`

<img src="/images/key_pair_params.png" width="50%" height="50%">

The private key `minecraft.pem` will be downloaded to your browser. On Linux or MacOS copy it to /home/<user_home>/.ssh and restrict privileges to RW(600)

```bash
cp ~/Downloads/minecraft.pem ~/.ssh/minecraft.pem
chmod 600 ~/.ssh/minecraft.pem
```

### Security group

In the left panel of the EC2 console find `Network & Security` -> `Security Groups` and click on it.

<img src="/images/find_security_groups.png" width="196" height="234">

In the upper right corner of the central panel you can find `Create security group` and click on it.

<img src="/images/create_security_group.png" width="222" height="95">

In `Basic details` section fill in `Security group name` `minecraft`, `Descrition` `Allow access to SSH and Minecraft server` and select VPC where you are planning to create your EC2 instance.

<img src="/images/sg_basic_details.png" width="50%" height="50%">

In `Inbound rules` section click on `Add rule` twice to add new inbound rules for SSH and Minecraft server. Your IP address you can find using this website [whatismyipaddress.com](https://whatismyipaddress.com/)

Fill in rules like you see on the picture using `your own IP` as `Source`
 
<img src="/images/sg_inbound_rules.png" width="50%" height="50%">

As soon as all inbound rules are created click on `Create security group` at the bottom of the page.

<img src="/images/create_sg.png" width="234" height="81">

### EC2 instance

Now, when you have SSH key pair and Security Group created, you can configure and launch EC2 instance.

In the left panel of the EC2 console find `Instances` -> `Instances` and click on it.

<img src="/images/find_instance.png" width="223" height="322">

In the upper right corner of the central panel you can find `Launch instances` and click on it.

<img src="/images/launch_instances.png" width="225" height="77">

Fill in `Name` `minecraft` in `Name and tags` section

<img src="/images/ec2_name_tag.png" width="50%" height="50%">

Select the `Ubuntu` tile in `Application and OS Images` section by clicking on it.

<img src="/images/ami_selection.png" width="50%" height="50%">

Select the `t3.small` in `Instance type` section

<img src="/images/ec2_type.png" width="50%" height="50%">

Check in to `Select existing security group` in `Network settings` section and select `minecraft` from `Security groups` dropbox. If you can't find `minecraft` in the list, click on `Edit` to select the correct VPC

<img src="/images/ec2_network_settings.png" width="50%" height="50%">

Set `10Gb` and `gp3` type for root volume in `Configure storage` section

<img src="/images/ec2_storage.png" width="50%" height="50%">

Select `IAM Instance Profile` `MinecrafConnct` in `Advanced details` section.

<img src="/images/ec2_iam_role.png" width="50%" height="50%">

To finish EC2 instance creation click on `Launch instance` in the right panel.

<img src="/images/ec2_launch_instance.png" width="376" height="97">

## Minecraft server

### SSH conection

Now you need to get SSH access to EC2 instance with the SSH key that you created before. First you should find EC2 insatnce public IP.
In the left panel of the EC2 console find `Instances` -> `Instances` and click on it.
Find your instance in the list and click on `Instance ID`, then find IP addess in `Public IPv4 address` section

To open SSH session to your EC2 instance you can use EC2 Instance Connect provided by AWS

Select your EC2 instance and click on `Connect` at the top of the EC2 console

<img src="/images/ec2_connect.png" width="50%" height="50%">

In the next window select `EC2 Instance Connect` tab and then click on `Connect` to start SSH session

<img src="/images/ec2_instance_connect.png" width="50%" height="50%">

`Optionally`, you can use SSH client on your PC to open SSH session, use the following command:

```bash
ssh -i ~/.ssh/minecraft.pem  ubuntu@<server_IP>
```

### Install Java

Now we need to install Java JDK to the instance because we will use Java for running Minecraft server later on

Ubuntu uses APT package manager, so first of all we need to update the list of packages and then install `openjdk-18-jre-headless` package

```bash
sudo apt update
sudo apt install openjdk-18-jre-headless
```
to ensure that Java JDK was installed correctly run `java -version`

### Install Minecraft

Let's prepare a working directory for Minecraft and grant privileges to `ubuntu` user to use it

```bash
sudo mkdir -p /opt/minecraft
sudo chown ubuntu:ubuntu /opt/minecraft
cd /opt/minecraft
```

Next you need to download Minecraft server, more details you can find on the site [www.minecraft.net/en-us/download/server](https://www.minecraft.net/en-us/download/server)

```bash
sudo wget https://piston-data.mojang.com/v1/objects/8f3112a1049751cc472ec13e397eade5336ca7ae/server.jar -P /opt/minecraft/
```

So now you are ready to make the first run of the Minecraft server to initialize it. During initialization the server will create all necessary files in the working directory. Before you can use Minecraft server you also need to agree with terms of End-User License Agreement(EULA) in `eula.txt` file.

```bash
java -Xmx1024M -Xms1024M -jar server.jar nogui
sed -i "s/eula=false/eula=true/" eula.txt
java -Xmx1024M -Xms1024M -jar server.jar nogui
```

As soon as you see `Done (109.201s)! For help, type "help"` message in the log you can consider that Minecraft server is up and running and you can try to connect to it using `public_IP:25565`

To stop the server just type `stop` command in the console or use key combination `Ctrl+C`


## Create Minecraft service with SystemD

In order to garanty that your Minecraft server is always up and running even after EC2 instance reboot it is nice to use SystemD to to automatically start and re-start Minecraft servers

Place the `minecraft.service` file in your `/etc/systemd/system/` directory

```bash
sudo nano /etc/systemd/system/minecraft.service
```

Copy the following configuration to the file `minecraft.service` and save it using key combination `Ctrl+X`

```
[Unit]
Description=Minecraft Server
Wants=network-online.target
After=network-online.target

[Service]
# Ensure to set the correct user and working directory (installation directory of your server) here
User=ubuntu
WorkingDirectory=/opt/minecraft

# You can customize the maximum amount of memory as well as the JVM flags here
ExecStart=/usr/bin/java -Xmx1024M -Xms1024M -jar server.jar nogui

# Restart the server when it is stopped or crashed after 30 seconds
# Comment out RestartSec if you want to restart immediately
Restart=always
RestartSec=30

# Alternative: Restart the server only when it stops regularly
# Restart=on-success

# Do not remove this!
StandardInput=null

[Install]
WantedBy=multi-user.target
```

Enable the service using `sudo systemctl enable minecraft` command

Start the service using `sudo systemctl start minecraft`

Now you can check the service status using `sudo systemctl status minecraft` or `sudo journalctl -xeu minecraft.service`