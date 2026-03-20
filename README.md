**WPA/WPA2 Handshake Capture and Crack with Aircrack-ng**
- **Date: March 20, 2026**

**Objective**

The goal of this lab was to capture a WPA/WPA2 handshake from a controlled Wi-Fi environment, confirm the capture in Wireshark, and perform an offline dictionary attack to recover the pre-shared key.

This exercise focuses on understanding how weak passwords can be exploited and how to reduce that risk in real environments.

**Ethical and legal scope**

All activity was carried out on a personal router using owned devices in a controlled lab setup. No external networks or third-party systems were involved.

**Tools used**

- Kali Linux (latest version)
- Aircrack-ng suite (airmon-ng, airodump-ng, aireplay-ng, aircrack-ng)
- Mdk4
- Wireshark
- Wireless adapter with monitor mode support (Realtek rtl88xxcu chipset)
- Wordlist located at /usr/share/wordlists/rockyou.txt

**Process**

**1.** I began by preparing the wireless adapter for monitor mode. Interfering processes were stopped, then monitor mode was enabled and verified.

**Bash:**
   - sudo airmon-ng check kill
   - sudo airmon-ng start wlan0
   - iwconfig
<img width="1152" height="622" alt="Turning wlan adaptor to monitor mode" src="https://github.com/user-attachments/assets/1eb61a91-2ea1-4085-8efb-188863261e15" />


#

**2.** I scanned for nearby networks and identified the target access point from my lab setup.

**Bash**
   - sudo airodump-ng wlan0

**The target network details were as follows:**
- BSSID: 60:83:E7:47:CA:4D
- ESSID: fatema
- Channel: 2
- Encryption: WPA2-PSK (CCMP)
<img width="1157" height="628" alt="Target Wifi" src="https://github.com/user-attachments/assets/b8503e3c-3601-443e-a554-803d9bf1fe65" />

#

**3.** I then focused the capture on this specific network to collect the necessary packets.

**Bash**
   - sudo iwconfig wlan0 channel 2
   - sudo airodump-ng --bssid 60:83:E7:47:CA:4D -c 2 --write 4Wayhandshake wlan0
<img width="1157" height="625" alt="Capture target wifi traffic" src="https://github.com/user-attachments/assets/2d9fc1b3-653f-4e4b-b895-7426ba58ef45" />

#

**4.** A client reconnection was triggered to generate traffic, and the handshake was successfully captured.

**Bash**
- sudo mdk4 wlan0 d -c 2
<img width="1160" height="623" alt="Capture target wifi traffic2" src="https://github.com/user-attachments/assets/f5ef8733-2246-4bdd-aa2a-dc47ce0c42a4" />

#

**5.** To confirm the capture, I opened the file in Wireshark.

**Bash**
- Wireshark 4Wayhandshake-01.cap
- Using the filter: eapol
<img width="1473" height="705" alt="Verify capture 4wayhandshake using wireshark" src="https://github.com/user-attachments/assets/a7e189d4-118f-4f13-b0ff-bea87d229b5f" />

I observed all four EAPOL key messages exchanged between the client and the access point. This confirmed that the full 4-way handshake had been captured.

#

**6.** With a valid handshake available, I moved on to the cracking phase using a dictionary attack.
   
   **Bash**
   - sudo aircrack-ng 4Wayhandshake-01.cap -w /usr/share/wordlists/rockyou.txt
<img width="1152" height="625" alt="WiFi password key found" src="https://github.com/user-attachments/assets/dad7ce1d-79f3-49d6-82f1-4388a7e73b26" />

The tool processed 3777 packets and identified one WPA2 target. It tested keys at a rate of about 1732 per second. The correct key was found after reaching 16.49 percent of the wordlist.

**Recovered key:**
- 123456@@

The output also included the derived master key, transient key, and EAPOL HMAC values.

**Defensive analysis**

- This lab demonstrates a known weakness in WPA/WPA2-PSK networks. The 4-way handshake can be captured without needing to stay connected to the network. Once obtained, it allows offline password guessing.

- If the passphrase is simple or commonly used, it can be recovered with standard wordlists.

**Mitigation**

- Use long and unpredictable passwords (Characters >10, including special characters)
- Avoid reuse of common or dictionary-based passwords
- Move to WPA3 where possible
- Enable Protected Management Frames to reduce deauthentication abuse
- Monitor wireless traffic for unusual authentication patterns
- Use 802.1X authentication in business environments instead of shared keys

**Screenshots included**

- Monitor mode setup
- Targeted packet capture with handshake confirmation
- Wireshark view showing EAPOL messages
- Network scan displaying access point and connected clients
- Aircrack-ng output showing successful key recovery
