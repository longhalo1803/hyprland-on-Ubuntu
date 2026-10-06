# Báo Cáo 06: Môi Trường TUI, Neovim (LazyVim) & Tùy Biến Trải Nghiệm Desktop (TUI & Advanced Customizations)

---

## 1. Tổng Quan Môi Trường TUI (Terminal User Interface)

Hệ sinh thái terminal được tối ưu hóa đồng bộ với phong cách thiết kế kính mờ (Islands-Glass) và bảng màu của **Kitty Terminal**, tạo nên môi trường làm việc tập trung cao độ, giàu thẩm mỹ và tiêu tốn tối thiểu tài nguyên phần cứng.

### Danh mục công cụ TUI cốt lõi:

| Công Cụ         | Phiên Bản          | Vai Trò & Điểm Nổi Bật                                                                | Cấu Hình Kích Hoạt           |
| :-------------- | :----------------- | :------------------------------------------------------------------------------------ | :--------------------------- |
| **`kitty`**     | v0.48.2+           | Terminal chính tăng tốc bằng GPU qua Nix Mesa wrapper; hỗ trợ độ mờ (opacity/blur)    | `~/.config/kitty/kitty.conf` |
| **`neovim`**    | v0.11.x+ (Nightly) | Code editor chính thay thế hoàn toàn VS Code; sử dụng khung LazyVim Distro            | `~/.config/nvim/`            |
| **`cava`**      | v1.0.0             | Trình hiển thị sóng âm thanh (Audio visualizer); tự động đồng bộ màu theo theme Kitty | `cava`                       |
| **`tty-clock`** | v2.3+              | Đồng hồ số digital tối giản; hỗ trợ căn giữa, chế độ screensaver và đổi màu           | `tty-clock -c -C 7 -s -b`    |
| **`btop`**      | v1.4.0+            | Giám sát tài nguyên CPU, RAM, GPU, mạng và tiến trình thời gian thực                  | `btop`                       |
| **`fastfetch`** | v2.68.0+           | Hiển thị thông tin phần cứng, nhân Linux và distro với tốc độ tức thì                 | `fastfetch`                  |
| **`cmatrix`**   | v2.0+              | Hiệu ứng mưa mã nguồn Matrix cổ điển phục vụ trình diễn và màn hình chờ               | `cmatrix -b -u 2`            |

---

## 2. Thiết Lập Neovim Chuẩn Cộng Đồng (LazyVim Full Distro)

### 2.1. Yêu cầu phiên bản Neovim (Neovim >= 0.11.2)

- **Vấn đề với bản APT:** Ubuntu 24.04 mặc định chỉ cung cấp Neovim `v0.9.5`, trong khi hệ sinh thái plugin hiện đại (LSP, Treesitter rewrite, snacks.nvim, blink.cmp) đã dừng hỗ trợ (deprecated) Neovim $\le 0.10$.
- **Giải pháp:** Cài đặt bản dựng nhị phân chính thức từ GitHub Releases (kênh Nightly / 0.11+):

```bash
curl -LO https://github.com/neovim/neovim/releases/download/nightly/nvim-linux-x86_64.tar.gz
sudo rm -rf /opt/nvim-linux-x86_64
sudo tar -C /opt -xzf nvim-linux-x86_64.tar.gz
sudo ln -sf /opt/nvim-linux-x86_64/bin/nvim /usr/local/bin/nvim
rm nvim-linux-x86_64.tar.gz
```

### 2.2. Khởi tạo LazyVim Starter

```bash
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
```

### 2.3. Cấu hình Nền Trong Suốt (Transparent Theme)

Để Neovim hòa quyện hoàn toàn với màu nền và hiệu ứng blur của Kitty Terminal, tạo file `~/.config/nvim/lua/plugins/theme.lua`:

```lua
return {
  {
    "folke/tokyonight.nvim",
    opts = {
      transparent = true,
      styles = {
        sidebars = "transparent",
        floats = "transparent",
      },
    },
  },
}
```

### 2.4. Cấu hình Định Dạng Mã Nguồn (Prettier Formatter)

Kích hoạt gói mở rộng chính thức của LazyVim tại `~/.config/nvim/lua/plugins/formatting.lua`:

```lua
return {
  { import = "lazyvim.plugins.extras.formatting.prettier" },
}
```

_Tự động định dạng: JavaScript, TypeScript, JSX, TSX, HTML, CSS, JSON, YAML, Markdown thông qua `conform.nvim` và gói nhị phân `prettier` do Mason quản lý._

### 2.5. Tự Động Lưu Mã Nguồn (Auto-Save 500ms Debounce)

Thêm bộ bắt sự kiện tự động lưu sau 500ms dừng gõ tại `~/.config/nvim/lua/config/autocmds.lua`:

```lua
-- Auto-save sau 500ms không có thao tác gõ (debounce)
local auto_save_timer = nil
vim.api.nvim_create_autocmd({ "TextChanged", "InsertLeave" }, {
  group = vim.api.nvim_create_augroup("AutoSaveDebounce", { clear = true }),
  pattern = "*",
  callback = function()
    if auto_save_timer then
      auto_save_timer:stop()
    else
      auto_save_timer = vim.uv.new_timer()
    end

    if auto_save_timer then
      auto_save_timer:start(
        500,
        0,
        vim.schedule_wrap(function()
          if vim.bo.modified and vim.bo.buftype == "" and vim.fn.expand("%") ~= "" then
            vim.cmd("silent! update")
          end
        end)
      )
    end
  end,
})
```

_Cơ chế:_ Sử dụng `vim.uv.new_timer()` native với nil-check an toàn cho Language Server, áp dụng trên `TextChanged` và `InsertLeave` nhằm triệt tiêu hiện tượng giật con trỏ khi đang soạn thảo.

### 2.6. Khóa Phiên Bản Bền Vững (Zero-breaking Stability)

Toàn bộ 32 plugin cốt lõi được đóng băng chính xác tại commit hash trong `~/.config/nvim/lazy-lock.json`. Hệ thống hoàn toàn không tự ý cập nhật ngầm, đảm bảo khả năng hoạt động ổn định lâu dài qua nhiều năm.

---

## 3. Tùy Biến Thanh Trạng Thái (Waybar Islands-Glass)

Hệ thống sử dụng bố cục Waybar tối ưu hóa cho màn hình laptop:

- **Bố cục kích hoạt:** `[TOP] Default Laptop-glass` (trỏ qua symlink `~/.config/waybar/config`).
- **Giao diện kính mờ:** `[Kitty] Islands-Glass.css` (trỏ qua symlink `~/.config/waybar/style.css`).
- **Cấu hình chống tràn đa màn hình:** Khóa thanh hiển thị trên màn hình laptop tích hợp trong `~/.config/waybar/UserModules`:
  ```json
  {
    "output": "eDP-1"
  }
  ```
- **Tùy biến nhóm module:** `ModulesCustom` và `ModulesGroups` điều khiển các nút bật/tắt nhanh, chỉ báo pin, tải phần cứng và mạng.

---

## 4. Bộ Chọn Hình Nền Quickshell Parallelogram 2D Flow (`Super + W`)

Thay thế các công cụ chọn hình nền truyền thống bằng giao diện thẻ bài hình bình hành vát góc độc lập, mượt mà viết bằng Qt6/QML:

- **Thư mục mã nguồn:** `~/.config/quickshell/wallpaper-flow/`
  - `shell.qml`: Định nghĩa `PanelWindow` trên Layer Shell cấp độ `Overlay`, hiển thị danh sách hình nền dạng thẻ bài không chồng lấn.
  - `backend.sh`: Quét thư mục ảnh, tạo thumbnail cache và xuất danh sách JSON tốc độ cao.
- **Kịch bản kích hoạt:** `~/.config/hypr/UserScripts/WallpaperFlowToggle.sh`
- **Tích hợp phím tắt:** Gán vào `Super + W` trong `UserKeybinds.conf`.

---

## 5. Bảng Phím Tắt Tùy Biến Nâng Cao (`UserKeybinds.conf`)

Tất cả các phím tắt do người dùng ghi đè được quản lý tập trung tại `~/.config/hypr/UserConfigs/UserKeybinds.conf` theo nguyên tắc `unbind` phím mặc định trước khi `bindd` phím mới:

| Phím Tắt                                 | Lệnh Kích Hoạt                    | Chức Năng Chi Tiết                                                                     |
| :--------------------------------------- | :-------------------------------- | :------------------------------------------------------------------------------------- |
| **`Super + D`**                          | `pkill rofi \|\| rofi -show drun` | Mở menu tìm kiếm và khởi chạy ứng dụng Rofi                                            |
| **`Super + M`**                          | `InfiniteCanvas.py enter`         | Kích hoạt chế độ **Infinite Canvas Submap** (điều hướng Tab / Shift+Tab / Enter / Esc) |
| **`Super + W`**                          | `WallpaperFlowToggle.sh`          | Mở/đóng bộ chọn hình nền Quickshell Parallelogram 2D Flow                              |
| **`Super + B`**                          | `pkill -SIGUSR1 waybar`           | Ẩn / hiện thanh trạng thái Waybar tức thì                                              |
| **`Super + F`**                          | `fullscreen, 1`                   | Phóng to cửa sổ toàn màn hình nhưng **vẫn giữ thanh Waybar**                           |
| **`Super + I`**                          | `gnome-control-center`            | Mở nhanh trung tâm cài đặt hệ thống GNOME Settings                                     |
| **`Super + Shift + S`** hoặc **`Print`** | `grim -g "$(slurp)" ... && ksnip` | Chụp màn hình vùng chọn và mở ngay trong trình chỉnh sửa Ksnip                         |
| **`Super + Print`**                      | `grim ... && ksnip`               | Chụp toàn bộ màn hình và mở trong Ksnip                                                |
| **`Super + A`**                          | `OverviewToggle.sh`               | Mở giao diện tổng quan cửa sổ (Window Overview)                                        |
