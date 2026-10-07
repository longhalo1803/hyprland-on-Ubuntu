# Báo Cáo 05: Hướng Dẫn Nâng Cấp, Bảo Trì & Duy Trì Độ Ổn Định Dài Hạn (Maintenance & Upgrade Guide)

---

## 1. Các Cấu Hình Sống Còn Để Hệ Thống Bền Vững & Không Xung Đột Với Ubuntu Host

Hệ thống của bạn đang chạy song song **Ubuntu 24.04 (môi trường máy chủ/GNOME Display Manager)** và **Hyprland (quản lý qua Nix)**. Để hai môi trường này không bao giờ "dẫm chân" lên nhau, các cấu hình sau đây là bắt buộc:

### 1.1. Đồng bộ D-Bus và Systemd User Environment

Trong file `~/.config/hypr/configs/Startup_Apps.conf`:

```ini
exec-once = dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDG_CURRENT_DESKTOP GTK_IM_MODULE QT_IM_MODULE XMODIFIERS
exec-once = systemctl --user import-environment WAYLAND_DISPLAY XDG_CURRENT_DESKTOP GTK_IM_MODULE QT_IM_MODULE XMODIFIERS
```

> [!IMPORTANT]
> **Tại sao bắt buộc?**
> Các tiến trình nền cấp User (PipeWire, WirePlumber, `xdg-desktop-portal`, GNOME Keyring) chạy qua `systemd --user`. Nếu không có 2 dòng này, các dịch vụ nền sẽ không biết cửa sổ Wayland nào đang mở, dẫn đến:
>
> - Không thể chia sẻ màn hình trên Discord / Google Meet / OBS.
> - Hộp thoại chọn file bị treo 25 giây.
> - Bàn phím gõ tiếng Việt (Fcitx5 / IBus) không nhận diện được ứng dụng.

### 1.2. Phân định Portal rõ ràng giữa GNOME và Hyprland

File `/usr/share/xdg-desktop-portal/hyprland-portals.conf` đã được thiết lập:

```ini
[preferred]
default=hyprland;gtk
```

Điều này ngăn chặn `xdg-desktop-portal` vô tình gọi portal của GNOME Shell khi bạn đang ngồi trong Hyprland, tránh xung đột tiến trình đồ họa.

### 1.3. Cơ chế xác thực đặc quyền Root (Polkit Agent)

Script `~/.config/hypr/scripts/Polkit.sh` đang nạp agent:
`/usr/lib/policykit-1-gnome/polkit-gnome-authentication-agent-1`
Agent này thuộc Ubuntu host, đảm bảo khi bạn chạy các lệnh quản trị hệ thống (`pkexec`, GParted, phần mềm phân vùng), cửa sổ nhập mật khẩu đồ họa luôn xuất hiện mượt mà.

---

## 2. Quy Trình Nâng Cấp Package Qua Nix Home-Manager Chuẩn Chỉ

Nix Home-Manager cho phép cập nhật phần mềm cô lập mà **không sợ hỏng hệ điều hành Ubuntu**. Tuy nhiên, cần tuân thủ 4 bước chuẩn an toàn:

### Bước 1: Kiểm tra thế hệ (Generation) đang chạy

Trước khi nâng cấp, xem thế hệ hiện tại để biết mốc an toàn có thể quay lại:

```bash
home-manager generations
```

_(Ghi nhớ số ID hiện tại, ví dụ: `id 16`)._

### Bước 2: Cập nhật chỉ mục gói (Channel Update)

```bash
nix-channel --update
```

### Bước 3: Chạy thử nghiệm biên dịch (Dry Run / Build Check)

Không vội áp dụng ngay, hãy kiểm tra xem bản cập nhật có lỗi syntax hay thiếu package nào không:

```bash
home-manager build
```

Nếu lệnh này hoàn thành mà không báo lỗi đỏ, bạn an tâm 100% để chuyển đổi.

### Bước 4: Kích hoạt cấu hình mới (Switch)

```bash
home-manager switch
```

---

## 3. Cơ Chế "Cứu Hộ Thần Tốc" (Instant Rollback) Khi Cập Nhật Bị Lỗi

Nếu sau khi `home-manager switch`, một gói phần mềm (như Waybar hoặc Quickshell) bị crash do phiên bản mới không tương thích với driver máy:

### Cách 1: Rollback bằng số Generation ID

```bash
# Xem danh sách các mốc thời gian
home-manager generations

# Quay trở lại thế hệ trước (ví dụ ID 16)
~/.local/state/nix/profiles/home-manager-16-link/activate
```

### Cách 2: Kích hoạt trực tiếp từ Nix Store (kể cả khi lệnh `home-manager` bị lỗi)

Mỗi thế hệ được lưu thành một thư mục riêng biệt trong `/nix/store/`. Bạn có thể kích hoạt lại tức thì bằng cách chạy script `activate` của thế hệ đó:

```bash
/nix/store/<hash>-home-manager-generation/activate
```

Hệ thống sẽ lập tức phục hồi lại toàn bộ symlink và phiên bản cũ trong 1 giây mà không cần tải lại bất kỳ file nào!

---

## 4. Quản Lý Dọn Dẹp Dung Lượng Ổ Đĩa (Nix Garbage Collection)

Nix giữ lại các thế hệ cũ để bạn có thể rollback, do đó `/nix/store` sẽ tăng dần dung lượng theo thời gian (hiện tại máy bạn đang chiếm mức rất gọn gàng: **5.6 GB**).

### Quy tắc dọn dẹp an toàn:

> [!WARNING]
> **Không chạy `nix-collect-garbage -d` một cách bừa bãi** nếu bạn vừa mới cấu hình xong một tính năng mới, vì nó sẽ xóa sạch tất cả các thế hệ cũ và bạn sẽ mất khả năng rollback!

**Quy trình dọn dẹp chuẩn định kỳ (Mỗi tháng 1 lần):**

1. Giữ lại các thế hệ trong vòng 14 ngày gần nhất, chỉ xóa các bản cũ hơn:
   ```bash
   home-manager expire-generations "-14 days"
   ```
2. Thu hồi dung lượng ổ cứng từ các file không còn dùng:
   ```bash
   nix-collect-garbage
   ```
3. Tối ưu hóa các file trùng lặp (Hardlink deduplication):
   ```bash
   nix-store --optimise
   ```

---

## 5. Quy Trình Cập Nhật Ubuntu Host (`apt upgrade`) An Toàn Cho NVIDIA

Laptop của bạn sử dụng card đồ họa **NVIDIA GeForce RTX 3050 Ti Mobile** kết hợp với **Intel Iris Xe**.

### Rủi ro lớn nhất:

Khi Ubuntu cập nhật Kernel mới (`linux-image-6.8.0-xx-generic`), driver NVIDIA độc quyền phải được biên dịch lại tương ứng qua công cụ **DKMS**. Nếu bạn khởi động lại máy khi quá trình biên dịch DKMS chưa xong, Hyprland sẽ không thể khởi động được (rơi vào màn hình đen tty).

### Quy trình cập nhật an toàn:

1. Chạy cập nhật hệ thống:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. **BẮT BUỘC KIỂM TRA TRƯỚC KHI REBOOT:**
   Kiểm tra xem module NVIDIA đã được build thành công cho kernel mới chưa:
   ```bash
   dkms status
   ```
   _Kết quả phải có dòng: `nvidia/...: installed` tương ứng với kernel mới nhất._
3. Kiểm tra card NVIDIA phản hồi tốt:
   ```bash
   nvidia-smi
   ```
4. Chỉ khởi động lại máy khi các kiểm tra trên đều báo thành công.

---

## 6. Quy Trình Cập Nhật Dotfiles JaKooLit Mà Không Bị Mất Cấu Hình

Bộ dotfiles của JaKooLit được thiết kế rất thông minh theo nguyên tắc tách biệt:

- **Thư mục Core (do tác giả cập nhật):** `~/.config/hypr/configs/`, `~/.config/waybar/configs/`
- **Thư mục Cá Nhân (bất khả xâm phạm):**
  - `~/.config/hypr/UserConfigs/`
  - `~/.config/hypr/UserScripts/`
  - `~/.config/waybar/UserModules` (Nơi chứa cấu hình `"output": "eDP-1"`)

### Các bước khi bạn muốn kéo phiên bản mới của JaKooLit:

1. Luôn sao lưu thư mục cấu hình hiện tại:
   ```bash
   cp -r ~/.config/hypr ~/.config/hypr.backup.$(date +%F)
   cp -r ~/.config/waybar ~/.config/waybar.backup.$(date +%F)
   ```
2. Sau khi cập nhật dotfiles mới, kiểm tra lại:
   - Dòng nạp Quickshell trong `Startup_Apps.conf` vẫn giữ nguyên `env LD_LIBRARY_PATH=...`.
   - File `OverviewToggle.sh` vẫn sử dụng `pgrep -f 'qs -c overview'`.
   - File `UserModules` vẫn chứa `"output": "eDP-1"`.
