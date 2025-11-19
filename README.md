# ESP32-Lab2-BLE
Bài thực hành ESP32 BLE – Exercise 2
📘 README – Bài thực hành ESP32 BLE (Bluetooth Low Energy)
1. Giới thiệu

Bài thực hành này giúp sinh viên hiểu và tự triển khai các chức năng BLE của ESP32 gồm:

BLE Peripheral (GATT Server)

BLE Central (GATT Client)

BLE 2 ESP32 (Client ↔ Server)

BLE nâng cao: Notify, Pairing, truyền chuỗi dài…

Các ví dụ được lập trình bằng Arduino IDE sử dụng thư viện ESP32 BLE Arduino.
Exercise2/
│── part1_BLE_Peripheral/
│    └── main.ino
│── part2_BLE_Central/
│    └── main.ino
│── part3_BLE_2_ESP32/
│    ├── server_esp32.ino
│    └── client_esp32.ino
│── part4_BLE_Advanced/
│    ├── BLE_notify.ino
│    ├── BLE_pairing.ino
│    └── BLE_long_string.ino
└── README.md   ← file này

3. Yêu cầu phần mềm & phần cứng
Phần mềm

Arduino IDE 2.x

Board ESP32 package

Library:

ESP32 BLE Arduino

ArduinoJSON (nếu dùng thêm)

Phần cứng

1–2 board ESP32 DevKit

Điện thoại Android / iOS có app nRF Connect hoặc LightBlue

4. Thực hành & Giải thích chi tiết mã nguồn
📌 Part 1 – ESP32 BLE Peripheral (GATT Server)
Mục tiêu

ESP32 phát BLE Advertising (tên: ESP32_BLE)

Tạo 1 BLE Service + 1 Characteristic

Điện thoại kết nối → đọc/ghi dữ liệu
