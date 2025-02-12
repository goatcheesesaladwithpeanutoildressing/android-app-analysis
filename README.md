# android-app-analysis

## Setup your first HTTPS traffic analysis

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
- Download frida-server and push it to the device
- `adb shell` then exec it
- Launch the app
- Listen to SSL keys R/W w/ `fritap -m -k keys.log --enable_spawn_gating [APP_PID] &`
- Listen to the traffic on the WiFi interface `tcpdump -i wlan0 -w traffic.pcap`
- `fg` and cancel ssl keys listener
- Decrypt traffic `editcap --inject-secrets tls,keys.log traffic.pcap decrypted_traffic.pcapng`
- Generate a human readable JSON file `tshark -2 -T ek --enable-protocol communityid -Ndmn -r decrypted_traffic.pcapng > traffic.json`
- `fx traffic.json` (or any viz tool)
- Server hosts should be clear, payloads too!

## Sample decrypted HTTPS request
- POST https://aax.amazon-adsystem.com/e/msdk/ads
- Vendor: Amazon
- Payload contains device info
- Payload contains consent string under `gdpr c` key

```text
{
  "text": [
    "POST /e/msdk/ads HTTP/1.1\\r\\n",
    "\\r\\n"
  ],
  "_ws_expert": {
    "http_http_chat": null,
    "_ws_expert__ws_expert_message": "POST /e/msdk/ads HTTP/1.1\\r\\n",
    "_ws_expert__ws_expert_severity": "2097152",
    "_ws_expert__ws_expert_group": "33554432"
  },
  "http_http_request_method": "POST",
  "http_http_request_uri": "/e/msdk/ads",
  "http_http_request_version": "HTTP/1.1",
  "http_http_accept": "application/json",
  "http_http_request_line": [
    "Accept: application/json\r\n",
    "content-type: application/json; charset=utf-8\r\n",
    "User-Agent: [redacted]\r\n",
    "Host: aax.amazon-adsystem.com\r\n",
    "Connection: Keep-Alive\r\n",
    "Accept-Encoding: gzip\r\n",
    "Content-Length: 1274\r\n"
  ],
  "http_http_content_type": "application/json; charset=utf-8",
  "http_http_user_agent": "[redacted]",
  "http_http_host": "aax.amazon-adsystem.com",
  "http_http_connection": "Keep-Alive",
  "http_http_accept_encoding": "gzip",
  "http_http_content_length_header": "1274",
  "http_http_content_length": "1274",
  "http_http_request_full_uri": "https://aax.amazon-adsystem.com/e/msdk/ads",
  "http_http_request": true,
  "http_http_request_number": "1",
  "http_http_response_in": "986",
  "http_http_file_data": "{\"dinfo\":{\"os\":\"Android\",\"model\":\"[redacted]":\"aps-
android-9.10.3-GOOGLE_AD_MANAGER\",\"slots\":[{\"sz\":\"320x50\",\"slot\":\"3326843f-fee9-43db-987c-1c3976545d58\",\"slotId\":1,\"supportedMediaTypes\":[\"DISPLAY\"]}],\"appId\":\"5b1d115e-3439-44af-9b0d-
25a8e571b21a\",\"pj\":{\"autoRefresh\":\"false\",\"mediationName\":\"GOOGLE_AD_MANAGER\",\"fwk\":\"native\"},\"isDTBMobile\":\"true\",\"ua\":\"Mozilla\\/5.0 ([redacted]; wv) AppleWebKit\\/537.36
(KHTML, like Gecko) Version\\/4.0 Chrome\\/132.0.6834.122 Mobile
Safari\\/537.36\",\"pkg\":{\"lbl\":\"leboncoin\",\"pn\":\"fr.leboncoin\",\"v\":\"100039100\",\"vn\":\"100.39.1\"},\"gdpr\":{\"c\":\"CQMZ9MAQMZ9MAAHABAFRBbFoAPLgAELgAAAAJoNB_G_dTSFi8X51YPtgcQ1P4VAjogAABgaJAwwBiBLAMIwEhmAIIADqACACABAAICRAAQ
BlCADAAAAAYIAAASAMAAAAIRAIIiAAAEAAAmJICABJC4AAAQAQgkgAABUAgAIAABogSFAAAAAAFAAAAAAAAAAAAAAAAAAAQAAAAAAAAgAAAAAACAAAEAAEAFAAAAAAAAAAAAAAAAAMELwATDQqIACwJCQg0DCAAACoIAgAgAAAAAJAwQAABAgAEAYACjAAAAAFAAAAAAAAABAAAAAAgAQgAAAAIEAAAAAEAAAAEAgEABAA
AAAAABAAAAAEAMAAAIAAgAAAAAoAQAAAAAgAJCgAAAAAAgAAAAAAAAAAEAAAAAAAAAAAAAAAAQAAAAAABADFAAYAAgrKMAAwABBWUgABgACCsoA\",\"e\":1}}"
}
```

- Can't build a requests graph since we don't have the initiator context, we can only observe
requests from device to servers, no piggybacking there
- With some LLMs engineering, we could easily classify hosts and detect possible trackers (fingerprinting, geo data, device info...)
- We can also create custom frida hooks to hook Java methods such as permission requests

## Static analysis
- Besides dynamic analysis that shows active threats, we can do some static analysis to identify the landscape of possible threats
- One common practice is to analyse .dex files from an unzipped APK
- `dexdump com.example.apk | grep "Class"`
- Leverage LLMs to classify hosts from installed packages
- Also, is great to check for "requestable" permissions by parsing the AndroidApplication.xml file

## Challenges
- Root detection by apps (banking apps...)
- SSL pinning
- Not all TLS libs supported
