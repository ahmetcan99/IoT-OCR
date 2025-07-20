# IoT-OCR Hybrid System
**ESP32-based image capture → Raspberry Pi OCR processing → FastAPI/SQLite storage**  

## Key Features
- **ESP32-CAM** image capture with **SPI flash config** (WiFi/UUID management via `config.json`)
- **OpenCV** preprocessing (perspective correction + CLAHE contrast enhancement)
- **Tesseract-OCR**
- **MQTT** for remote configuration updates (no firmware reflash needed)
- **FastAPI** backend with SQLite logging

## Future Improvements
- **Planned UART/I2C Integration**: Designed for high-speed ESP32↔RPi communication, but currently uses WiFi due to time constraints.  
  - *Protocol specs ready*: 8Mbps UART (TX/RX pins) + I2C (for control signals)  
  - *Blockers*: Hardware synchronization challenges (will implement in v2.0)  

##  Tech Stack
| Component       | Technologies Used                      |
|-----------------|----------------------------------------|
| **Edge Device** | ESP32 (C++), SPI Flash, FreeRTOS       |
| **Backend**     | Raspberry Pi, FastAPI (Python 3.9)     |
| **OCR**         | Tesseract-OCR 5.0, OpenCV 4.5          |
| **Comms**       | MQTT (config), RESTful API (image data)|
| **Database**    | SQLite with UUID-based record tagging  |

##  Quick Start
### Prerequisites
- ESP32-CAM module
- Raspberry Pi 4 (2GB+ RAM recommended)
- Python 3.9 on Pi

##  Project Structure
```bash
IoT-OCR/
├── esp32-code-main/          # ESP32 PlatformIO Project
│   ├── .vscode/             # VSCode configuration
│   ├── include/             # Header files
│   ├── lib/                 # Custom libraries
│   ├── src/                 # Core C++ source files
│   │   └── main.cpp         # Primary firmware logic
│   ├── test/                # Unit tests
│   ├── .gitignore           # Git exclusion rules
│   └── platformio.ini       # PlatformIO configuration
│
├── raspberrypi-gateway-main/ # Python Backend
│   ├── db/                  # Database operations
│   ├── mqtt_controller/     # MQTT config management
│   ├── ocr/                 # OpenCV+Tesseract processing
│   ├── restapi/             # FastAPI endpoints
│   ├── .gitignore           # Git exclusion rules
│   ├── main.py             # Entry point
│   └── requirements.txt    # Python dependencies
```
##  Installation

### **ESP32 Setup**
1. **Install Required Tools**:
   ```bash
   # Install PlatformIO Core
   pip install platformio

   # Install ESP32 LittleFS tools
   pio pkg install --tool "platformio/tool-esptoolpy @ ~1.40200.0"
   pio pkg install --tool "platformio/tool-mklittlefs @ ~1.203.0"
   ```
2. **Prepare Configuration File**
   ```bash
   cd esp32-code-main
    mkdir -p data  # Create LittleFS directory
    cat > data/config.json <<EOF
    {
      "uuid": "",  # Device UUID (auto-generated on first boot)
      "description": "Device Description",
      "wifi_ssid": "YOUR_WIFI_SSID",  # Replace with your network
      "wifi_password": "YOUR_WIFI_PASSWORD",
      "server_name": "Raspberry Pi IP",
      "server_path": "/upload_photo/",
      "server_port": #Raspberry PI Port Number
      "mqtt_server": "MQTT broker IP", 
      "mqtt_username": "MQTT Connection Username",
      "mqtt_password": "MQTT Connection Password",
      "mqtt_port": #MQTT Broker Port number,
      "interval": 10000  # Photo capture interval (ms)
    }
   ```
3. **Flash Filesystem & Firmware**
   ```bash
   #First: Upload LittleFS (includes config.json)
    pio run --target uploadfs
   # Then: Upload main firmware
    pio run --target upload
   ```
### **Raspberry Pi Setup**
```bash
cd raspberrypi-gateway-main
pip install -r requirements.txt  # Installs dependencies
python main.py  # Starts server at https://localhost:8000
```
##  Critical Configuration Notes
Mandatory Changes: 
  Replace YOUR_WIFI_SSID and YOUR_WIFI_PASSWORD with your network credentials
  Update server_name and mqtt_server IPs if Raspberry Pi uses a different address
UUID Generation:
    If left empty (""), ESP32 will connect to Raspberry PI via MQTT Connection, Raspberry PI will auto-generate a UUID for ESP32 and save it in the database.
