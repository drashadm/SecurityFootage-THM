# SecurityFootage-THM
Blue Team CTF: Recovered deleted security footage from a .pcap file using Wireshark, Python, and FFmpeg. Demonstrates real-world forensic skills in network packet analysis, MJPEG stream extraction, and flag hunting. Challenge from TryHackMe's “Security Footage” room.

# TryHackMe: Security Footage – CTF Walkthrough

## Challenge Objective
Recover deleted security footage from a `.pcap` file and locate the hidden flag.

---

## Skills Demonstrated
- Packet analysis using Wireshark
- Stream reassembly from TCP traffic
- JPEG file carving from raw binary data
- Forensic video reconstruction with FFmpeg
- OCR flag extraction techniques

---

## Step-by-Step Solution

### 1. Analyze the PCAP File
Opened the file `security-footage-1648933966395.pcap` in **Wireshark**.

- Found that most traffic used `TCP`
- Applied the display filter:

  ```
  tcp contains "Content-Type"
  ```

- This revealed:
  ```
  Content-Type: multipart/x-mixed-replace; boundary=BoundaryString
  ```

  Discovered a **motion JPEG (MJPEG)** stream!

---

### 2. Extract the Raw MJPEG Stream

- Located the reassembled stream in packet `#11**`
- Right-clicked → Follow → TCP Stream → saved **as Raw**
- Saved file as `stream.raw`

---

### 3. Extract JPEG Frames with Python

Used a custom script to carve `.jpg` images from `stream.raw`:

```python
from pathlib import Path

def extract_jpegs(file_path, output_dir):
    with open(file_path, "rb") as f:
        data = f.read()

    Path(output_dir).mkdir(parents=True, exist_ok=True)
    start_marker = b'\xff\xd8'
    end_marker = b'\xff\xd9'
    count = 0
    pos = 0

    while True:
        start = data.find(start_marker, pos)
        if start == -1:
            break
        end = data.find(end_marker, start)
        if end == -1:
            break
        end += 2
        with open(f"{output_dir}/frame_{count:04d}.jpg", "wb") as img:
            img.write(data[start:end])
        count += 1
        pos = end
```

Extracted **541 frames**.

---

### 4. Reconstruct the Video

Using `ffmpeg`, turned image sequence into a playable `.mp4`:

```bash
ffmpeg -framerate 10 -i frame_%04d.jpg -c:v libx264 -pix_fmt yuv420p reconstructed.mp4
```

Video created: `reconstructed.mp4`

---

### 5. Locate the Flag

- Watched the video manually frame-by-frame
- (Also attempted: Used `pytesseract` for OCR scanning every 10th frame)
- Flag was visible in a key frame in format:

  ```
  ....{...}
  ```

---

## Blue Team Insights

### What Happened?
- A live MJPEG stream was intercepted and stored in a PCAP
- It was transmitted over **unencrypted HTTP**

### How to Defend:
- Enforce **HTTPS/TLS** on all video endpoints
- Monitor for long-lived HTTP streams using **IDS (Suricata/Zeek)**
- Segment IoT devices like cameras on a **separate VLAN**
- Use **firewalls** to restrict access to surveillance feeds

---

## 🧾 Flag: _(hidden to avoid spoilers)_

---

---

## ✍️ Author

**Drashadm**  
 Cybersecurity Purple Teamer | SOC & Threat Hunter  

