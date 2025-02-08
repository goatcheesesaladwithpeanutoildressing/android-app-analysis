# android-app-analysis


### Root a phone (Android)
- Recovery
- Flash Lineage OS
- Flash Magisk
- Enable Developer Options
- Enable USB Debugging
- Force USB File Transfer Mode

### Setup the MITM server
- Ideally a Raspberry PI w/ > 4Go RAM / > 64Go SD CARD
- Debian 12
- Install Python
- Install ADB
- Install Frida
- Install Fritap
- Setup the WAP (Wireless Accesss Point)

### Connect phone to MITM
- Connect to the WAP
- Plug it to MITM USB
- Allow any access permissions
- `adb devices` should print the attached device id
- `adb shell` should ssh to the phone
- `su` inside the shell gives you root priviledges

### Install an APK
- Install apkeep
- Unzip the APK if necessary
- `adb install com.example.apk` / `adb install-multiple split-apk1 split-apk2 split-apk3`

### Capture network and log the ssl keys
- Launch the app
- Listen to SSL keys R/W w/ `fritap -m -k keys.log --enable_spawn_gating [APP_PID] &`
- Listen to the traffic on the WiFi interface `tcpdump -i wlan0 -w traffic.pcap`
- `fg` and cancel ssl keys listener
- Decrypt traffic `editcap --inject-secrets tls,keys.log traffic.pcap decrypted_traffic.pcapng`
- Generate a human readable JSON file `tshark -2 -T ek --enable-protocol communityid -Ndmn -r decrypted_traffic.pcapng > traffic.json`
- fx traffic.json (or any viz tool)
