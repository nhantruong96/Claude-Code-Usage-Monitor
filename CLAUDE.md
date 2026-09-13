# CLAUDE.md - ghi chú làm việc cho repo này

Đây là fork riêng của `CodeZeno/Claude-Code-Usage-Monitor` (remote `origin` là
`nhantruong96/Claude-Code-Usage-Monitor`, remote `upstream` là repo gốc). File
này ghi các quyết định đã chốt trong lúc làm việc để phiên sau không phải hỏi lại.

## Quyết định đã chốt

### 13/09/2026 - Chỉ báo nhịp tiêu thụ (pace badge) trong theme mặc định

Người quyết: Nhan Truong.

Nội dung chốt:

1. Mốc chuẩn lấy theo **mức dự phóng cuối cửa sổ**, tính bằng
   `phần trăm đã dùng / phần trăm thời gian đã trôi * 100`, nghĩa là mức mà cửa
   sổ sẽ đạt tới nếu giữ nguyên nhịp trung bình từ đầu cửa sổ đến hiện tại.
   Ngưỡng cần theo dõi luôn là 100% ở mọi thời điểm trong cửa sổ.
2. Hiển thị dạng **mũi tên kèm số dự phóng**, ví dụ `↑112%`, với vùng chết ±10%
   (`↑` khi tỷ lệ >= 1.1, `↓` khi <= 0.9, `=` khi ở giữa).
3. Phạm vi: **chỉ sửa theme JSON**, không thêm binding mới trong Rust. Phần Rust
   duy nhất được sửa là kỳ vọng kích thước trong test và một test mới cho chính
   template của theme.
4. Chỉ làm cho Claude (cửa sổ session 5 giờ và weekly 7 ngày) trước; các provider
   còn lại chờ xác nhận sau khi xem thực tế trên taskbar.

Điều bị huỷ bỏ vì quyết định này: phương án đo **burn rate tức thời** (so sánh
lượng tiêu thụ giữa hai lần poll). Lý do là `DataContext` của theme engine hoàn
toàn không có trạng thái lịch sử - chỉ có giá trị của lần poll hiện tại cộng
`time.now.unix` - nên burn rate buộc phải sửa Rust để lưu mẫu vào cache, vượt
ngoài phạm vi đã chốt.

Độ dài cửa sổ bị hard-code trong biểu thức của theme (chia `reset.seconds` cho
`180` với cửa sổ 5 giờ và cho `6048` với cửa sổ 7 ngày). Muốn dùng chung cho
Codex, Cursor hay OpenCode thì phải mang độ dài cửa sổ vào `UsageSection` trong
`src/models.rs`, vì hiện không chỗ nào lưu giá trị này.

## Môi trường

Ngày 13/09/2026 đã dựng xong bộ công cụ build ngay trên máy này: Rust stable
1.98.1 (`rustc` và `cargo` nằm trong thư mục `.cargo/bin` của hồ sơ người dùng,
host MSVC), MSVC C++ toolset 14.51.36231 và Windows SDK 10.0.26100.0 được bổ
sung vào Visual Studio Community 2026. Muốn chạy `cargo test` thì thêm thư mục
`.cargo/bin` đó vào `PATH` của phiên PowerShell trước.

Hai cạm bẫy đã trả giá khi cài, đừng lặp lại:

1. Gói winget `Rustlang.Rustup` chỉ cài `rustup` chứ không kèm toolchain nào, và
   lần tải đầu bị đứt để lại toolchain thiếu manifest. Phải gỡ bằng
   `rustup toolchain uninstall stable-x86_64-pc-windows-msvc` rồi cài lại.
2. Gọi `setup.exe` của Visual Studio qua `Start-Process -Verb RunAs` thì tham số
   `--installPath` trỏ vào đường dẫn có khoảng trắng bị cắt ngay tại khoảng
   trắng (log ghi `installPath: C:\Program`, thoát với mã 1), còn cờ `--wait`
   thì bị từ chối với mã 87. Cách chạy được là dùng
   `--productId Microsoft.VisualStudio.Product.Community` kèm
   `--channelId VisualStudio.18.Release` và bỏ hẳn `--wait`.

CI của repo chỉ chạy `cargo build --release` khi push tag `v*`, không chạy
`cargo test`, nên test phải chạy tại máy.

Cảnh báo clippy `using chunks_exact with a constant chunk size` tại
`src/poller/claude.rs:726` và `:737` là có sẵn từ trước, không liên quan đến
thay đổi ở đây.
