# Báo Cáo 02: Danh Mục Package & Hệ Sinh Thái Giao Diện (Packages & Ecosystem)

---

## 1. Danh Mục Các Gói Quản Lý Bởi Nix Home-Manager

Toàn bộ các gói phần mềm phục vụ môi trường Hyprland được khai báo tập trung tại `~/.config/home-manager/home.nix`:

```nix
home.packages = with pkgs; [
  kitty-wrapped
  quickshell-wrapped
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
  mesa

  # Audio visualizer
  cava
];
```

### Bảng phân tích chi tiết từng gói và vai trò:

| Package             | Phiên Bản Cài Đặt | Vai Trò Trong Hệ Thống Desktop                                                    |
| :------------------ | :---------------- | :-------------------------------------------------------------------------------- |
| **`kitty`**         | v0.48.2           | Terminal Emulator GPU OpenGL (được wrap driver Mesa từ Nix Store)                 |
| **`waybar`**        | v0.15.0           | Thanh trạng thái đa năng (Status Bar) khóa trên màn hình chính eDP-1              |
| **`quickshell`**    | v0.3.1            | Framework Qt6/QML Wayland Layer Shell phụ trách **Window Overview**               |
| **`cava`**          | v1.0.0            | Audio visualizer trực quan hóa sóng âm thanh (tự đồng bộ màu nền & chữ của Kitty) |
| **`mesa`**          | v26.2.2           | Thư viện driver đồ họa tăng tốc phần cứng (EGL, DRI, GBM) của Nix                 |
| **`rofi`**          | v2.0.0            | Trình khởi chạy ứng dụng (App Launcher), menu Waybar layout, đổi theme            |
| **`fuzzel`**        | v1.14.1           | Menu tìm kiếm ứng dụng Wayland siêu nhẹ                                           |
| **`mako`**          | v1.11.0           | Daemon hiển thị thông báo popup                                                   |
| **`pamixer`**       | v1.6              | Công cụ dòng lệnh điều khiển âm lượng hệ thống qua PipeWire                       |
| **`brightnessctl`** | v0.5.1            | Công cụ điều chỉnh độ sáng màn hình                                               |
| **`wlogout`**       | v1.2.2            | Menu tắt máy, khởi động lại, khóa màn hình đồ họa                                 |
| **`grim`**          | v1.5.0            | Công cụ chụp ảnh màn hình Wayland                                                 |
| **`slurp`**         | v1.5.0            | Công cụ chọn vùng màn hình tương tác bằng chuột                                   |
| **`swappy`**        | v1.8.0            | Trình chỉnh sửa ảnh chụp màn hình nhanh                                           |
| **`wl-clipboard`**  | v2.3.0            | Bộ công cụ quản lý clipboard trên Wayland (`wl-copy`, `wl-paste`)                 |
| **`fastfetch`**     | v2.68.1           | Tiện ích hiển thị thông tin phần cứng và distro                                   |

### 1.2. Hệ Sinh Thái TUI & Trình Biên Tập Mã Nguồn Bổ Trợ (TUI & Dev Stack)

Các công cụ dòng lệnh đồ họa (TUI) được thiết lập độc lập nhằm tối ưu hiệu năng và thẩm mỹ:

| Công Cụ         | Phiên Bản | Nguồn Cài Đặt            | Vai Trò & Điểm Nổi Bật                                                                    |
| :-------------- | :-------- | :----------------------- | :---------------------------------------------------------------------------------------- |
| **`neovim`**    | v0.11.x+  | GitHub Release (Nightly) | Trình soạn thảo chính (LazyVim Distro, Prettier, Auto-save 500ms, TokyoNight Transparent) |
| **`tty-clock`** | v2.3+     | APT / Source             | Đồng hồ số digital căn giữa màn hình (`tty-clock -c -C 7 -s -b`)                          |
| **`btop`**      | v1.4.0+   | APT / Nix                | Trình theo dõi tiến trình và tài nguyên phần cứng (CPU, GPU, RAM)                         |
| **`cmatrix`**   | v2.0+     | APT                      | Hiệu ứng mưa mã nguồn màn hình chờ                                                        |

---

## 2. Nguồn Gốc Giao Diện & Bộ Cấu Hình JaKooLit Hyprland-Dots

- **Tác giả:** JaKooLit (nhà phát triển cộng đồng nổi tiếng về các bản phân phối Hyprland đẹp và ổn định nhất).
- **Mã nguồn gốc:** [JaKooLit/Hyprland-Dots](https://github.com/JaKooLit/Hyprland-Dots) (Nhánh hỗ trợ đa nền tảng Ubuntu / Debian / Arch / NixOS).
- **Phiên bản cấu hình đang chạy:** **`v2.3.20`** (xác định qua `DOTS_VERSION=2.3.20` trong `~/.config/hypr/configs/ENVariables.conf`).

---

## 3. Bản Đồ Cấu Trúc Các Thư Mục Cấu Hình (`~/.config/`)

### 1. `~/.config/hypr/` (Trung tâm điều khiển Hyprland)

- **`configs/`:**
  - `Keybinds.conf`: Định nghĩa toàn bộ phím tắt (`Super + Q`, `Super + A`, `Super + Return`, v.v.).
  - `Startup_Apps.conf`: Danh sách các ứng dụng/daemon tự khởi động khi đăng nhập.
  - `ENVariables.conf`: Khai báo biến môi trường toàn cục (toolkit Qt, GTK, NVIDIA, PATH).
  - `SystemSettings.conf`: Cấu hình touchpad, cử chỉ đa điểm (`gesture = 3, up, dispatcher, exec...`), hoạt ảnh animation, border.
  - `Monitors.conf`: Độ phân giải, vị trí và tần số quét của các màn hình.
- **`scripts/`:**
  - `OverviewToggle.sh`: Kịch bản kích hoạt và chuyển đổi giao diện Overview.
  - `Refresh.sh`: Nạp lại Waybar, SwayNC, Rofi mà không cần khởi động lại máy.
  - `LockScreen.sh`: Kích hoạt màn hình khóa Hyprlock.

### 2. `~/.config/waybar/` (Thanh trạng thái Waybar)

- **`configs/`:** Chứa 39 kiểu dáng (layout) khác nhau. Bố cục đang kích hoạt: `[TOP] Default Laptop-glass`.
- **`style/`:** Chứa các bộ CSS giao diện. Bộ CSS đang kích hoạt: `[Kitty] Islands-Glass.css` (bo góc tròn, kính mờ theo Kitty).
- **`Modules` & `ModulesWorkspaces`:** Định nghĩa các widget hiển thị.
- **`UserModules`:** Nơi người dùng ghi đè cấu hình cá nhân (khóa cố định output vào `eDP-1` cho laptop).

### 3. `~/.config/quickshell/` (Hệ Sinh Thái QML Desktop Shell)

- **`overview/shell.qml`:** Bộ Window Overview kích hoạt qua `Super + A` hoặc vuốt 3 ngón tay.
- **`wallpaper-flow/shell.qml`:** Bộ chọn hình nền **Parallelogram 2D Flow** độc lập kích hoạt qua `Super + W`.
  - Hiển thị danh sách hình nền dạng thẻ bài vát góc, tỷ lệ rộng không chồng lấn.
  - Tích hợp `backend.sh` sinh cache thumbnail tốc độ cao và script điều khiển `WallpaperFlowToggle.sh`.
- **`overview/services/`:**
  - `GlobalStates.qml`: Lưu trữ trạng thái đóng/mở của Overview.
  - `HyprlandData.qml`: Lấy dữ liệu cửa sổ, workspace và màn hình thời gian thực qua socket IPC của Hyprland.

### 4. Các Daemon Hệ Thống Khác Đang Chạy

- **`swww-daemon`:** Daemon quản lý hình nền động, hiệu ứng chuyển cảnh wallpaper mượt mà.
- **`swaync`:** Trung tâm thông báo (Notification Center) tích hợp bảng điều khiển nhanh (Quick Settings).
- **`hypridle`:** Quản lý tiết kiệm điện (tắt màn hình, khóa máy khi không hoạt động).
- **`hyprsunset`:** Điều chỉnh nhiệt độ màu ánh sáng xanh (Night Light).
- **`cliphist`:** Daemon ghi nhớ lịch sử khay nhớ tạm (clipboard) cả văn bản và hình ảnh.
