# เอกสารโปรแกรม MQTT LED Controller

## คำอธิบายโปรแกรม

โปรแกรมนี้เป็น GUI Application ที่ใช้ Python Tkinter สำหรับควบคุม LED ผ่าน MQTT Protocol โดยสามารถส่งคำสั่ง ON/OFF ไปยัง MQTT Broker เพื่อควบคุมสถานะของ LED

## ไลบรารีที่ใช้

```python
import tkinter as tk           # สำหรับสร้าง GUI
import paho.mqtt.client as mqtt # สำหรับการสื่อสาร MQTT
```

## การกำหนดค่าเริ่มต้น

```python
MQTT_BROKER = '0.tcp.ap.ngrok.io'  # ที่อยู่ MQTT Broker
MQTT_PORT = 10016                  # Port สำหรับเชื่อมต่อ
MQTT_TOPIC = '/chacharin/led'      # Topic สำหรับส่งข้อมูล
```

## คลาส MqttGui

### Constructor (__init__)
```python
def __init__(self):
    self.client = mqtt.Client()                    # สร้าง MQTT Client
    self.client.connect(MQTT_BROKER, MQTT_PORT, 60) # เชื่อมต่อกับ Broker
    
    self.root = tk.Tk()                           # สร้างหน้าต่างหลัก
    self.root.title("MQTT LED Controller")        # ตั้งชื่อหน้าต่าง
    
    # สร้างปุ่ม ON
    self.button_on = tk.Button(self.root, text="ON", command=self.publish_on)
    self.button_on.pack(pady=10)
    
    # สร้างปุ่ม OFF
    self.button_off = tk.Button(self.root, text="OFF", command=self.publish_off)
    self.button_off.pack(pady=10)
```

### ฟังก์ชัน publish_on()
```python
def publish_on(self):
    self.client.publish(MQTT_TOPIC, "on")    # ส่งข้อความ "on"
    print("Sent MQTT message: on")          # แสดงผลใน console
```

### ฟังก์ชัน publish_off()
```python
def publish_off(self):
    self.client.publish(MQTT_TOPIC, "off")   # ส่งข้อความ "off"
    print("Sent MQTT message: off")         # แสดงผลใน console
```

### ฟังก์ชัน run()
```python
def run(self):
    self.root.mainloop()    # เริ่มต้น GUI event loop
```

## ฟังก์ชันหลัก

```python
def main():
    gui = MqttGui()    # สร้าง instance ของ MqttGui
    gui.run()          # เรียกใช้งาน GUI

if __name__ == '__main__':
    main()             # เรียกใช้ฟังก์ชันหลัก
```

## การทำงานของโปรแกรม

1. **การเชื่อมต่อ MQTT**: โปรแกรมจะเชื่อมต่อไปยัง MQTT Broker ที่กำหนดไว้
2. **สร้าง GUI**: สร้างหน้าต่างที่มีปุ่ม ON และ OFF
3. **การส่งข้อมูล**: เมื่อกดปุ่มจะส่งข้อความไปยัง MQTT Topic ที่กำหนด
4. **การแสดงผล**: แสดงข้อความยืนยันใน console

## การใช้งาน

1. ติดตั้ง library ที่จำเป็น:
   ```bash
   pip install paho-mqtt
   ```

2. รันโปรแกรม:
   ```bash
   python mqtt_led_controller.py
   ```

3. กดปุ่ม ON เพื่อเปิด LED หรือ OFF เพื่อปิด LED

## ข้อควรระวัง

- ตรวจสอบการเชื่อมต่อเครือข่าย
- ตรวจสอบว่า MQTT Broker พร้อมใช้งาน
- ตรวจสอบ Topic และ Port ให้ถูกต้อง

## การปรับปรุงที่แนะนำ

1. เพิ่มการจัดการ error handling
2. เพิ่มการแสดงสถานะการเชื่อมต่อ
3. เพิ่มการยืนยันการส่งข้อมูลสำเร็จ
4. เพิ่ม GUI elements อื่นๆ เช่น status bar

## โค้ดโปรแกรมทั้งหมด

```python
#!/usr/bin/env python3
import tkinter as tk
import paho.mqtt.client as mqtt

MQTT_BROKER = '0.tcp.ap.ngrok.io'
MQTT_PORT = 10016
MQTT_TOPIC = '/chacharin/led'

class MqttGui:
    def __init__(self):
        self.client = mqtt.Client()
        self.client.connect(MQTT_BROKER, MQTT_PORT, 60)
        
        self.root = tk.Tk()
        self.root.title("MQTT LED Controller")
        
        self.button_on = tk.Button(self.root, text="ON", command=self.publish_on)
        self.button_on.pack(pady=10)
        
        self.button_off = tk.Button(self.root, text="OFF", command=self.publish_off)
        self.button_off.pack(pady=10)
    
    def publish_on(self):
        self.client.publish(MQTT_TOPIC, "on")
        print("Sent MQTT message: on")
    
    def publish_off(self):
        self.client.publish(MQTT_TOPIC, "off")
        print("Sent MQTT message: off")
    
    def run(self):
        self.root.mainloop()

def main():
    gui = MqttGui()
    gui.run()

if __name__ == '__main__':
    main()
```
