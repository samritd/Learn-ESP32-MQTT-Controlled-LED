# ESP32 MQTT Button Controller

## ภาพรวมของโปรแกรม
โปรแกรมนี้เป็นการควบคุมปุ่มกดผ่าน ESP32 ที่เชื่อมต่อกับ WiFi และส่งข้อมูลผ่าน MQTT Protocol ไปยัง Topic `/samrird/button` เมื่อมีการกดปุ่ม

## คุณสมบัติหลัก
- เชื่อมต่อ WiFi อัตโนมัติ
- ส่งข้อมูลผ่าน MQTT เมื่อกดปุ่ม
- ระบบ Debounce เพื่อป้องกันการส่งซ้ำ
- Auto-reconnect เมื่อสูญเสียการเชื่อมต่อ

## การตั้งค่าระบบ

### การเชื่อมต่อ WiFi
- **SSID**: MARA1
- **Password**: MARAMARA1

### การตั้งค่า MQTT
- **Server**: 0.tcp.ap.ngrok.io
- **Port**: 11882
- **Topic**: /samrird/button
- **Message**: "1" (เมื่อกดปุ่ม)

### การตั้งค่าฮาร์ดแวร์
- **Button Pin**: GPIO 27
- **Button Type**: INPUT_PULLUP (กด = LOW, ไม่กด = HIGH)

## การต่อวงจร
```
ESP32 GPIO 27 ←→ ปุ่มกด ←→ GND
```
(ไม่ต้องใช้ Resistor เพิ่มเติม เนื่องจากใช้ Internal Pull-up)

## การทำงานของระบบ

### 1. การเริ่มต้นระบบ (Setup)
- เริ่ม Serial Communication ที่ 115200 baud
- ตั้งค่า GPIO 27 เป็น INPUT_PULLUP
- เชื่อมต่อ WiFi
- เชื่อมต่อ MQTT Broker

### 2. การทำงานหลัก (Loop)
- ตรวจสอบการเชื่อมต่อ MQTT
- อ่านสถานะปุ่มจาก GPIO 27
- เมื่อตรวจพบการกดปุ่ม (HIGH→LOW) จะส่งข้อความ "1" ไปยัง Topic
- หน่วงเวลา 50ms เพื่อ Debounce

### 3. การจัดการการเชื่อมต่อ
- หากสูญเสียการเชื่อมต่อ MQTT จะพยายามเชื่อมต่อใหม่อัตโนมัติ
- สร้าง Client ID แบบสุ่มทุกครั้งเพื่อหลีกเลี่ยงความขัดแย้ง

## Source Code

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

// WiFi credentials
const char* ssid = "MARA1";
const char* password = "MARAMARA1";

// MQTT broker details
const char* mqtt_server = "0.tcp.ap.ngrok.io";
const int mqtt_port = 11882;
const char* mqtt_topic = "/samrird/button";

// Button setup
const int BUTTON_PIN = 27;
int lastButtonState = HIGH;  // default not pressed (INPUT_PULLUP)

WiFiClient espClient;
PubSubClient client(espClient);

void setup_wifi() {
  Serial.print("Connecting to WiFi");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected. IP: " + WiFi.localIP().toString());
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Connecting to MQTT...");
    String clientId = "ESP32Client-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
    } else {
      Serial.print("failed (rc=");
      Serial.print(client.state());
      Serial.println("), retrying in 5 seconds...");
      delay(5000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  int currentState = digitalRead(BUTTON_PIN);

  // Detect bump: HIGH -> LOW transition (button press)
  if (lastButtonState == HIGH && currentState == LOW) {
    Serial.println("Button bumped! Sending message...");
    client.publish(mqtt_topic, "1");
    delay(50);  // Short debounce
  }

  lastButtonState = currentState;
}
```

## การติดตั้งและใช้งาน

### 1. เตรียมอุปกรณ์
- ESP32 Development Board
- ปุ่มกด (Push Button)
- สายไฟสำหรับต่อวงจร

### 2. การต่อวงจร
- ต่อขาหนึ่งของปุ่มกับ GPIO 27 ของ ESP32
- ต่อขาอีกข้างของปุ่มกับ GND ของ ESP32

### 3. การอัพโหลด Code
- เปิด Arduino IDE
- เลือก Board เป็น ESP32
- Copy Code ข้างต้นและอัพโหลดลงบอร์ด

### 4. การทดสอบ
- เปิด Serial Monitor ที่ 115200 baud
- รอให้ระบบเชื่อมต่อ WiFi และ MQTT
- กดปุ่มเพื่อทดสอบการส่งข้อมูล

## ข้อมูลที่ส่งไปยัง MQTT Topic

**Topic**: `/samrird/button`
**Message**: `"1"`
**Trigger**: เมื่อกดปุ่มที่ GPIO 27

## หมายเหตุ
- ใช้ ngrok tunnel สำหรับการเชื่อมต่อ MQTT ในการพัฒนา
- สำหรับการใช้งานจริง ควรใช้ MQTT Broker ที่มี Static IP
- ระบบใช้ Internal Pull-up Resistor ดังนั้นไม่ต้องต่อ Resistor เพิ่มเติม
