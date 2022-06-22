# Network-Reconnaissance-with-Kismet
An independent, ethics-first learning project by Sean Lehey.

## **Disclaimer**
This project is intended to demonstrate my learning process using Kismet. Please note that the scope of this project is simply limited to reconnaissance and identification of nearby SSIDs. No attempt was made to join any networks, circumvent any security measures, or take adverse action on network devices. Ethics are my foremost priority while learning networking tools, so MAC addresses, SSIDs, and other identifying information will be censored to respect the privacy of these networks and devices.

## **Tools Used**

### **Hardware**
- 1x Alfa AWUS036ACH Wireless Network Adapter

### **Software**
- Kali Linux running on Oracle Virtualbox    
- Linux Command Line
- Aircrack-NG  
- Kismet  


## **Acknowledgements**
This video by SecurityFWD which introduced me to Kismet: https://www.youtube.com/watch?v=Qs9xPmUqzHI

Rogue Vault's WiFi adapter troubleshooting guide found here: https://www.youtube.com/watch?v=aXagM9G_nFE


## Part 1: Setting up the Wireless Network Adapter
Note: This section assumes we already have a Kali Linux image loaded into Virtualbox, as well as a compatible wireless network adapter. If we're running a Linux distribution without the appropriate tools installed by default, we'll have to install Kismet, Aircrack-NG, and Net-Tools to proceed.

### Installing the Chipset Drivers
First, we need to launch Kali Linux via Virtualbox and open up the terminal.

Next, we'll want to install the drivers for our network adapter's chipset. Our adapter uses the following package. This can be done through a simple CLI command:

`sudo apt install realtek-rtl88xxau-dkms`

![InstallDrivers](https://user-images.githubusercontent.com/107720480/174893689-2321bb0c-fe22-42d3-88d0-212cef4715c1.png)

Press enter to install the drivers, then plug our adapter into the computer.

Next, in Virtualbox, click *Devices*, *USB*, then select the chipset associated with our network adapter. Remember from the driver installation step that our device uses a Realtek chipset.

![Chipset Selection](https://user-images.githubusercontent.com/107720480/174895424-2bba50ca-d9d6-4d9d-b5b7-0b475e5da627.png)


### Setting the Adapter to Monitor Mode

It could be the case that you have multiple wireless network adapter cards. Let's use the terminal to identify which one we want to be working with by typing the following:

`sudo ifconfig`

![ifconfig](https://user-images.githubusercontent.com/107720480/174899710-26a8f1f6-c357-4afc-a5c5-5d3dad022ff7.png)

We can see from the screenshot that we only have one wireless network adapter indicated by *wlan0*. This is the card we will be working with.

Let's use airmon-ng to confirm that the drivers installed successfully and that our card can be run in monitor mode:

`sudo airmon-ng`

![airmon-ng](https://user-images.githubusercontent.com/107720480/174901436-df735093-8add-4b13-9864-86669cf3f351.png)

We can see *wlan0* listed, so it's time to launch Kismet!


## Part 2: Scanning for Nearby Networks

### Launching Kismet

So far we've been using CLI commands in the Linux terminal. Let's access Kismet's GUI to make this process much easier on the eye.

First, use the following command in the terminal:

`kismet -c wlan0`

This will start Kismet using your wireless network adapter in monitor mode. There will be quite a lot of text to parse through in the CLI, though, so let's move to a more organized GUI.

Go to the web browser (Firefox in this case) and type the following into the URL field:

`localhost:2501`

Create a memorable username and password, then click *Log In*.

![kismetlogin](https://user-images.githubusercontent.com/107720480/174903486-1a22ff17-09f7-4a65-be15-20ca6d8c7ecb.png)

### Understanding the Information Flow

If everything works properly, we should now see Kismet populate with nearby SSIDs and their associated information:

![kismetMainobf1](https://user-images.githubusercontent.com/107720480/174914354-c231e4fa-69fb-44ee-b2c5-702f5bfda8f4.png)
*Note: The SSID and BSSID columns have been censored to protect the privacy of these network devices.*

Take a moment to look over the main dashboard. There is information here that would be extremely useful in a security audit, a sanctioned penetration test, an incident investigation, and more.

### Relevant Datapoints

#### *SSID (Name)* and *BSSID (MAC Address)*

- The SSID, or **S**ervice **S**et **ID**entifier, is the name given to a WiFi access point. It could be a seemingly random set of characters, which is often the case from the factory, or it can be customized by the network administrator or owner.
- The BSSID, or **B**asic **S**ervice **S**et **ID**entifier is the MAC address of the wireless access point. This denotes the manufacturer and model number of the product in question.

#### *Crypto*

Perhaps the most important piece of information, especially when it comes to security auditing, is the *Crypto* column, or the encryption type of the WiFi network. In networking, there are a handful of encryption protocols used these days to protect from unwanted access. The strongest of which, WPA3 and WPA2, are the current standards. However, in the above example, we also see WEP and *Open*. WEP is a deprecated security standard that can be easily cracked, and therefore represents a security concern for whoever is running that network. *Open*, which lacks any encryption or password at all, is the least secure.

#### *Sgn*

Another important column is *Sgn*, or signal strength. This is measured in dBm (decibel-milliwatts). The closer this value is to zero, the stronger the signal strength of the device. This is useful for physically locating rogue APs or other unwanted devices if you're willing to play *hot and cold* with the dBm value of the device. As the source of the Kismet scan moves closer to the intended device, the dBm or *Sgn* should increase towards zero. While it's not precise enough to measure exact meter or foot distance, it's a good metric with instant feedback that can help in the aforementioned scenario.

### Viewing Associated Devices on Nearby Networks

This part's really cool. If you left-click any access point in the main dashboard, navigate to the *WiFi (802.11)* menu, and select *Associated Clients*, Kismet will display some information about associated devices such as their MAC address, manufacturer, and connection type, as seen here:

![deviceInfo](https://user-images.githubusercontent.com/107720480/174921103-ce1e1dde-330f-421c-90fc-915b12cad15a.png)   
*Note: This is an association scan of my own personal home network.*

While the associated devices option won't share the exact make and model, you can still find that out by looking up the MAC address (censored above). For this example, the associated device is my laptop. If I had an iPhone or MacBook connected to my network, it would display another entry where the manufacturer is Apple. One very useful function of this tool is to check if there are any potentially unwanted devices on a network, like a Raspberry Pi, for instance, which could be a potential indicator of compromise.


## Part 3: Conclusion and Final Thoughts

Kismet's utility should not be understated. Along with the use cases mentioned above, there are countless more scenarios where a tool like this would come in handy. I'm currently working on a project that uses data gathered from Kismet to create a *known wireless environment* database for any assigned local area (depending on signal strength of wireless adapter), which conducts scans on a daily basis and assigns a device familiarity rating based on frequency of scan detections. This could be used as a reference and monitoring tool for secured sites where unknown devices could potentially arouse concern.

Finally, I want to reiterate that ethics should always be your foremost priority when using network tools like Kismet. Do not seek unauthorized network or device access, and limit the scope of your practice to your own network and devices whenever possible.
