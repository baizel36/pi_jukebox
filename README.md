# **Building a Raspberry Pi Jukebox with Ansible**

## **Introduction**

This project automates the process of building a jukebox on a Raspberry Pi using Mopidy-Spotify. Before you read any further, this DOES require you to have a paid Spotify account.  

The idea for this solution came during a party at my brothers house.  We were shooting pool and he passed his phone around all evening so people could add songs to the Youtube music playlist.  I thought to myself, "There has to be a way to do this on a Raspberry Pi". Over the course of the next week, I looked for a solution.  It took several hours of research to get it consistently working smoothly. I wanted to share it and hopefully make getting your jukebox operational a little bit easier.    

My brother isn't a technical person.  I wanted a solution where all he had to do was plug in the Raspberry Pi and it would be functional.  That was the inspiration for this particular solution. Once you go through this set up once, making a second jukebox is really easy and fast. 

## **Set up your laptop**

You will need to install Ansible on your on your laptop.  I run Ubuntu on my laptop. With that being said, I am sure you can do this on Windows. I just don't know how.  I avoid Windows as much as possible.  

Ansible is easily installed in Debian Linux using:

```sudo apt-get install ansible```

When I put this project together.  My Ansible version was: 2.9.6.  That is the only software required on your PC for this project.

You can get a copy of my Ansible playbook code for this project on my github:

### **Initial configuration of the Raspberry Pi**

On the Raspberry Pi side of the house, this project was tested on Debian 10 Buster.  It likely will not function without modification on any other operating system.  For simplicity, I recommend you start with a clean Buster image.  Do not change the default user pi. That user is hardcoded into the script(s). 

If you need a copy of the Buster image you can source it from here:

https://pimylifeup.com/download-raspbian/#raspbianbuster

This is how you can check your operating system version: 

   ```cat /etc/os-release```

   ![os name](/ansible/README/markdown_images/os_name.png)

Enable SSH on the Raspberry Pi
   
   ```sudo raspi-config```
  
   ![raspi config](/ansible/README/markdown_images/raspi_config_cmd.png)
   ![raspi config](/ansible/README/markdown_images/raspi_config_int_options.png)
   ![raspi config](/ansible/README/markdown_images/enable_ssh.png)

The Raspberry Pi will need to have a static ip address assigned. We will set a static ip address by editing the /etc/dhcpcd.conf file. You can edit the file with your favorite text editor.  You will need to uncomment 3 lines in the section "Example Static IP configuration".  Define a static ip address, static router(default_gateway), and static domain servers with valid addresses from your network. The screenshot shows my configuration. I used two domain servers.  My router (192.168.1.1) and Cloud Flare(1.1.1.1) for redundancy.

   ```vi /etc/dhcpcd.conf```

   ![static ip](/ansible/README/markdown_images/dhcpcd.conf_config.png)

After editing the dhcpcd.conf file.  Connect to the wi-fi, and verify you have internet access before proceeding. After making one successful connection, the Raspberry Pi should auto connect on reboot. 

That completes the initial configuration of the Raspberry Pi.  Reboot the Raspberry Pi. After the reboot, make sure that you can SSH into the Raspberry Pi.  The first time you connect you will need to accept the host fingerprint.  The SSH command is:
   
   ```ssh pi@192.168.1.101```

   (substitute your statically defined ip)

   Once you have successfully connected via SSH.  You are ready to configure the Ansible playbook for deployment.

## **Gather Configuration Information**

You will need to go to this website and get authorization to use the Spotify API. You will need your Spotify credentials to get the API credentials. You can find your Spotify username by logging into your Spotify account.  I used the mobile app on IOS.  Go to: settings > Account to view your username.  The username will not be your human friendly name.  It will be long string of random characters.  You will need that username and your password to generate your API client_id and client_secret. You will need your username, password, client_id, and client_secret to use as variables in the Ansible playbook. 

   https://mopidy.com/ext/spotify/#authentication
   

The Raspberry Pi will be used as a dedicated jukebox.  The Ansible playbook will install an expect script that will automatically start the mopidy server and will also automatically connect to a designated bluetooth speaker everytime the Raspberry Pi is rebooted. You will need to have the MAC address of your bluetooth speaker.  

 ## **Set up Ansible for Deployment**

You will need to navigate to the group_vars folder inside the Ansible project and update the all.yml file with the information you gathered in the previous section.  The information does need to be inside quotes as pictured below. The ip address will be the one you statically defined on the Raspberry Pi.  The username,password,client_id,and client_secret are your Spotify credentials. The MAC address will be for your bluetooth speaker.

   ![group_vars](/ansible/README/markdown_images/group_vars.png)

You will also need to update the pi_host.ini file in the Ansible folder with your Raspberry Pi ip address.  

   ![group_vars](/ansible/README/markdown_images/pi_hosts.ini.png)

## **Deploy the Jukebox**

Now your ready to run the installation. Navigate to your Ansible folder. The below screenshot shows the location in my terminal.  You can also see the ansible structure in my VSCode file pane. 

   ![group_vars](/ansible/README/markdown_images/vscode.png)

Once you navigate inside your ansible directory issue the following command to begin the installation:

```ansible-playbook -i pi_hosts.ini spotify.yml -k -K```

When you run this command you will be prompted twice for the password you used when you set up the Raspberry Pi.The ansible-playbook will install all the necessary software, set all the permissions, and copy all the custom configuration files to the Raspberry Pi.  When completed, the playbook will reboot the Raspberry Pi. Have the bluetooth speaker in pair mode while the Raspberry Pi reboots.  When the reboot is completed it will automatically connect to the speaker and start the mopidy server.  The expect script does run slow on reboot, so allow approximately two minutes for the script to fully complete. 

After the reboot is complete, type the following address into your webbrowser search bar:  

192.168.1.101:6680 (Substitute your IP if it was not 192.168.1.101).  

The screenshots below were taken on my PC for ease of copy and pasting.  The pages display much nicer on a mobile platform.  However, it is fully functional on Firefox and Chrome on a PC.  

You should see the following: 

   ![group_vars](/ansible/README/markdown_images/mopidy1.png)

You will click the link to "mobile" and you will now have the user interface:

   ![group_vars](/ansible/README/markdown_images/mopidy2.png)

Now anytime you have a party, anyone connected to your wifi network can access the jukebox. They can add songs to your party playlist from their own phone! I usually text my friends a link to page so they don't have to type in the address.  When they open the page the first time, I have them save the page to their favorites or home screen so its easily accessible. I create a community playlist and have every body add their song(s) to the playlist.

Now if your friends ask you to make them a jukebox, you have an automated solution to make it easy.  I hope someone finds this useful.

In a future post, I will do a walk through of the Ansible structure and operation.

If you have questions, or need assistance, please feel free to e-mail me:  baizel36@gmail.com
