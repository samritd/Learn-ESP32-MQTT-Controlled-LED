# ESP32 MQTT LED Control Documentation

## รายละเอียดโครงการ
ESP32 ควบคุม LED ผ่าน MQTT โดยรับคำสั่ง "on"/"off" จาก MQTT broker ผ่าน ngrok tunnel

**คุณสมบัติหลัก:** เชื่อมต่อ WiFi อัตโนมัติ | เชื่อมต่อ MQTT ผ่าน ngrok | ควบคุม LED บน GPIO 2 | ระบบ Auto-reconnect

## การวิเคราะห์โค้ด

### Libraries และ Configuration
```cpp
#include <WiFi.h>        // WiFi connection
#include <PubSubClient.h> // MQTT client
const char* ssid = "MARA1";
const char* mqtt_server = "0.tcp.ap.ngrok.io";
const char* mqtt_topic = "/Samritd";
#define LED_PIN 2
```

### ฟังก์ชันหลัก
**`setup_wifi()`** - จัดการการเชื่อมต่อ WiFi และแสดง IP address
**`callback()`** - รับและประมวลผลข้อความจาก MQTT, ควบคุม LED ตามคำสั่ง
**`reconnect()`** - จัดการการเชื่อมต่อ MQTT อีกครั้งพร้อม subscribe topic
**`setup()`** - ตั้งค่าเริ่มต้น LED, Serial, WiFi และ MQTT
**`loop()`** - ตรวจสอบการเชื่อมต่อและจัดการ MQTT messages

## การติดตั้งและใช้งาน

### อุปกรณ์ที่ต้องการ
ESP32 Development Board | สาย USB | Arduino IDE พร้อม PubSubClient library

### ขั้นตอนการติดตั้ง
1. เปิด Arduino IDE → เลือก ESP32 Dev Module
2. ติดตั้ง PubSubClient library ผ่าน Library Manager
3. Copy โค้ดและแก้ไขค่า WiFi, MQTT settings ตามต้องการ
4. Upload โค้ดไปยัง ESP32

### การทดสอบ
**Serial Monitor:** `WiFi connected → IP address: 192.168.1.xxx → Attempting MQTT connection...connected`
**MQTT Test:** ส่ง "on" → topic "/Samritd" → LED เปิด | ส่ง "off" → topic "/Samritd" → LED ปิด

## การแก้ไขปัญหา

| ปัญหา | สาเหตุ | การแก้ไข |
|-------|--------|----------|
| ไม่เชื่อมต่อ WiFi | SSID/Password ผิด | ตรวจสอบข้อมูล WiFi |
| ไม่เชื่อมต่อ MQTT | Server/Port ผิด, ngrok หมดอายุ | ตรวจสอบ ngrok tunnel |
| LED ไม่ทำงาน | GPIO ผิด, Logic Level ผิด | ตรวจสอบการเชื่อมต่อ, สลับ HIGH/LOW |

## การปรับปรุงเพิ่มเติม

### เพิ่มการส่งสถานะกลับ
```cpp
if (msg == "on") {
    digitalWrite(LED_PIN, HIGH);
    client.publish("/Samritd/status", "LED_ON");
}
```

### เพิ่มการควบคุมความสว่าง
```cpp
if (msg.startsWith("brightness:")) {
    int brightness = msg.substring(11).toInt();
    analogWrite(LED_PIN, brightness);
}
```

### เพิ่ม Watchdog Timer
```cpp
#include <esp_task_wdt.h>
esp_task_wdt_init(30, true); // 30 second watchdog
```

## ข้อมูลเทคนิค
**MQTT Topics:** Command `/Samritd` | Status `/Samritd/status` (optional)
**GPIO:** GPIO 2 = Built-in LED (Active Low/High ขึ้นอยู่กับ board)
**ngrok:** Format `[subdomain].tcp.ap.ngrok.io:[port]`

## โค้ดต้นฉบับสมบูรณ์

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

// WiFi credentials
const char* ssid = "MARA1";
const char* password = "MARAMARA1";

// MQTT broker details
const char* mqtt_server = "0.tcp.ap.ngrok.io";
const int mqtt_port = 19004;
const char* mqtt_topic = "/Samritd";

#define LED_PIN 2  // GPIO 2 is often the built-in LED on ESP32

WiFiClient espClient;
PubSubClient client(espClient);

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* message, unsigned int length) {
  Serial.print("Message arrived on topic: ");
  Serial.println(topic);

  String msg;
  for (unsigned int i = 0; i < length; i++) {
    msg += (char)message[i];
  }

  Serial.print("Message: ");
  Serial.println(msg);

  if (msg == "on") {
    digitalWrite(LED_PIN, HIGH);  // LED ON
  } else if (msg == "off") {
    digitalWrite(LED_PIN, LOW); // LED OFF
  }
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    String clientId = "ESP32Client-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      client.subscribe(mqtt_topic);
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" trying again in 5 seconds");
      delay(5000);
    }
  }
}

void setup() {
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, HIGH);  // Start with LED OFF

  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
}
```

### การใช้งานโค้ด
1. Copy โค้ดไปยัง Arduino IDE
2. แก้ไข WiFi credentials และ MQTT settings ตามสภาพแวดล้อม
3. เลือก Board: ESP32 Dev Module และ Port ที่เชื่อมต่อ
4. Upload โค้ดและทดสอบด้วยการส่งคำสั่ง "on"/"off" ผ่าน MQTT client

### ตัวอย่างการปรับแต่งสำหรับการใช้งานจริง
```cpp
const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";
const char* mqtt_server = "YOUR_MQTT_BROKER_IP";
const int mqtt_port = 1883;  // Standard MQTT port
const char* mqtt_topic = "home/led/control";
```

**การบำรุงรักษา:** ตรวจสอบการเชื่อมต่อ WiFi/MQTT เป็นระยะ | อัพเดท firmware | ใช้ SSL/TLS encryption | สำรองโค้ดและการตั้งค่า

---
*เอกสารอ้างอิง ESP32 MQTT LED Control System - สำหรับการพัฒนาและบำรุงรักษา*
