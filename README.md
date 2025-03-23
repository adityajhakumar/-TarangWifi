
# 📡 TarangWifi

**TarangWifi** is a real-time Wi-Fi signal tracker and optimizer built with **Streamlit**. Move around your environment and find the best Wi-Fi spots by tracking **signal strength**, **latency**, and **internet speed** metrics—all displayed with intuitive charts and reports.

---

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Streamlit App](https://img.shields.io/badge/Streamlit-Ready-orange)](https://github.com/adityajhakumar/-TarangWifi)

---

## 🌐 Live Demo  
🚀 [GitHub Repository](https://github.com/adityajhakumar/-TarangWifi)

---

## 🚀 Features

✅ **Real-Time Wi-Fi Signal Tracking**  
✅ **Latency & Speed Tests (Optional)**  
✅ **Location Tagging for Spot Analysis**  
✅ **Interactive Graphs (Signal, Latency, Speed)**  
✅ **Summary Report of Best/Worst Spots**  
✅ **Export Report as CSV**  
✅ **Windows and Linux Compatible**  

---

## 🎥 Demo  
![image](https://github.com/user-attachments/assets/fa4c90c5-ad35-4644-b183-092b0846f335)
![image](https://github.com/user-attachments/assets/7e98977a-632b-4616-82d7-86b1c22200ef)

![image](https://github.com/user-attachments/assets/7c2a444f-45ab-437a-98b7-66a4fae36b11)
![image](https://github.com/user-attachments/assets/fdaefa02-5c81-47d9-802c-2b03690e2363)
![image](https://github.com/user-attachments/assets/e0ef4c10-f17f-445d-90e7-be0446476b72)
![image](https://github.com/user-attachments/assets/48d332ed-2311-4a5e-b835-0e1f15e35ce9)
![image](https://github.com/user-attachments/assets/f6ec1824-b35f-49d3-ab3f-a8063b2c91bb)

---


# 📄 **TarangWifi Code Explanation**

---

## 1️⃣ **Importing Libraries**

```python
import streamlit as st
import pandas as pd
import plotly.graph_objs as go
import subprocess
import platform
import speedtest
from datetime import datetime
import time
import re
```

- **streamlit**: For building interactive web apps.
- **pandas**: Data manipulation and storage of tracking data.
- **plotly.graph_objs**: For creating interactive, real-time charts.
- **subprocess**: To run system-level commands (fetching Wi-Fi info, ping).
- **platform**: Detecting OS (Windows/Linux) to run the right commands.
- **speedtest**: To check download and upload internet speeds.
- **datetime, time**: Handling timestamps and delays.
- **re (Regular Expressions)**: Parsing command-line output for Wi-Fi details.

---

## 2️⃣ **Helper Functions**

---

### 🔹 `get_wifi_info()`
Fetches **SSID**, **BSSID**, **Signal Strength**, **Frequency**, and **Channel** info.

#### How it works:
- Detects OS (`Linux` or `Windows`).
- Runs OS-specific commands:
  - **Linux**: Uses `iwconfig`
  - **Windows**: Uses `netsh wlan show interfaces`
- Uses **regular expressions** to extract relevant details from the command output.

```python
def get_wifi_info():
    system_platform = platform.system()

    wifi_info = {
        'SSID': 'Unknown',
        'BSSID': 'Unknown',
        'Signal': None,
        'Frequency': 'Unknown',
        'Channel': 'Unknown'
    }

    try:
        if system_platform == 'Linux':
            # Run 'iwconfig' to get Wi-Fi info
            result = subprocess.check_output(["iwconfig"], universal_newlines=True)
            
            # Extract relevant info using regex
            ssid_match = re.search(r'ESSID:"(.+?)"', result)
            freq_match = re.search(r'Frequency:(\d+\.\d+)', result)
            signal_match = re.search(r'Signal level=(-?\d+) dBm', result)

            if ssid_match:
                wifi_info['SSID'] = ssid_match.group(1)
            if freq_match:
                wifi_info['Frequency'] = f"{freq_match.group(1)} GHz"
            if signal_match:
                wifi_info['Signal'] = int(signal_match.group(1))

        elif system_platform == 'Windows':
            # Run 'netsh' to get Wi-Fi info
            result = subprocess.check_output(["netsh", "wlan", "show", "interfaces"], universal_newlines=True)

            # Extract data via regex
            ssid_match = re.search(r'SSID\s+:\s(.*)', result)
            bssid_match = re.search(r'BSSID\s+:\s(.*)', result)
            signal_match = re.search(r'Signal\s+:\s(\d+)%', result)
            freq_match = re.search(r'Radio type\s+:\s(.*)', result)

            if ssid_match:
                wifi_info['SSID'] = ssid_match.group(1).strip()
            if bssid_match:
                wifi_info['BSSID'] = bssid_match.group(1).strip()
            if signal_match:
                wifi_info['Signal'] = int(signal_match.group(1).strip())
            if freq_match:
                wifi_info['Frequency'] = freq_match.group(1).strip()

    except Exception as e:
        st.error(f"Error fetching Wi-Fi info: {e}")

    return wifi_info
```

---

### 🔹 `get_latency(host="8.8.8.8")`
Checks the **ping latency** to Google's DNS (8.8.8.8).

- Windows: `ping -n 1`
- Linux: `ping -c 1`
- Extracts **latency** from ping output using string parsing.

```python
def get_latency(host="8.8.8.8"):
    try:
        param = '-n' if platform.system().lower() == 'windows' else '-c'
        command = ['ping', param, '1', host]
        
        result = subprocess.run(command, stdout=subprocess.PIPE, universal_newlines=True)
        output = result.stdout

        if platform.system().lower() == 'windows':
            latency_line = [line for line in output.splitlines() if "Average" in line]
            latency = latency_line[0].split('=')[-1].replace('ms', '').strip()
        else:
            latency_line = [line for line in output.splitlines() if "time=" in line]
            latency = latency_line[0].split('time=')[1].split(' ')[0]

        return float(latency)

    except Exception as e:
        st.warning(f"Latency check failed: {e}")
        return None
```

---

### 🔹 `run_speedtest()`
Uses `speedtest.Speedtest()` to measure **download** and **upload** speeds.

```python
def run_speedtest():
    try:
        stest = speedtest.Speedtest()
        download_speed = round(stest.download() / 1_000_000, 2)  # Convert to Mbps
        upload_speed = round(stest.upload() / 1_000_000, 2)
        return download_speed, upload_speed

    except Exception as e:
        st.warning(f"Speedtest failed: {e}")
        return None, None
```

---

## 3️⃣ **Streamlit App Layout**

### 🔹 App Setup

```python
st.set_page_config(page_title="Wi-Fi Signal Tracker & Optimizer", layout="wide")
st.title("📡 TarangWifi")
st.caption("Move around and find the best Wi-Fi spots in real time!")
```

---

## 4️⃣ **Session State Initialization**

Session state ensures data persists across Streamlit reruns.

```python
if 'tracking' not in st.session_state:
    st.session_state.tracking = False

if 'data' not in st.session_state:
    st.session_state.data = pd.DataFrame(columns=['Time', 'Location', 'SSID', 'BSSID', 'Signal', 'Latency', 'Download', 'Upload'])
```

---

## 5️⃣ **Sidebar Controls**

```python
with st.sidebar:
    st.header("Controls")
    
    location = st.text_input("🏷️ Enter Current Location", "Unknown")

    if st.button("🚀 Start Real-Time Tracking"):
        st.session_state.tracking = True
        st.session_state.last_update = 0

    if st.button("🛑 Stop Tracking"):
        st.session_state.tracking = False

    if st.button("⚡ Run Speedtest"):
        d_speed, u_speed = run_speedtest()
        if d_speed and u_speed:
            st.success(f"Download: {d_speed} Mbps | Upload: {u_speed} Mbps")

    if st.button("⏱️ Check Latency"):
        ping_latency = get_latency()
        if ping_latency:
            st.success(f"Latency: {ping_latency} ms")
```

---

## 6️⃣ **Tracking Functionality**

Runs every 3 seconds if tracking is active.

```python
if st.session_state.tracking:
    now = time.time()

    wifi_info = get_wifi_info()
    latency = get_latency()

    download, upload = None, None  # Optional speedtest during tracking

    current_time = datetime.now().strftime("%H:%M:%S")
    new_row = {
        'Time': current_time,
        'Location': location,
        'SSID': wifi_info['SSID'],
        'BSSID': wifi_info['BSSID'],
        'Signal': wifi_info['Signal'],
        'Latency': latency,
        'Download': download,
        'Upload': upload
    }

    # Append data to DataFrame
    st.session_state.data = pd.concat(
        [st.session_state.data, pd.DataFrame([new_row])],
        ignore_index=True
    )

    st.toast(f"Updated {current_time}")

    # Metrics display
    wifi_col1, wifi_col2, wifi_col3 = st.columns(3)
    wifi_col1.metric("SSID", wifi_info.get('SSID', 'Unknown'))
    wifi_col2.metric("BSSID", wifi_info.get('BSSID', 'Unknown'))
    wifi_col3.metric("Frequency", wifi_info.get('Frequency', 'Unknown'))

    col1, col2, col3 = st.columns(3)
    col1.metric("📶 Signal Strength", f"{wifi_info.get('Signal', 'N/A')} dBm")
    col2.metric("⏱️ Latency", f"{latency} ms" if latency else "N/A")
    col3.metric("⚡ Download/Upload", f"{download} / {upload} Mbps" if download else "Not Tested")

    time.sleep(3)
    st.rerun()
```

---

## 7️⃣ **Graphs & Visualization**

Plotting graphs with **Plotly**:
- Signal strength over time
- Latency over time
- Download/Upload speeds over time

```python
df = st.session_state.data

if not df.empty:
    st.subheader("📊 Real-Time Tracking Data")

    # Signal strength
    fig_signal = go.Figure()
    fig_signal.add_trace(go.Scatter(x=df['Time'], y=df['Signal'], mode='lines+markers'))
    st.plotly_chart(fig_signal, use_container_width=True)

    # Latency
    fig_latency = go.Figure()
    fig_latency.add_trace(go.Scatter(x=df['Time'], y=df['Latency'], mode='lines+markers'))
    st.plotly_chart(fig_latency, use_container_width=True)

    # Download/Upload speeds
    fig_speed = go.Figure()
    fig_speed.add_trace(go.Scatter(x=df['Time'], y=df['Download'], mode='lines+markers'))
    fig_speed.add_trace(go.Scatter(x=df['Time'], y=df['Upload'], mode='lines+markers'))
    st.plotly_chart(fig_speed, use_container_width=True)
```

---

## 8️⃣ **Summary Report**

If tracking is stopped:
- Shows **best and worst spots**
- Averages for signal, latency, speed
- Allows downloading a CSV report

```python
if not st.session_state.tracking:
    st.subheader("📋 Summary Report")

    best_row = df.loc[df['Signal'].idxmax()]
    worst_row = df.loc[df['Signal'].idxmin()]

    st.markdown(f"**Best Spot** ➡️ `{best_row['Location']}` with **Signal** `{best_row['Signal']} dBm`")
    st.markdown(f"**Worst Spot** ➡️ `{worst_row['Location']}` with **Signal** `{worst_row['Signal']} dBm`")

    st.write(f"📶 **Average Signal**: {df['Signal'].mean():.2f} dBm")
    st.write(f"⏱️ **Average Latency**: {df['Latency'].mean():.2f} ms")
    st.write(f"⚡ **Average Download**: {df['Download'].mean():.2f} Mbps")
    st.write(f"⚡ **Average Upload**: {df['Upload'].mean():.2f} Mbps")

    # Download CSV
    csv = df.to_csv(index=False).encode('utf-8')
    st.download_button("📥 Download Report as CSV", data=csv, file_name='wifi_report.csv', mime='text/csv')
```

---

## 9️⃣ **Footer**

```python
st.markdown("---")
st.markdown(
    "<div style='text-align: center; color: gray;'>"
    "Made with ❤️ by <strong>Aditya Kumar Jha</strong>"
    "</div>",
    unsafe_allow_html=True
)
```

---

# ✅ **High-Level Workflow**

1. App starts ➡️ User enters location ➡️ Starts tracking
2. App collects Wi-Fi signal, latency, (optional) speed every 3 seconds
3. Displays real-time metrics + graphs
4. User stops tracking ➡️ Generates summary + CSV download

---


## 🛠️ Installation

### 1. Clone the Repository
```bash
git clone https://github.com/adityajhakumar/-TarangWifi.git
cd -TarangWifi
```

### 2. Create a Virtual Environment (Optional but Recommended)
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

If you don't have a `requirements.txt`, you can install dependencies directly:
```bash
pip install streamlit pandas plotly speedtest-cli
```

---

## ▶️ Run the App
```bash
streamlit run tarangwifi.py
```

Then open the app in your browser:  
`http://localhost:8501`

---

## 📂 Project Structure
```
-TarangWifi/
├── tarangwifi.py        # Main Streamlit app
├── README.md            # Project documentation
├── LICENSE              # Apache 2.0 License
└── requirements.txt     # Python dependencies
```

---

## ⚙️ How It Works

1. **Start Tracking:**  
   - Enter your **current location**.  
   - Click `🚀 Start Real-Time Tracking` to begin collecting data.

2. **Move Around:**  
   - TarangWifi fetches your **Wi-Fi SSID**, **BSSID**, **Signal Strength**, and **Frequency** every 3 seconds.  
   - It also runs **ping latency checks**.

3. **Speedtest (Optional):**  
   - Run a speed test manually to measure **download** and **upload speeds**.

4. **Stop Tracking:**  
   - Click `🛑 Stop Tracking`.  
   - Review the **best** and **worst** spots based on signal strength.  
   - Analyze graphs and averages.

5. **Export:**  
   - Download a **CSV report** of your tracked data.

---

## 📊 Metrics Collected
| Metric            | Description                                 |
|-------------------|---------------------------------------------|
| SSID              | Network Name                                |
| BSSID             | Network Identifier (MAC Address)            |
| Signal Strength   | Wi-Fi signal strength in dBm                |
| Frequency         | Wi-Fi frequency (GHz)                       |
| Latency           | Ping time to a test server (ms)             |
| Download Speed    | Internet download speed (Mbps)              |
| Upload Speed      | Internet upload speed (Mbps)                |
| Location          | Custom location label entered by the user   |

---

## ✅ Supported Platforms
- **Windows** (via `netsh wlan show interfaces`)  
- **Linux** (via `iwconfig`)  

⚠️ MacOS support isn't implemented yet.

---

## 📝 License

This project is licensed under the **Apache License 2.0**.  
See the full license [here](https://github.com/adityajhakumar/-TarangWifi/blob/main/LICENSE).

```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0
```

---

## 🙌 Acknowledgements
- Inspired by the need for **better Wi-Fi optimization tools** in real-world spaces.
- Thanks to open-source libraries: **Streamlit**, **Pandas**, **Plotly**, and **Speedtest-cli**.

---

## 👨‍💻 Author
**Aditya Kumar Jha**  
[GitHub](https://github.com/adityajhakumar) 

---

## 🌟 Contributions  
Contributions, issues, and feature requests are welcome!  
Feel free to **fork** this repo and **submit a pull request**.
