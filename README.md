# ccx — quản lí nhiều tài khoản Claude Code trên một máy Linux

Một script bash duy nhất, không phụ thuộc gì ngoài `bash`, `awk`, `python3` (chỉ để đọc JSON).

## Nguyên lí

Claude Code đọc toàn bộ cấu hình từ thư mục trỏ bởi biến `CLAUDE_CONFIG_DIR`.
`ccx` cho mỗi account một thư mục riêng dưới `~/.claude-accounts/<tên>/`, nên
credentials, lịch sử hội thoại, `settings.json` và MCP của từng account **tách biệt
hoàn toàn**:

- đăng nhập sẵn cả 3 account, không cần `/logout` → `/login` mỗi lần đổi;
- chạy song song nhiều terminal, mỗi terminal một account;
- đụng rate limit ở account này thì `ccx next` nhảy sang account khác.

Kèm theo là một sổ ghi `~/.claude-accounts/ledger.tsv`, mỗi dòng 3 cột cách nhau bằng TAB:

```
<tên account>	<epoch lần dùng cuối>	<epoch hết cooldown>
```

## Cài đặt

```bash
install -Dm755 ccx ~/.local/bin/ccx
```

Đảm bảo `~/.local/bin` nằm trong `PATH`, rồi thêm alias:

```bash
ccx shellinit >> ~/.bashrc && source ~/.bashrc
```

## Các lệnh

| Lệnh | Việc |
| --- | --- |
| `ccx add <tên>` | tạo thư mục account (chmod 700) rồi mở Claude Code để gõ `/login` |
| `ccx ls` | bảng TÊN / EMAIL / GÓI / DÙNG LẦN CUỐI / TRẠNG THÁI (mặc định khi chạy `ccx`) |
| `ccx use <tên> [args…]` | chạy Claude Code bằng account đó, truyền tiếp mọi tham số |
| `ccx next [args…]` | chọn account **không** trong cooldown và có lần dùng cuối cũ nhất |
| `ccx sh <tên>` | mở shell con đã export sẵn `CLAUDE_CONFIG_DIR` và `CCX_ACCOUNT` |
| `ccx env <tên>` | in các dòng `export` để `eval` |
| `ccx limit <tên> [giờ]` | đánh dấu đã chạm limit, mặc định 5 giờ |
| `ccx clear <tên>` | xoá cooldown |
| `ccx usage <tên>` | mở Claude Code kèm nhắc gõ `/usage` |
| `ccx login <tên>` | đăng nhập lại |
| `ccx token <tên>` | dán OAuth token dài hạn (`claude setup-token`), lưu chmod 600 |
| `ccx rm <tên>` | xoá account, phải gõ lại đúng tên để xác nhận |
| `ccx shellinit` | in đoạn alias để thêm vào `~/.bashrc` |
| `ccx --help` | in phần chú thích đầu file |

## Ví dụ

```bash
ccx add max1        # đăng nhập account thứ nhất
ccx add max2
ccx add max3

ccx ls
# TÊN   EMAIL                 GÓI  DÙNG LẦN CUỐI  TRẠNG THÁI
# max1  kang+max1@gmail.com   max  3h trước       sẵn sàng
# max2  kang+max2@gmail.com   max  30p trước      đã chạm limit, còn 4h12p
# max3  kang+max3@gmail.com   max  chưa dùng      sẵn sàng

ccx next                       # tự chọn max3
ccx use max1 --model opus      # chỉ định account, tham số truyền thẳng cho claude
CCX_DANGER=0 ccx use max1      # chạy KHÔNG kèm --dangerously-skip-permissions
ccx limit max1                 # vừa dính "5-hour limit reached" → đánh dấu 5 giờ
```

`ccx sh <tên>` mở một shell con đã có sẵn biến môi trường — tiện khi cần chạy nhiều
lệnh `claude` liên tiếp bằng cùng một account.

## Token dài hạn (tuỳ chọn)

Nếu muốn khỏi phụ thuộc phiên đăng nhập trình duyệt:

```bash
CLAUDE_CONFIG_DIR=~/.claude-accounts/max1 claude setup-token
ccx token max1      # dán token vào, lưu ở ~/.claude-accounts/max1/oauth_token (chmod 600)
```

Khi file `oauth_token` tồn tại, `use` / `next` / `sh` sẽ tự export `CLAUDE_CODE_OAUTH_TOKEN`.

## Biến môi trường

| Biến | Mặc định | Ý nghĩa |
| --- | --- | --- |
| `CCX_HOME` | `~/.claude-accounts` | thư mục gốc chứa account và `ledger.tsv` |
| `CCX_CLAUDE_BIN` | `claude` | lệnh Claude Code cần chạy |
| `CCX_DANGER` | `1` | luôn chạy `claude` với `--dangerously-skip-permissions`; đặt `0` để tắt |

## Lưu ý

- Script chỉ để quản lí các tài khoản **của chính bạn**. Việc dùng chung một tài khoản
  cho nhiều người vi phạm điều khoản dịch vụ của Anthropic.
- `ccx limit` là đánh dấu **thủ công** — Claude Code không có API để script tự biết đã
  chạm limit; cứ dính "5-hour limit reached" thì gõ `ccx limit <tên>`.
- Đã kiểm tra `bash -n` và `shellcheck -S warning` sạch.
