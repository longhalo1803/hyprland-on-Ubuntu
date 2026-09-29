# Báo Cáo 03: Hướng Dẫn Cấu Hình Từng Bước Từ Con Số 0 (Step-by-Step Setup Guide)

> [!IMPORTANT]
> **Phiên bản chuẩn hóa 100% tái tạo môi trường:**
> Tài liệu này được biên soạn và kiểm duyệt để đảm bảo bất kỳ máy tính chạy **Ubuntu 24.04 LTS mới tinh (Clean Install)** nào làm theo đúng tuần tự các bước dưới đây đều sẽ thiết lập thành công 100% môi trường Hyprland cô lập bằng Nix Home-Manager mà không gặp lỗi thiếu gói, lỗi font, hay lỗi khởi tạo đồ họa.

---

## Giai Đoạn 1: Cài Đặt Hyprland Native & Các Dịch Vụ Nền Trên Ubuntu 24.04

Vì Ubuntu 24.04 mặc định không có sẵn Hyprland v0.56.2+ trong kho APT chính thức, chúng ta cần nạp PPA chuẩn và cài đặt compositor cùng các daemon hệ thống:

### 1. Thêm PPA và cài đặt Hyprland + Portal + Daemons

```bash
sudo apt update
sudo apt install -y software-properties-common curl git build-essential libnotify-bin

# Thêm PPA chứa Hyprland và các tiện ích tương thích Ubuntu 24.04 (Noble)
sudo add-apt-repository -y ppa:cppiber/hyprland
sudo apt update

# Cài đặt Hyprland compositor, Portal, màn hình khóa và daemon thông báo
sudo apt install -y hyprland xdg-desktop-portal-hyprland hypridle hyprlock sway-notification-center policykit-1-gnome
```

### 2. Cài đặt font biểu tượng bắt buộc (JetBrainsMono Nerd Font)

Bộ giao diện JaKooLit sử dụng các ký tự icon đặc biệt. Nếu thiếu font này, thanh Waybar và các menu sẽ bị lỗi ô vuông (tofu):

```bash
mkdir -p ~/.local/share/fonts
cd /tmp
curl -OL https://github.com/ryanoasis/nerd-fonts/releases/latest/download/JetBrainsMono.tar.xz
tar -xf JetBrainsMono.tar.xz -C ~/.local/share/fonts/
fc-cache -fv
```

### 3. Cài đặt tiện ích hình nền `swww`

```bash
# Tải binary swww dựng sẵn cho x86_64
cd /tmp
curl -OL https://github.com/LGFae/swww/releases/latest/download/swww-x86_64-unknown-linux-gnu.tar.gz
tar -xzf swww-x86_64-unknown-linux-gnu.tar.gz
sudo mv swww swww-daemon /usr/bin/
sudo chmod +x /usr/bin/swww /usr/bin/swww-daemon
```

---

## Giai Đoạn 2: Cài Đặt Nix & Thiết Lập Tầng Cô Lập Home-Manager

### 1. Cài đặt Nix Package Manager (Determinate Nix Multi-User)

```bash
sh <(curl -L https://nixos.org/nix/install) --daemon
. /etc/profile.d/nix.sh
nix --version
```

### 2. Cài đặt Home-Manager

```bash
nix-channel --add https://github.com/nix-community/home-manager/archive/master.tar.gz home-manager
nix-channel --update
nix-shell '<home-manager>' -A install
```

### 3. Tạo file cấu hình [`~/.config/home-manager/home.nix`](~/.config/home-manager/home.nix)

Tạo file khai báo toàn bộ công cụ người dùng kèm cầu nối driver đồ họa OpenGL/EGL (`pkgs.mesa`) đóng gói tự động:

```bash
mkdir -p ~/.config/home-manager
cat << 'EOF' > ~/.config/home-manager/home.nix
{ config, pkgs, ... }:

let
  kitty-wrapped = pkgs.symlinkJoin {
    name = "kitty-wrapped";
    paths = [ pkgs.kitty ];
    buildInputs = [ pkgs.makeWrapper ];
    postBuild = ''
      wrapProgram $out/bin/kitty \
        --prefix LD_LIBRARY_PATH : "${pkgs.mesa}/lib" \
        --set LIBGL_DRIVERS_PATH "${pkgs.mesa}/lib/dri" \
        --set GBM_BACKENDS_PATH "${pkgs.mesa}/lib/gbm"
    '';
  };

  quickshell-wrapped = pkgs.symlinkJoin {
    name = "quickshell-wrapped";
    paths = [ pkgs.quickshell ];
    buildInputs = [ pkgs.makeWrapper ];
    postBuild = ''
      wrapProgram $out/bin/quickshell \
        --prefix LD_LIBRARY_PATH : "${pkgs.mesa}/lib" \
        --set LIBGL_DRIVERS_PATH "${pkgs.mesa}/lib/dri" \
        --set GBM_BACKENDS_PATH "${pkgs.mesa}/lib/gbm"
      wrapProgram $out/bin/qs \
        --prefix LD_LIBRARY_PATH : "${pkgs.mesa}/lib" \
        --set LIBGL_DRIVERS_PATH "${pkgs.mesa}/lib/dri" \
        --set GBM_BACKENDS_PATH "${pkgs.mesa}/lib/gbm"
    '';
  };
in
{
  home.username = builtins.getEnv "USER";
  home.homeDirectory = builtins.getEnv "HOME";
  home.stateVersion = "24.05";

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
  ];

  programs.home-manager.enable = true;
}
EOF
```

### 4. Kích hoạt môi trường phần mềm cô lập

```bash
home-manager switch
```

---

## Giai Đoạn 3: Triển Khai Giao Diện JaKooLit Hyprland-Dots

### 1. Tải bộ cấu hình

```bash
git clone --depth=1 https://github.com/JaKooLit/Hyprland-Dots.git ~/Downloads/Hyprland-Dots
```

### 2. Triển khai vào `~/.config/`

```bash
mkdir -p ~/.config
cp -r ~/Downloads/Hyprland-Dots/config/hypr ~/.config/
cp -r ~/Downloads/Hyprland-Dots/config/waybar ~/.config/
cp -r ~/Downloads/Hyprland-Dots/config/swaync ~/.config/
cp -r ~/Downloads/Hyprland-Dots/config/rofi ~/.config/
cp -r ~/Downloads/Hyprland-Dots/config/kitty ~/.config/

chmod +x ~/.config/hypr/scripts/*.sh
chmod +x ~/.config/hypr/UserScripts/*.sh
```

---

## Giai Đoạn 4: Cầu Nối Ổn Định & Tính Năng Quickshell Overview

### 1. Đồng bộ mã nguồn giao diện QML Overview

```bash
mkdir -p ~/.config/quickshell/overview
cp -r ~/Downloads/Hyprland-Dots/config/quickshell/* ~/.config/quickshell/overview/ 2>/dev/null || true
```

> [!NOTE]
> Nhờ cơ chế `symlinkJoin` khai báo trực tiếp trong `home.nix` ở Giai đoạn 2, các nhị phân `kitty`, `quickshell`, và `qs` trong `~/.nix-profile/bin/` đã tự động tích hợp sẵn bridge driver GPU của Nix. Bạn **không cần** tạo bất kỳ script wrapper thủ công nào trong `~/.local/bin`.

### 3. Vá lỗi nhận diện tiến trình trong [`OverviewToggle.sh`](~/.config/hypr/scripts/OverviewToggle.sh)

Script gốc dùng `pgrep -x quickshell` sẽ bị fail do wrapper binary của Nix có tên `.quickshell-wra`. Ta thay thế bằng `pgrep -f`:

```bash
sed -i 's/pgrep -x quickshell/pgrep -f quickshell || pidof quickshell/g' ~/.config/hypr/scripts/OverviewToggle.sh
```

### 4. Cập nhật biến môi trường Hyprland

- **Tại [`~/.config/hypr/configs/ENVariables.conf`](~/.config/hypr/configs/ENVariables.conf):**
  Thêm dòng sau vào ngay đầu file:
  ```ini
  env = PATH,$HOME/.local/bin:$PATH
  ```
- **Tại [`~/.config/hypr/configs/Startup_Apps.conf`](~/.config/hypr/configs/Startup_Apps.conf):**
  Cập nhật dòng khởi chạy Quickshell:
  ```ini
  exec-once = env LD_LIBRARY_PATH=$HOME/.nix-profile/lib LIBGL_DRIVERS_PATH=$HOME/.nix-profile/lib/dri GBM_BACKENDS_PATH=$HOME/.nix-profile/lib/gbm qs -c overview
  ```

---

## Giai Đoạn 5: Cấu Hình Waybar Chỉ Hiển Thị Màn Hình Chính

Thêm chỉ định màn hình vào [`~/.config/waybar/UserModules`](~/.config/waybar/UserModules):

```json
{
  "output": "eDP-1"
}
```

_(Nếu muốn đặt màn hình ngoài Dell làm chính, đổi giá trị thành `"HDMI-A-1"`)._

Sau đó nạp lại giao diện:

```bash
~/.config/hypr/scripts/Refresh.sh
```

---

## Giai Đoạn 6: Đăng Nhập & Kiểm Thử

1. Đăng xuất khỏi Ubuntu (Log out).
2. Tại màn hình đăng nhập GDM, nhấp vào biểu tượng bánh răng ở góc dưới bên phải màn hình và chọn **Hyprland**.
3. Đăng nhập và kiểm tra:
   - Thanh Waybar hiển thị đúng trên màn hình chính.
   - Bấm **`Super + A`** hoặc **vuốt 3 ngón tay hướng lên** trên Touchpad -> Giao diện Overview hiển thị trơn tru.
