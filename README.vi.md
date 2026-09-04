# G-Labs Voiceover

[English](README.md) | **Tiếng Việt**

Ứng dụng desktop biến kịch bản thành file giọng đọc hoàn chỉnh — **hai bộ máy TTS trong một chỗ**: giọng của CapCut và giọng đọc Microsoft Edge (**322 giọng neural, 142 ngôn ngữ/vùng**). Dán văn bản, chọn giọng, nhận một file MP3 đã ghép kèm phụ đề SRT khớp thời lượng. ffmpeg đóng gói sẵn bên trong; không cần cài gì thêm.

![Màn hình chính](docs/screenshots/01-main.png)

## Tính năng

- **Hai bộ máy, một quy trình**: chuyển qua lại **CapCut** và **Microsoft Edge** mà không đổi gì khác — cùng bảng đoạn, cùng tham số, cùng cách xuất. Edge miễn phí và không giới hạn, nên vừa là nguồn giọng chính vừa là lối thoát khi CapCut bị siết
- **Tách câu thông minh**: mỗi câu một đoạn, mỗi dòng một đoạn, hoặc **gộp thông minh** theo số ký tự — từng đoạn tự tạo, nghe lại, tạo lại và tải riêng
- **Nhập .txt / .srt**: nhập SRT giữ nguyên mốc thời gian từng câu, và **"khớp thời lượng phụ đề"** sẽ tăng tốc mỗi câu (chỉ tăng, không giảm, tối đa 1.8×) để rơi đúng mốc gốc — audio ghép ra khớp timeline phụ đề
- **Chế độ Hội thoại**: viết `<Tên> lời thoại` mỗi dòng, gán một giọng cho mỗi nhân vật, cả đoạn hội thoại tạo trong một lượt với giọng riêng từng câu; xuất SRT giữ nguyên tên nhân vật
- **Đổi tốc độ & khoảng lặng không cần tạo lại**: hai thanh trượt tốc độ và khoảng nghỉ chỉ ghép lại tại máy — không gọi API, không tốn lượt
- **Xuất file**: MP3 đã ghép, SRT, hoặc cả gói ZIP. Tên file có số thứ tự + vài chữ đầu + thời gian, nên không bao giờ ghi đè bản cũ
- **Nghe thử giọng**: mỗi giọng có mẫu sẵn để nghe trước khi tốn lượt — giọng đa ngôn ngữ có mẫu từng thứ tiếng, giọng bản địa có mẫu tiếng của mình
- **Kho proxy**: dán proxy đủ định dạng (`host:port:user:pass`, `user:pass@host:port`, `socks5://…`, IPv6) vào danh sách lưu; mỗi lượt gọi CapCut/Edge xoay vòng qua danh sách. **Kiểm tra lại** dò từng proxy, tự nhận diện loại (HTTP/SOCKS4/SOCKS5) và đánh dấu cái hỏng — xoá một nhát. Mật khẩu được che trong danh sách
- **Webhook API** (tuỳ chọn bật): một máy chủ HTTP cục bộ để công cụ ngoài hoặc AI agent gọi — nộp một đoạn text hoặc cả kịch bản nhiều đoạn, chờ task, rồi tải file MP3 đã ghép (kèm file từng đoạn và SRT nếu cần). Có khoá API riêng tự sinh, chỉ chạy khi tài khoản đủ quyền, mặc định bind `127.0.0.1`. Tài liệu đầy đủ: [WEBHOOK.md](WEBHOOK.md)
- **Ghi nhớ mọi thứ**: kịch bản, bảng đoạn, giọng đã chọn, tốc độ, khoảng lặng, giọng yêu thích, mức zoom, ngôn ngữ và tab đang mở đều còn nguyên sau khi tắt mở lại — lưu ở thư mục dữ liệu của hệ điều hành, không phải localStorage
- **Zoom toàn giao diện** (70–140%) ngay trên titlebar — bố cục co giãn thật, thu nhỏ để hiện được nhiều hơn chứ không chỉ làm chữ bé đi
- **Lịch sử tạo**: mỗi lượt tự lưu, tìm được theo nội dung, nạp lại đúng tab đã tạo
- **11 ngôn ngữ**: Tiếng Việt, English, हिन्दी, Türkçe, Português, 简体中文, اردو (RTL), বাংলা, Русский, Español, ไทย
- **Tự cập nhật** từ GitHub Releases, ngay trong app

## Ảnh chụp

| Hội thoại — mỗi nhân vật một giọng | Kho giọng (322 giọng Edge) |
|---|---|
| ![Hội thoại](docs/screenshots/02-dialogue.png) | ![Giọng](docs/screenshots/03-voices.png) |

| Kho proxy (tự nhận loại, che mật khẩu) | Lịch sử tạo |
|---|---|
| ![Proxy](docs/screenshots/04-proxy.png) | ![Lịch sử](docs/screenshots/05-history.png) |

| Giao diện phải-sang-trái (اردو) | Webhook API (tự động hoá) |
|---|---|
| ![RTL](docs/screenshots/06-rtl-urdu.png) | ![Webhook API](docs/screenshots/07-webhook.png) |

## Webhook API — tự động hoá

Bật tab **Webhook API**, Voiceover chạy một máy chủ HTTP cục bộ nhỏ để công cụ của
bạn (hoặc một AI agent) gọi:

```bash
curl -X POST http://127.0.0.1:8788/api/tts \
  -H "X-API-Key: <khoá lấy trong panel>" -H "Content-Type: application/json" \
  -d '{"provider":"edge","text":"Xin chào","voice":"vi-VN-HoaiMyNeural","srt":true}'
# → { "task_id": "…" }  → chờ GET /api/status/{id}  → tải ở /api/files
```

Nộp `text` hoặc mảng `segments`, nhận một file `master.mp3` đã ghép (kèm file từng
đoạn và SRT nếu yêu cầu). Xác thực bằng khoá API riêng mỗi máy; chỉ sinh khi tài
khoản đủ quyền; máy chủ bind `127.0.0.1` trừ khi bạn mở ra mạng LAN. Tài liệu đầy
đủ — mọi endpoint, trường body và cấu trúc phản hồi — nằm ở **[WEBHOOK.md](WEBHOOK.md)**,
viết để một AI agent đọc là tích hợp được ngay.

## Nên dùng bộ máy nào?

| | CapCut | Microsoft Edge |
|---|---|---|
| Số giọng | 121 | **322**, 142 ngôn ngữ/vùng |
| Giới hạn | có hạn mức mỗi phiên | không |
| Hợp cho | chất giọng đặc trưng CapCut | làm số lượng lớn, ngôn ngữ hiếm |

Một số giọng CapCut là **một model đa ngôn ngữ** chứ không phải người bản xứ — chúng có nhãn 🌐, vì khi đọc ngôn ngữ không phải tiếng gốc sẽ hơi lơ lớ. Giọng bản địa nghe tự nhiên; nhãn này để bạn chọn cho đúng ý.

## Cài đặt

Tải bản mới nhất ở [Releases](https://github.com/duckmartians/G-Labs-Voiceover/releases):

- **Windows**: `GLabsVoiceover-<phiên-bản>-setup.exe`
- **macOS (Apple Silicon)**: `GLabsVoiceover-<phiên-bản>-arm64.dmg` — chưa ký; lần đầu mở: chuột phải → Open, hoặc chạy `xattr -dr com.apple.quarantine "/Applications/G-Labs Voiceover.app"`

Thiết lập lưu ở thư mục dữ liệu người dùng của hệ điều hành (`%APPDATA%\G-Labs Voiceover` trên Windows, `~/Library/Application Support/G-Labs Voiceover` trên macOS).

## Yêu cầu tài khoản

Cần **tài khoản G-Labs đã trả phí bất kỳ công cụ nào** (gói Lite trở lên), hoặc có add-on Voice còn hạn. Đăng nhập bằng tài khoản Google đã liên kết với G-Labs — app mở trình duyệt hệ thống. Xem [các gói & công cụ](https://duckmartians.info).

Cấu hình endpoint giọng do máy chủ bản quyền giao theo từng phiên và chỉ giữ trong bộ nhớ; không có gì về chúng nằm sẵn trong app.

## Build từ mã nguồn

```bash
npm run install:all      # root + backend + frontend
npm run dev              # frontend :5180, backend :3011
npm run dev:app          # chạy vỏ Electron

npm run build:mac        # → release/GLabsVoiceover-<phiên-bản>-arm64.dmg
```

Windows: chạy `setup.bat` một lần, rồi `build-exe.bat`.
