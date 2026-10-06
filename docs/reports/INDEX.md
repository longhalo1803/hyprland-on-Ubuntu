# Bộ Tài Liệu Báo Cáo: Hệ Thống Hyprland Cô Lập Qua Nix Home-Manager Trên Ubuntu 24.04 LTS

Chào mừng bạn đến với bộ tài liệu hoàn chỉnh về kiến trúc, danh mục phần mềm, quy trình thiết lập và các giải pháp tối ưu hóa độ ổn định cho môi trường desktop **Hyprland** chạy trên nền tảng **Ubuntu 24.04 LTS** được cô lập bằng **Nix Home-Manager**.

---

## 📚 Danh Mục Các Báo Cáo Chi Tiết

| Tài Liệu | Tên Báo Cáo                                                                                           | Nội Dung Chính                                                                                                                                                                                   |
| :------: | :---------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|  **01**  | [**Kiến Trúc Hệ Thống & Mô Hình Cô Lập**](./01_SYSTEM_ARCHITECTURE.md)                                | Sơ đồ kiến trúc tổng thể, cơ chế cô lập glibc giữa Nix và Ubuntu host, thông số GPU Hybrid (Intel + Nvidia), cấu hình màn hình kép.                                                              |
|  **02**  | [**Danh Mục Package & Hệ Sinh Thái Giao Diện**](./02_PACKAGES_AND_ECOSYSTEM.md)                       | Chi tiết 15 packages trong `home.nix`, vai trò từng gói, nguồn gốc JaKooLit Hyprland-Dots v2.3.20 và sơ đồ thư mục `~/config/`.                                                                  |
|  **03**  | [**Hướng Dẫn Cấu Hình Từng Bước Từ Con Số 0**](./03_SETUP_AND_ISOLATION_GUIDE.md)                     | Quy trình tái lập môi trường hoàn chỉnh: từ cài đặt Nix, Home-Manager, clone dotfiles JaKooLit, thiết lập Quickshell Overview và Waybar.                                                         |
|  **04**  | [**Xử Lý Xung Đột & Tối Ưu Hóa Ổn Định**](./04_TROUBLESHOOTING_AND_STABILITY.md)                      | Phân tích chuyên sâu các lỗi kỹ thuật: Khởi tạo đồ họa EGL OpenGL, wrapper binary Nix (`.quickshell-wra`), cầu nối driver Mesa và cách ly Waybar đa màn hình.                                    |
|  **05**  | [**Hướng Dẫn Nâng Cấp, Bảo Trì & Duy Trì Độ Ổn Định Dài Hạn**](./05_MAINTENANCE_AND_UPGRADE_GUIDE.md) | Quy trình nâng cấp Nix packages an toàn, kiểm tra dry-run, cơ chế rollback thần tốc khi có lỗi, dọn dẹp dung lượng `/nix/store` và lưu ý kernel NVIDIA DKMS khi `apt upgrade`.                   |
|  **06**  | [**Môi Trường TUI, Neovim (LazyVim) & Tùy Biến Desktop**](./06_TUI_AND_EXTENSIONS_GUIDE.md)           | Bộ công cụ TUI (Kitty, tty-clock, btop, cava), thiết lập Neovim Nightly >= 0.11.2 (LazyVim, Prettier, Auto-save 500ms debounce), Quickshell Wallpaper Flow và bảng phím tắt `UserKeybinds.conf`. |

---

## ⚡ Các Điểm Chạm Kỹ Thuật Quan Trọng (Key Takeaways)

1. **Vị trí cấu hình phần mềm tập trung:**
   - File khai báo gói Nix: [`~/config/home-manager/home.nix`](~/.config/home-manager/home.nix)
   - Lệnh kích hoạt cấu hình mới: `home-manager switch`
2. **Cơ chế tăng tốc phần cứng cho ứng dụng GUI Nix:**
   - Đảm bảo có `pkgs.mesa` trong `home.nix`.
   - Các biến môi trường cầu nối: `LD_LIBRARY_PATH=$HOME/.nix-profile/lib`, `LIBGL_DRIVERS_PATH=$HOME/.nix-profile/lib/dri`, `GBM_BACKENDS_PATH=$HOME/.nix-profile/lib/gbm`.
3. **Thao tác cử chỉ & phím tắt kích hoạt Window Overview:**
   - Phím tắt: **`Super + A`**
   - Cử chỉ Touchpad: **Vuốt 3 ngón tay hướng lên**
   - Script điều khiển: [`~/config/hypr/scripts/OverviewToggle.sh`](~/.config/hypr/scripts/OverviewToggle.sh)
4. **Cấu hình màn hình hiển thị thanh Waybar:**
   - Ghi đè vĩnh viễn cho tất cả 39 layout tại: [`~/config/waybar/UserModules`](~/.config/waybar/UserModules)
