# Telegram VoIP Notifier cho ZimaOS

Gói Docker Compose để chạy **Telegram VoIP Notifier** trên ZimaOS, dùng image
public `ghcr.io/trankhanhduy2929-beep/telegram_voip:0.1.10` (amd64/arm64).

Repo này chỉ chứa cấu hình triển khai, không chứa source. Source gốc vẫn nằm ở
repo private; image được build và publish từ pipeline riêng.

## Yêu cầu

- ZimaOS trên `amd64` hoặc `arm64`.
- Có Internet để pull image và dùng Edge TTS.
- `api_id` / `api_hash` lấy tại <https://my.telegram.org>.
- Nên dùng trong LAN hoặc VPN, không mở cổng `3332` trực tiếp ra Internet.

## Cài trên ZimaOS

1. Mở ZimaOS Dashboard, bấm nút **+** góc trên bên phải.
2. Chọn **Install a customized app**.
3. Bấm **Import**, chọn tab **Docker Compose**, dán nội dung
   [`docker-compose.yml`](docker-compose.yml) hoặc tải file lên.
4. Giữ nguyên:
   - cổng `3332`
   - volume `/DATA/AppData/telegram-voip` → `/data`
5. Bấm **Install**, đợi container chạy.
6. Mở app **Telegram VoIP Notifier** từ Dashboard.

Nếu ZimaOS hỏi thư mục dữ liệu, tạo trước bằng Terminal:

```sh
mkdir -p /DATA/AppData/telegram-voip
```

## Lấy API token

Lần đầu chạy, container tự sinh token tại `/data/api_token`. Lấy bằng Terminal
ZimaOS:

```sh
docker exec telegram-voip cat /data/api_token
```

Hoặc mở thư mục `/DATA/AppData/telegram-voip/` trong Files và đọc file
`api_token`. Dán token vào màn hình đăng nhập của app.

Có thể đặt token cố định bằng biến `TGVOIP_API_TOKEN` trước khi cài/khởi động
lại container.

## Cấu hình Telegram

1. Mở app, nhập API token.
2. Vào **Cài đặt → Thông tin Telegram**, nhập `api_id`, `api_hash`, bấm **Lưu**.
3. Thêm tài khoản Telegram, nhập số điện thoại, OTP và mật khẩu 2FA nếu có.
4. Giọng TTS mặc định là **Hoài My** (`vi-VN-HoaiMyNeural`). Có thể đổi trong
   Cài đặt; `vi-VN-NamMinhNeural` chỉ là giọng nam thay thế.

## Biến môi trường

Tất cả đều tùy chọn:

| Biến | Mặc định | Ý nghĩa |
| --- | --- | --- |
| `TZ` | `Asia/Ho_Chi_Minh` | Múi giờ log |
| `TGVOIP_TTS_VOICE` | `vi-VN-HoaiMyNeural` | Giọng Edge TTS |
| `TGVOIP_TTS_RATE` | `+0%` | Tốc độ đọc |
| `TGVOIP_API_ID` | trống | Telegram API ID |
| `TGVOIP_API_HASH` | trống | Telegram API hash |
| `TGVOIP_API_TOKEN` | tự sinh | Token cho UI/REST API |

Có thể đặt trước khi cài bằng `.env` (xem [`.env.example`](.env.example)) hoặc
sửa Environment trong ZimaOS sau khi cài.

## Dùng với Home Assistant

Nếu Home Assistant chạy nơi khác, cài custom integration
`telegramvoip_homeassistant` và cấu hình:

- URL: `http://<IP-ZimaOS>:3332`
- API token: token ở trên

Nếu Home Assistant chạy cùng Docker network với container này, dùng
`http://telegram-voip:3332`.

## Dữ liệu và backup

Volume `/DATA/AppData/telegram-voip` chứa:

- `accounts.json`
- `sessions/` — phiên đăng nhập Telegram
- `api_token`
- `cache/tts/`
- `.env` do UI cập nhật

Backup toàn bộ thư mục này. Không chia sẻ hoặc commit thư mục `sessions/` vì
chứa quyền truy cập tài khoản Telegram.

## Nâng cấp

Đổi tag image trong `docker-compose.yml` sang bản mới, sau đó recreate app
trong ZimaOS (không xóa volume `/DATA/AppData/telegram-voip`).

Ví dụ bản mới `0.2.0`:

```yaml
image: ghcr.io/trankhanhduy2929-beep/telegram_voip:0.2.0
```

## Chạy bằng Docker Compose CLI

```sh
mkdir -p /DATA/AppData/telegram-voip
docker compose up -d
docker logs -f telegram-voip
```

Dừng:

```sh
docker compose down
```

## Ghi chú

- Đây là container độc lập, không phải Home Assistant add-on; không có ingress
  tự đăng nhập. UI sẽ yêu cầu API token.
- Gọi Telegram P2P cần outbound UDP; nếu mạng/NAT chặn UDP, phần gọi có thể
  không có tiếng. Tin nhắn và TTS qua TCP vẫn hoạt động.
- Edge TTS có thể tạm lỗi từ phía Microsoft; app thử lại 5 lần và báo lỗi thay
  vì phát giọng `espeak-ng` sai.
- Image public chứa code runtime Python/JS. Repo public này không chứa source,
  nhưng không nên coi image public là nơi giấu code.
