# Báo Cáo 04: Xử Lý Xung Đột & Tối Ưu Hóa Ổn Định (Troubleshooting & Stability Fixes)

Tài liệu này ghi lại toàn bộ các "nút thắt cổ chai" kỹ thuật thực tế phát sinh khi kết hợp Nix Home-Manager trên nền Ubuntu 24.04 LTS (môi trường non-NixOS) cùng giải pháp khắc phục triệt để đã áp dụng thành công.

---

## 1. Xung Đột Đồ Họa & Ngữ Cảnh Dựng Hình (EGL / OpenGL Backend Failure)

### Triệu chứng ban đầu

Khi chạy `qs -c overview` hoặc gọi IPC `overview toggle`, tiến trình không thể hiển thị giao diện thẻ bài Overview lên màn hình và ghi nhật ký:

```text
WARN qt.qpa.wayland: EGL not available
WARN: QRhiGles2: Failed to create temporary context
WARN: QRhiGles2: Failed to create context
WARN: Failed to create RHI (backend 2)
ERROR: Failed to create graphics context for qs::wayland::layershell::WlrLayershell: "Failed to initialize graphics backend for OpenGL."
```

### Phân tích nguyên nhân gốc rễ

1. **Cô lập Dynamic Linker:** Quickshell được biên dịch bởi Nix, sử dụng ELF interpreter độc lập `/nix/store/...-glibc-2.42-84/lib/ld-linux-x86-64.so.2`. Thư viện `libglvnd` của Nix khi tìm kiếm ICD driver đồ họa chỉ tìm trong `/run/opengl-driver` (đường dẫn đặc thù chỉ có trên NixOS), không thể nhìn thấy driver Mesa của Ubuntu tại `/usr/lib/x86_64-linux-gnu/dri/`.
2. **Nguy cơ xung đột glibc (libc symbol lookup error):**
   Nếu cố tình gán `LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu` để trỏ về Ubuntu, hệ thống sẽ gặp lỗi đổ vỡ thư viện nghiêm trọng:
   ```text
   qs: symbol lookup error: /usr/lib/x86_64-linux-gnu/libc.so.6: undefined symbol: __nptl_change_stack_perm, version GLIBC_PRIVATE
   ```
   Do binary của Nix tải `libc.so.6` của Ubuntu 24.04 (`glibc 2.39`) thay vì `glibc 2.42` của Nix.

### Giải pháp triệt để đã áp dụng

- **Cài đặt `pkgs.mesa` trực tiếp qua Home-Manager:** Biên dịch driver EGL/DRI/GBM đồng bộ 100% với phiên bản glibc của Nix.
- **Tạo cầu nối biến môi trường (Environment Bridge):**
  ```bash
  export LD_LIBRARY_PATH="$HOME/.nix-profile/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
  export LIBGL_DRIVERS_PATH="$HOME/.nix-profile/lib/dri"
  export GBM_BACKENDS_PATH="$HOME/.nix-profile/lib/gbm"
  ```
  Nhờ đó, QtWayland nạp thành công `libEGL_mesa.so` và driver Iris/Intel TigerLake mà không đụng chạm đến thư viện hệ thống mẹ.

---

## 2. Lỗi Nhận Diện Tiến Trình Do Cơ Chế C Wrapper Của Nix

### Triệu chứng

Trong script gốc `~/.config/hypr/scripts/OverviewToggle.sh`, lệnh:

```bash
if pgrep -x quickshell >/dev/null 2>&1; then ...
```

luôn trả về giá trị `1` (thất bại, không tìm thấy tiến trình), dẫn đến việc script liên tục spawn tiến trình mới (`qs -c overview &`) mỗi khi người dùng bấm phím tắt, gây ra hàng loạt tiến trình zombie và đụng độ socket.

### Phân tích nguyên nhân

Nix sử dụng tiện ích `makeCWrapper` để bọc binary gốc của Quickshell. Do đó, tên tiến trình ghi nhận trong kernel (`/proc/<pid>/comm`) thực tế là:

```text
.quickshell-wra
```

Vì cờ `-x` (exact match) của lệnh `pgrep` chỉ so sánh chuỗi chính xác với `/proc/<pid>/comm`, nó không thể tìm thấy tiến trình có tên `quickshell`.

### Giải pháp đã áp dụng

Cập nhật cú pháp trong `~/.config/hypr/scripts/OverviewToggle.sh`:

```bash
if pgrep -f 'qs -c overview' >/dev/null 2>&1 || pidof qs >/dev/null 2>&1; then
    ...
```

Cờ `-f` cho phép đối chiếu trên toàn bộ chuỗi dòng lệnh (full command line), nhận diện hoàn hảo cả wrapper và binary thực thi của Nix.

---

## 3. Cấu Trúc Wrapper Chuyên Dụng Tại `~/.local/bin/`

Để đảm bảo mọi lệnh gọi CLI như `qs` hoặc `quickshell` ở bất kỳ đâu (từ Hyprland, Terminal, hay script thứ 3) đều tự động mang theo môi trường đồ họa, chúng tôi đã tạo wrapper:

- **File `~/.local/bin/qs`:**
  ```bash
  #!/usr/bin/env bash
  export LD_LIBRARY_PATH="$HOME/.nix-profile/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
  export LIBGL_DRIVERS_PATH="$HOME/.nix-profile/lib/dri"
  export GBM_BACKENDS_PATH="$HOME/.nix-profile/lib/gbm"
  exec "$HOME/.nix-profile/bin/qs" "$@"
  ```
- **Thứ tự ưu tiên PATH:** Bổ sung `env = PATH,$HOME/.local/bin:$PATH` vào đầu `~/.config/hypr/configs/ENVariables.conf`, đảm bảo Hyprland luôn ưu tiên wrapper này trước binary thô của Nix.

---

## 4. Quản Lý Đa Màn Hình Trên Waybar (Multi-Monitor Output Isolation)

### Triệu chứng

Mặc định bộ dotfiles JaKooLit hiển thị thanh bar đồng thời trên cả màn hình laptop (`eDP-1`) và màn hình rời (`HDMI-A-1`). Việc này gây rối mắt và không đúng nhu cầu của người dùng muốn thanh trạng thái chỉ nằm ở màn hình chính.

### Cơ chế xử lý của Waybar

Waybar hỗ trợ thuộc tính `"output"` ở cấp gốc (root level) của JSON config. Giá trị có thể là tên màn hình đơn lẻ (`"eDP-1"`), danh sách (`["eDP-1", "HDMI-A-1"]`), hoặc cú pháp phủ định (`["!HDMI-A-1"]`).

### Điểm chạm tối ưu nhất (UserModules Hook)

Thay vì chỉnh sửa trực tiếp vào file layout hiện tại (sẽ bị mất khi người dùng chọn đổi giao diện qua Rofi menu), chúng tôi xác định toàn bộ 39 layout trong `~/.config/waybar/configs/` đều nạp file:
👉 `~/.config/waybar/UserModules`

Bằng cách khai báo:

```json
{
  "output": "eDP-1"
}
```

Thiết lập sẽ được tự động kế thừa và bảo toàn 100% qua tất cả các đợt đổi theme hay update dotfiles.
