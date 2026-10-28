# Glass Launcher (Android TV)

Launcher kiểu Tizen: hàng app phía dưới, tile kính, nền Ambient đổi màu theo icon app đang chọn (giống YouTube Ambient Mode).
Viết bằng Kotlin + View thuần: **không** Compose, AppCompat, RecyclerView hay thư viện ngoài.

## Điều khiển
- Trái / Phải: chọn app. OK: mở app.
- Giữ OK (hoặc nút Menu): trang thông tin app (xoá cache, gỡ cài đặt...).
- Lên: thanh trên cùng (Cài đặt, Mạng, bật/tắt chuyển động Ambient).
- Back: về app đầu tiên. Home khi đang ở launcher: cũng về đầu.

## Build APK

### Cách 1: GitHub Actions (không cần cài gì)
1. Tạo repo GitHub, đẩy toàn bộ thư mục này lên nhánh `main`.
2. Vào tab **Actions** → chạy **Build APK** → tải artifact `GlassLauncher-apk`.

### Cách 2: Android Studio
Mở thư mục này → Build → Build APK(s). Chọn variant `release` (đã bật R8 và ký bằng debug key nên cài thẳng được).

### Cách 3: dòng lệnh
Cần JDK 17 và Android SDK (platform 34): `./gradlew assembleRelease`
File ra: `app/build/outputs/apk/release/app-release.apk`

## Cài lên TV
```
adb connect <ip-tv>:5555
adb install -r app-release.apk
adb shell cmd package set-home-activity com.glass.launcher/.MainActivity
```
Dòng cuối đặt làm launcher mặc định. Nếu TV vẫn mở launcher cũ khi bấm Home, mở app "Glass Launcher" từ danh sách ứng dụng rồi bấm Home và chọn "Luôn dùng".

## Tối ưu cho TV 1,5 GB RAM
- Cả hàng app là 1 View tự vẽ, không có child view, không layout khi cuộn, không cấp phát bộ nhớ trong onDraw.
- Không blur thật: kính được giả bằng gradient trong suốt + viền sáng + highlight, nền Ambient là 3 vệt sáng mờ (1 bitmap alpha 25 KB tô màu lúc vẽ).
- Hiệu ứng dùng 1 hàm làm mượt theo thời gian (không phụ thuộc FPS) và vòng lặp tự dừng khi xong, nên lúc rảnh gần như không tốn CPU.
- Ambient chạy 30 fps. Nút "Ambient" trên thanh trên cùng chuyển sang chế độ tĩnh (0% tải khi rảnh) nếu TV vẫn thấy nặng.
- Icon vẽ ở 144 px trên luồng nền, cache tối đa 8 MB, tự giải phóng khi hệ thống thiếu RAM.
