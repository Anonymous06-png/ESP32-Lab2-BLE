🚀 ESP32 – Lab2: BLE (Bluetooth Low Energy)
📘 1. Giới thiệu

Bài thực hành này giúp sinh viên nắm vững cách triển khai các chức năng BLE trên ESP32, bao gồm:

BLE Peripheral (GATT Server)

BLE Central (GATT Client)

BLE 2 ESP32 giao tiếp qua BLE

BLE nâng cao: Notify, Pairing, truyền chuỗi dài,...

Toàn bộ ví dụ được lập trình bằng Arduino IDE với thư viện ESP32 BLE Arduino.


## 📁 2. Cấu trúc thư mục `Exercise2/`
Exercise2
│
├── 📂 part1_BLE_Peripheral/
│ └── 📄 main.ino
│
├── 📂 part2_BLE_Central/
│ └── 📄 main.ino
│
├── 📂 part3_BLE_2_ESP32/
│ ├── 📄 server_esp32.ino
│ └── 📄 client_esp32.ino
│
├── 📂 part4_BLE_Advanced/
│ ├── 📄 BLE_notify.ino
│ ├── 📄 BLE_pairing.ino
│ └── 📄 BLE_long_string.ino
│
└── 📄 README.md ← file mô tả này

🔧 3. Phần mềm & thư viện yêu cầu

Arduino IDE 2.x

ESP32 Board Package

Thư viện:

ESP32 BLE Arduino

(Tùy chọn) ArduinoJSON

🟦 4. Part 1 – BLE Peripheral (GATT Server)
📌 Chức năng

ESP32 phát BLE advertising (ESP32_BLE)

Tạo một service + characteristic READ/WRITE

Kết nối bằng app nRF Connect để đọc/ghi dữ liệu

💡 Code chính
BLEDevice::init("ESP32_BLE");  // Khởi tạo BLE và đặt tên quảng bá

BLEServer *pServer = BLEDevice::createServer();  
// Tạo BLE server – ESP32 đóng vai trò Peripheral

BLEService *pService = pServer->createService(SERVICE_UUID);
// Tạo service có UUID riêng

BLECharacteristic *pCharacteristic = pService->createCharacteristic(
    CHARACTERISTIC_UUID,
    BLECharacteristic::PROPERTY_READ | BLECharacteristic::PROPERTY_WRITE
);
// Tạo characteristic có quyền READ & WRITE

▶️ Kết quả mong đợi

App nRF Connect thấy ESP32_BLE

Đọc được chuỗi "Hello from ESP32"

Ghi dữ liệu từ điện thoại → hiển thị trên Serial Monitor

🟩 5. Part 2 – BLE Central (GATT Client)
📌 Chức năng

ESP32 quét BLE xung quanh

Kết nối đến ESP32 Peripheral

Đọc/ghi characteristic

💡 Code chính
BLEScan* pScan = BLEDevice::getScan();
pScan->setActiveScan(true);  // Scan chủ động, tốc độ nhanh hơn

BLEScanResults results = pScan->start(5);


→ ESP32 sẽ tìm xem có thiết bị nào quảng bá đúng SERVICE_UUID không.

🟧 6. Part 3 – Hai ESP32 giao tiếp BLE
ESP32 A (Peripheral) → gửi dữ liệu → ESP32 B (Central)


Khi chạy song song:

ESP32 A gửi chuỗi "Temp: xx" (giả lập)

ESP32 B nhận → in Serial

🟪 7. Part 4 – BLE nâng cao
🟣 Notify

ESP32 server tự động gửi dữ liệu khi thay đổi (không cần polling).

🔒 Secure Pairing

Cấu hình passkey → điện thoại phải nhập mã mới kết nối.

📦 Truyền chuỗi dài

Chia nhỏ gói (MTU ~ 20 bytes), ghép lại ở phía client.

📌 8. Kết luận

Thông qua lab này, sinh viên hiểu được:

Kiến trúc BLE: Advertising → Connecting → GATT

Sự khác nhau giữa Peripheral và Central

Kỹ thuật đọc/ghi Characteristic

Cách mở rộng BLE: notify, pairing, truyền dữ liệu

BLE phù hợp các ứng dụng IoT tầm ngắn, tiêu thụ năng lượng thấp.
