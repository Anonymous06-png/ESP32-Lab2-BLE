📘 ESP32 – Lab2: BLE (Bluetooth Low Energy)
1. Giới thiệu

Bài thực hành này giúp sinh viên hiểu và triển khai các tính năng BLE của ESP32:

BLE Peripheral (GATT Server)

BLE Central (GATT Client)

BLE giao tiếp giữa 2 ESP32

BLE nâng cao: Notify, Pairing, truyền chuỗi dài

Toàn bộ được lập trình bằng Arduino IDE + thư viện ESP32 BLE Arduino.

2. Nội dung & mã nguồn từng phần
🔵 Part 1 – BLE Central (Client)

ESP32 làm BLE Central: quét → tìm thiết bị BLE mục tiêu → kết nối → nhận Notify hoặc Read characteristic.

🔑 Chức năng chính

Quét BLE và tìm đúng Service UUID

Kết nối đến BLE Server

Lấy Remote Service + Characteristic

Nhận Notify hoặc đọc giá trị

Tự động quét lại nếu mất kết nối

#include <BLEDevice.h>
#include <BLEScan.h>
#include <BLEClient.h>

static BLEUUID serviceUUID("4fafc201-1fb5-459e-8fcc-c5c9c331914b");
static BLEUUID charUUID("beb5483e-36e1-4688-b7f5-ea07361b26a8");

BLERemoteCharacteristic* pRemoteCharacteristic;
BLEAdvertisedDevice* targetDevice;
bool doConnect = false;

class DeviceCallbacks : public BLEAdvertisedDeviceCallbacks {
  void onResult(BLEAdvertisedDevice dev) {
    if (dev.isAdvertisingService(serviceUUID)) {
      BLEDevice::getScan()->stop();
      targetDevice = new BLEAdvertisedDevice(dev);
      doConnect = true;
      Serial.println("Found target device!");
    }
  }
};

void setup() {
  Serial.begin(115200);
  BLEDevice::init("");

  BLEScan* scan = BLEDevice::getScan();
  scan->setAdvertisedDeviceCallbacks(new DeviceCallbacks());
  scan->setActiveScan(true);
  scan->start(0);   // Quét liên tục
}

void loop() {
  if (doConnect) {
    BLEClient* client = BLEDevice::createClient();
    if (client->connect(targetDevice)) {
      auto service = client->getService(serviceUUID);
      pRemoteCharacteristic = service->getCharacteristic(charUUID);

      if (pRemoteCharacteristic->canNotify()) {
        pRemoteCharacteristic->registerForNotify(
          [](BLERemoteCharacteristic*, uint8_t* data, size_t len, bool){
            Serial.print("Notify: ");
            Serial.write(data, len);
            Serial.println();
          }
        );
      }
    }
    doConnect = false;
  }
}

🔹 Mô tả nhanh

BLEDevice::init() → khởi tạo BLE & đặt tên

createService() → tạo service BLE

createCharacteristic() → tạo characteristic có READ + WRITE

startAdvertising() → phát BLE để thiết bị khác tìm thấy

🔶 PART 2 — BLE CENTRAL (GATT CLIENT)
🟢 Part 2 – BLE Peripheral (Server)

ESP32 làm BLE Server: tạo service → tạo characteristic → Notify dữ liệu cảm biến → nhận lệnh bật/tắt LED.

🔑 Chức năng chính

Tạo BLE Server + Service

Characteristic 1 (READ + NOTIFY): gửi nhiệt độ giả lập

Characteristic 2 (WRITE): nhận lệnh bật/tắt LED

Notify mỗi 2 giây khi có device kết nối

Callback xử lý kết nối và ghi dữ liệu

#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLE2902.h>

#define SERVICE_UUID      "4fafc201-1fb5-459e-8fcc-c5c9c331914b"
#define SENSOR_CHAR_UUID  "beb5483e-36e1-4688-b7f5-ea07361b26a8"
#define LED_CHAR_UUID     "8ec90002-f315-4f60-9fb8-838830daea50"

BLECharacteristic* sensorChar;
BLECharacteristic* ledChar;
bool deviceConnected = false;
int ledPin = 2;

class ServerCB : public BLEServerCallbacks {
  void onConnect(BLEServer*) {
    deviceConnected = true;
    Serial.println("Device connected.");
  }
  void onDisconnect(BLEServer*) {
    deviceConnected = false;
    Serial.println("Device disconnected.");
    BLEDevice::startAdvertising();
  }
};

class LEDWriteCB : public BLECharacteristicCallbacks {
  void onWrite(BLECharacteristic *c) {
    String v = c->getValue();
    if (v == "1") { digitalWrite(ledPin, HIGH); Serial.println("LED ON"); }
    else          { digitalWrite(ledPin, LOW);  Serial.println("LED OFF"); }
  }
};

void setup() {
  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);

  BLEDevice::init("ESP32_BLE");
  BLEServer* server = BLEDevice::createServer();
  server->setCallbacks(new ServerCB());

  BLEService* service = server->createService(SERVICE_UUID);

  sensorChar = service->createCharacteristic(
      SENSOR_CHAR_UUID, BLECharacteristic::PROPERTY_READ | BLECharacteristic::PROPERTY_NOTIFY);
  sensorChar->addDescriptor(new BLE2902());

  ledChar = service->createCharacteristic(
      LED_CHAR_UUID, BLECharacteristic::PROPERTY_WRITE);
  ledChar->setCallbacks(new LEDWriteCB());

  service->start();
  BLEDevice::startAdvertising();
  Serial.println("BLE Server started!");
}

void loop() {
  if (deviceConnected) {
    int temp = random(20, 30);
    sensorChar->setValue(String(temp).c_str());
    sensorChar->notify();
    delay(2000);
  }
}

🔷 PART 3 — BLE COMMUNICATION GIỮA 2 ESP32

Cấu trúc:

ESP32 A → Peripheral: gửi "Temp: xx"

ESP32 B → Central: nhận dữ liệu & in Serial

Giải thích nhanh

A cập nhật giá trị characteristic mỗi 2 giây

B đọc lại characteristic liên tục

Đây là dạng BLE polling cơ bản

🟣 PART 4 — BLE NÂNG CAO
1️⃣ Notify

Server tự gửi dữ liệu khi thay đổi mà client không cần đọc lại.

2️⃣ Secure Pairing (Passkey)

ESP32 yêu cầu nhập mã PIN khi điện thoại kết nối

Tăng bảo mật BLE

3️⃣ Truyền chuỗi dài

BLE chỉ gửi ~20 bytes mỗi packet

Phải chia nhỏ → gửi → ghép lại phía client

3. Cách chạy

Mở Arduino IDE → chọn board ESP32 Dev Module

Tải thư viện ESP32 BLE Arduino

Nạp code từng phần:

part1 → Peripheral

part2 → Central

part3 → chạy 2 board

Mở app nRF Connect (Android/iOS) để test

4. Kết luận

Student sẽ hiểu rõ:

Quy trình BLE: Advertising → Connecting → GATT

Sự khác nhau giữa Central / Peripheral

Cách đọc/ghi characteristic

Notify, Pairing, truyền gói dữ liệu BLE

BLE là giải pháp tối ưu cho các ứng dụng IoT tầm ngắn, tiết kiệm năng lượng.

