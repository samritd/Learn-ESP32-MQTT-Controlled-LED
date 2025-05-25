# ESP32 MQTT Servo Control System

## ภาพรวมของระบบ

โปรเจกต์นี้เป็นระบบควบคุม Servo Motor ผ่าน MQTT Protocol โดยใช้ ESP32 เป็น Microcontroller หลัก ระบบสามารถรับคำสั่งจากระยะไกลผ่านอินเทอร์เน็ตเพื่อควบคุมการหมุนของ Servo Motor

## การทำงานของระบบ

1. ESP32 เชื่อมต่อกับ WiFi Network
2. เชื่อมต่อกับ MQTT Broker ผ่าน ngrok tunnel
3. Subscribe รับข้อมูลจาก MQTT Topic `/Samritd/servo`
4. แปลงข้อมูลที่รับมาเป็นมุม (0-180 องศา)
5. สั่งให้ Servo Motor หมุนไปยังมุมที่ต้องการ

## Library ที่ใช้

```cpp
#include <WiFi.h>        // สำหรับการเชื่อมต่อ WiFi
#include <PubSubClient.h> // สำหรับการสื่อสาร MQTT
#include <ESP32Servo.h>   // สำหรับควบคุม Servo Motor
```

## การกำหนดค่าเริ่มต้น

### WiFi Configuration
```cpp
const char* ssid = "MARA1";           // ชื่อ WiFi Network
const char* password = "MARAMARA1";    // รหัสผ่าน WiFi
```

### MQTT Configuration
```cpp
const char* mqtt_server = "0.tcp.ap.ngrok.io"; // MQTT Broker Address
const int mqtt_port = 10016;                   // MQTT Port
const char* mqtt_topic = "/Samritd/servo";   // MQTT Topic
```

### Hardware Configuration
```cpp
const int SERVO_PIN = 14;  // GPIO Pin ที่เชื่อมต่อกับ Servo
```

## ฟังก์ชันหลัก

### 1. setup_wifi()
**วัตถุประสงค์:** เชื่อมต่อ ESP32 กับ WiFi Network

**การทำงาน:**
- เริ่มต้นการเชื่อมต่อ WiFi
- รอจนกว่าจะเชื่อมต่อสำเร็จ
- แสดง IP Address ที่ได้รับ

```cpp
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
  
  Serial.println("\nWiFi connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}
```

### 2. callback()
**วัตถุประสงค์:** จัดการข้อความที่รับมาจาก MQTT

**การทำงาน:**
- รับข้อความจาก MQTT Topic
- แปลงข้อความเป็นตัวเลข (มุม)
- ตรวจสอบความถูกต้องของมุม (0-180 องศา)
- สั่งให้ Servo หมุนไปยังมุมที่กำหนด

```cpp
void callback(char* topic, byte* payload, unsigned int length) {
  // แสดงข้อความที่รับมา
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");

  // แปลง payload เป็น String
  String msg;
  for (unsigned int i = 0; i < length; i++) {
    msg += (char)payload[i];
  }
  Serial.println(msg);

  // แปลงเป็นมุมและควบคุม Servo
  int angle = msg.toInt();
  if (angle >= 0 && angle <= 180) {
    myservo.write(angle);
    Serial.print("Servo moved to: ");
    Serial.println(angle);
  } else {
    Serial.println("Invalid angle received.");
  }
}
```

### 3. reconnect()
**วัตถุประสงค์:** จัดการการเชื่อมต่อ MQTT ใหม่เมื่อการเชื่อมต่อขาดหาย

**การทำงาน:**
- ตรวจสอบสถานะการเชื่อมต่อ MQTT
- พยายามเชื่อมต่อใหม่หากขาดการเชื่อมต่อ
- Subscribe Topic เมื่อเชื่อมต่อสำเร็จ

```cpp
void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    String clientId = "ESP32Client-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      client.subscribe(mqtt_topic);
      Serial.print("Subscribed to topic: ");
      Serial.println(mqtt_topic);
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" retrying in 5 seconds");
      delay(5000);
    }
  }
}
```

## ฟังก์ชัน setup() และ loop()

### setup()
```cpp
void setup() {
  Serial.begin(115200);           // เริ่มต้น Serial Communication
  myservo.attach(SERVO_PIN);      // กำหนด Pin สำหรับ Servo
  setup_wifi();                   // เชื่อมต่อ WiFi
  client.setServer(mqtt_server, mqtt_port);  // กำหนด MQTT Server
  client.setCallback(callback);   // กำหนด Callback Function
}
```

### loop()
```cpp
void loop() {
  if (!client.connected()) {
    reconnect();                  // เชื่อมต่อใหม่หากขาดการเชื่อมต่อ
  }
  client.loop();                  // จัดการ MQTT Messages
}
```

## การเชื่อมต่อฮาร์ดแวร์

### ESP32 Pin Configuration
- **Servo Signal Pin:** GPIO 14
- **Servo VCC:** 5V หรือ 3.3V (ขึ้นอยู่กับ Servo)
- **Servo GND:** GND

### Servo Motor Wiring
```
Servo Motor    |    ESP32
---------------|----------
Red (VCC)      |    5V/3.3V
Brown (GND)    |    GND
Orange (Signal)|    GPIO 14
```

## การทดสอบระบบ

### 1. การตรวจสอบการเชื่อมต่อ
- เปิด Serial Monitor ที่ Baud Rate 115200
- ตรวจสอบการเชื่อมต่อ WiFi และ IP Address
- ตรวจสอบการเชื่อมต่อ MQTT Broker

### 2. การทดสอบควบคุม Servo
ส่งข้อความไปยัง MQTT Topic `/Samritd/servo` ด้วยค่าต่างๆ:
- `0` - Servo หมุนไป 0 องศา
- `90` - Servo หมุนไป 90 องศา
- `180` - Servo หมุนไป 180 องศา

## ข้อควรระวัง

### 1. Power Supply
- Servo Motor อาจต้องการกระแสไฟสูง โดยเฉพาะ High-torque Servo
- ควรใช้แหล่งจ่ายไฟแยกต่างหากสำหรับ Servo หากจำเป็น

### 2. Angle Validation
- ระบบมีการตรวจสอบมุมที่ 0-180 องศาเท่านั้น
- การส่งค่านอกช่วงนี้จะไม่ส่งผลต่อ Servo

### 3. Network Connection
- ตรวจสอบให้แน่ใจว่า ngrok tunnel ยังทำงานอยู่
- ตรวจสอบการเชื่อมต่อ WiFi อย่างสม่ำเสมอ

## การพัฒนาต่อยอด

### 1. เพิ่มฟีเจอร์ Feedback
```cpp
// ส่งตำแหน่งปัจจุบันกลับไปยัง MQTT
client.publish("/Samritd/servo/position", String(angle).c_str());
```

### 2. เพิ่มการเคลื่อนไหวแบบ Smooth
```cpp
// เคลื่อนไหวแบบค่อยเป็นค่อยไป
void smoothMove(int targetAngle) {
  int currentAngle = myservo.read();
  int step = (targetAngle > currentAngle) ? 1 : -1;
  
  for (int pos = currentAngle; pos != targetAngle; pos += step) {
    myservo.write(pos);
    delay(15);
  }
}
```

### 3. เพิ่ม Watchdog Timer
```cpp
#include "esp_system.h"
// เพิ่ม Watchdog เพื่อความเสถียรของระบบ
```

## สรุป

ระบบนี้เป็นตัวอย่างที่ดีของการใช้ ESP32 ควบคุมอุปกรณ์ฮาร์ดแวร์ผ่าน MQTT Protocol สามารถนำไปประยุกต์ใช้ในงาน IoT หรือ Home Automation ได้หลากหลาย โดยมีความเสถียรและความปลอดภัยในการทำงาน

---

## Complete Source Code

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <ESP32Servo.h>

// WiFi credentials
const char* ssid = "MARA1";
const char* password = "MARAMARA1";

// MQTT broker details
const char* mqtt_server = "0.tcp.ap.ngrok.io";
const int mqtt_port = 10016;
const char* mqtt_topic = "/Samritd/servo";

// Servo configuration
Servo myservo;
const int SERVO_PIN = 14;

// WiFi and MQTT clients
WiFiClient espClient;
PubSubClient client(espClient);

// Connect to WiFi
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
  
  Serial.println("\nWiFi connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}

// Handle incoming MQTT messages
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");

  String msg;
  for (unsigned int i = 0; i < length; i++) {
    msg += (char)payload[i];
  }
  Serial.println(msg);

  // Convert message to int and move servo
  int angle = msg.toInt();
  if (angle >= 0 && angle <= 180) {
    myservo.write(angle);
    Serial.print("Servo moved to: ");
    Serial.println(angle);
  } else {
    Serial.println("Invalid angle received.");
  }
}

// Ensure MQTT connection stays alive
void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    String clientId = "ESP32Client-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      client.subscribe(mqtt_topic);
      Serial.print("Subscribed to topic: ");
      Serial.println(mqtt_topic);
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" retrying in 5 seconds");
      delay(5000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  myservo.attach(SERVO_PIN);
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
