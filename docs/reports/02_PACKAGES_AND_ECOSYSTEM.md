# Báo Cáo 02: Danh Mục Package & Hệ Sinh Thái Giao Diện (Packages & Ecosystem)

---

## 1. Danh Mục Các Gói Quản Lý Bởi Nix Home-Manager

Toàn bộ các gói phần mềm phục vụ môi trường Hyprland được khai báo tập trung tại [`~/config/home-manager/home.nix`](~/.config/home-manager/home.nix):

```nix
home.packages = with pkgs; [
  kitty
  waybar
  mako
  fuzzel
  fastfetch
  rofi
  pamixer
  brightnessctl
  wlogout
  grim
  slurp
  swappy
  wl-clipboard
  quickshell
  mesa
];
```

### Bảng phân tích chi tiết từng gói và vai trò:

| Package             | Phiên Bản Cài Đặt | Vai Trò Trong Hệ Thống Desktop                                                                                       |
| :------------------ | :---------------- | :------------------------------------------------------------------------------------------------------------------- |
| **`kitty`**         | v0.48.2           | Terminal Emulator tăng tốc phần cứng GPU (OpenGL), hỗ trợ hiển thị hình ảnh inline, font ligatures                   |
| **`waybar`**        | v0.15.0           | Thanh trạng thái đa năng (Status Bar), hiển thị Workspace, Pin, Tải CPU/RAM, Âm lượng, Đồng hồ, Khay hệ thống        |
| **`quickshell`**    | v0.3.1            | Framework dựng giao diện desktop dựa trên Qt6/QML và Wayland Layer Shell; phụ trách tính năng **Window Overview**    |
| **`mesa`**          | v26.2.2           | Thư viện driver đồ họa tăng tốc phần cứng (EGL, DRI, GBM) của Nix, cầu nối để các ứng dụng GUI Nix nhận diện GPU máy |
| **`rofi`**          | v1.7.5+           | Trình khởi chạy ứng dụng (App Launcher), menu chuyển đổi Layout Waybar, Menu đổi hình nền                            |
| **`fuzzel`**        | v1.10+            | Menu tìm kiếm ứng dụng tối giản, siêu nhẹ trên Wayland (dùng làm launcher dự phòng)                                  |
| **`mako`**          | v1.9+             | Daemon hiển thị thông báo popup nhẹ nhàng trên Wayland                                                               |
| **`pamixer`**       | v1.6+             | Công cụ dòng lệnh điều khiển âm lượng hệ thống qua PipeWire / PulseAudio                                             |
| **`brightnessctl`** | v0.5.1            | Công cụ điều chỉnh độ sáng màn hình laptop qua phím chức năng Fn                                                     |
| **`wlogout`**       | v1.2+             | Menu tắt máy, khởi động lại, khóa màn hình, đăng xuất toàn màn hình dạng đồ họa                                      |
| **`grim`**          | v1.4+             | Công cụ chụp ảnh màn hình Wayland                                                                                    |
| **`slurp`**         | v1.5+             | Công cụ chọn vùng màn hình tương tác bằng chuột                                                                      |
| **`swappy`**        | v1.5+             | Trình chỉnh sửa ảnh chụp màn hình nhanh (vẽ mũi tên, text, làm mờ, highlight)                                        |
| **`wl-clipboard`**  | v2.2+             | Bộ công cụ quản lý clipboard trên Wayland (`wl-copy`, `wl-paste`)                                                    |
| **`fastfetch`**     | v2.20+            | Tiện ích hiển thị thông tin phần cứng, logo phân phối và cấu hình hệ điều hành trong terminal                        |

---

## 2. Nguồn Gốc Giao Diện & Bộ Cấu Hình JaKooLit Hyprland-Dots

- **Tác giả:** JaKooLit (nhà phát triển cộng đồng nổi tiếng về các bản phân phối Hyprland đẹp và ổn định nhất).
- **Mã nguồn gốc:** [JaKooLit/Hyprland-Dots](https://github.com/JaKooLit/Hyprland-Dots) (Nhánh hỗ trợ đa nền tảng Ubuntu / Debian / Arch / NixOS).
- **Phiên bản cấu hình đang chạy:** **`v2.3.20`** (xác định qua `DOTS_VERSION=2.3.20` trong [`~/config/hypr/configs/ENVariables.conf`](~/.config/hypr/configs/ENVariables.conf)).

---

## 3. Bản Đồ Cấu Trúc Các Thư Mục Cấu Hình (`~/config/`)

### 1. `~/config/hypr/` (Trung tâm điều khiển Hyprland)

- **`configs/`:**
  - [`Keybinds.conf`](~/.config/hypr/configs/Keybinds.conf): Định nghĩa toàn bộ phím tắt (`Super + Q`, `Super + A`, `Super + Return`, v.v.).
  - [`Startup_Apps.conf`](~/.config/hypr/configs/Startup_Apps.conf): Danh sách các ứng dụng/daemon tự khởi động khi đăng nhập.
  - [`ENVariables.conf`](~/.config/hypr/configs/ENVariables.conf): Khai báo biến môi trường toàn cục (toolkit Qt, GTK, NVIDIA, PATH).
  - [`SystemSettings.conf`](~/.config/hypr/configs/SystemSettings.conf): Cấu hình touchpad, cử chỉ đa điểm (`gesture = 3, up, dispatcher, exec...`), hoạt ảnh animation, border.
  - [`Monitors.conf`](~/.config/hypr/configs/Monitors.conf): Độ phân giải, vị trí và tần số quét của các màn hình.
- **`scripts/`:**
  - [`OverviewToggle.sh`](~/.config/hypr/scripts/OverviewToggle.sh): Kịch bản kích hoạt và chuyển đổi giao diện Overview.
  - [`Refresh.sh`](~/.config/hypr/scripts/Refresh.sh): Nạp lại Waybar, SwayNC, Rofi mà không cần khởi động lại máy.
  - [`LockScreen.sh`](~/.config/hypr/scripts/LockScreen.sh): Kích hoạt màn hình khóa Hyprlock.

### 2. `~/config/waybar/` (Thanh trạng thái Waybar)

- **`configs/`:** Chứa 39 kiểu dáng (layout) khác nhau (`[TOP] Peony`, `[TOP] Sleek`, `[BOT] Camellia`, v.v.).
- **`Modules` & `ModulesWorkspaces`:** Định nghĩa các widget hiển thị.
- **`UserModules`:** Nơi người dùng ghi đè cấu hình cá nhân (được `include` trong tất cả 39 layout).

### 3. `~/config/quickshell/` (Bộ Widget QML Overview)

- **`overview/shell.qml`:** Điểm khởi đầu nạp toàn bộ cấu hình QML.
- **`overview/modules/overview/`:**
  - `Overview.qml`: Tạo `PanelWindow` trên Layer Shell cấp độ `Overlay` và nhận lệnh IPC `open`/`close`/`toggle`.
  - `OverviewWidget.qml`: Lưới các workspace thu nhỏ và bố cục hiển thị cửa sổ.
  - `OverviewWindow.qml`: Thẻ bài đại diện cho từng cửa sổ đang mở.
- **`overview/services/`:**
  - `GlobalStates.qml`: Lưu trữ trạng thái đóng/mở của Overview.
  - `HyprlandData.qml`: Lấy dữ liệu cửa sổ, workspace và màn hình thời gian thực qua socket IPC của Hyprland.

### 4. Các Daemon Hệ Thống Khác Đang Chạy

- **`swww-daemon`:** Daemon quản lý hình nền động, hiệu ứng chuyển cảnh wallpaper mượt mà.
- **`swaync`:** Trung tâm thông báo (Notification Center) tích hợp bảng điều khiển nhanh (Quick Settings).
- **`hypridle`:** Quản lý tiết kiệm điện (tắt màn hình, khóa máy khi không hoạt động).
- **`hyprsunset`:** Điều chỉnh nhiệt độ màu ánh sáng xanh (Night Light).
- **`cliphist`:** Daemon ghi nhớ lịch sử khay nhớ tạm (clipboard) cả văn bản và hình ảnh.
