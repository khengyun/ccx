# ccx — khi bạn quá giàu và có quá nhiều tài khoản Claude Code

> Một account Claude Max không đủ? Hai cũng chưa đã? Bạn có ba, bốn, năm cái và
> đang phát mệt vì `/logout` → `/login` mỗi lần dính "5-hour limit reached"?
> Chúc mừng, bạn là *target audience* của tool này. Chia buồn với ví tiền của bạn.

`ccx` là **một file bash duy nhất**, nhẹ hơn cả cảm giác tội lỗi khi trả tiền cho
account thứ ba. Không cài gì thêm ngoài `bash`, `awk` và `python3` (chỉ để đọc JSON,
hứa đấy). Đủ dùng, không màu mè, không framework, không "AI-powered account manager".

## Nó làm gì

Claude Code đọc toàn bộ cấu hình từ thư mục trỏ bởi biến `CLAUDE_CONFIG_DIR`.
`ccx` cho mỗi account một "căn hộ riêng" dưới `~/.claude-accounts/<tên>/`, nên
credentials, lịch sử hội thoại, `settings.json` và MCP của từng account **tách biệt
hoàn toàn**, không ai giẫm chân ai:

- đăng nhập sẵn cả đàn account một lần, từ đó không cần `/logout` → `/login` nữa;
- mở song song nhiều terminal, mỗi cửa sổ một account, cày như một farm;
- account này hết hơi thì `ccx next` tự nhảy sang account rảnh lâu nhất — luân phiên
  công bằng như chia ca trực;
- mặc định luôn chạy `claude --dangerously-skip-permissions`, vì đã giàu thì việc gì
  phải bấm "Yes" từng lần (tắt được, xem bên dưới).

Kèm theo là một cuốn "sổ chấm công" `~/.claude-accounts/ledger.tsv`, mỗi dòng 3 cột
cách nhau bằng TAB:

```
<tên account>	<epoch lần dùng cuối>	<epoch hết cooldown>
```

## Cài đặt (30 giây, nhanh hơn thời gian bạn nạp thẻ)

```bash
install -Dm755 ccx ~/.local/bin/ccx
```

Đảm bảo `~/.local/bin` nằm trong `PATH`, rồi thêm alias cho đỡ mỏi tay:

```bash
ccx shellinit >> ~/.bashrc && source ~/.bashrc
```

## Các lệnh

| Lệnh | Việc |
| --- | --- |
| `ccx add <tên>` | tạo "căn hộ" cho account (chmod 700) rồi mở Claude Code để gõ `/login` |
| `ccx ls` | điểm danh: TÊN / EMAIL / GÓI / DÙNG LẦN CUỐI / TRẠNG THÁI (mặc định khi chạy `ccx` trơn) |
| `ccx use <tên> [args…]` | chạy Claude Code bằng account đó, tham số truyền thẳng cho `claude` |
| `ccx next [args…]` | tự chọn account **không** trong cooldown và nghỉ lâu nhất — "ai rảnh thì làm" |
| `ccx sh <tên>` | mở shell con đã export sẵn `CLAUDE_CONFIG_DIR` và `CCX_ACCOUNT` |
| `ccx env <tên>` | in các dòng `export` để `eval` |
| `ccx limit <tên> [giờ]` | đánh dấu "em này hết hơi rồi", mặc định nghỉ 5 giờ |
| `ccx clear <tên>` | xoá cooldown (khi bạn biết chắc nó đã hồi) |
| `ccx usage <tên>` | mở Claude Code kèm nhắc gõ `/usage` để xem còn bao nhiêu máu |
| `ccx login <tên>` | đăng nhập lại |
| `ccx token <tên>` | dán OAuth token dài hạn (`claude setup-token`), lưu chmod 600 |
| `ccx rm <tên>` | đuổi account đi, phải gõ lại đúng tên để xác nhận (không có undo, nghiêm túc đấy) |
| `ccx shellinit` | in đoạn alias để thêm vào `~/.bashrc` |
| `ccx --help` | in phần chú thích đầu file |

## Một ngày điển hình của người giàu

```bash
ccx add max1        # account thứ nhất, đăng nhập bằng /login
ccx add max2        # account thứ hai, "để dự phòng"
ccx add max3        # account thứ ba, thôi khỏi giải thích

ccx ls
# TÊN   EMAIL                 GÓI  DÙNG LẦN CUỐI  TRẠNG THÁI
# max1  kang+max1@gmail.com   max  3h trước       sẵn sàng
# max2  kang+max2@gmail.com   max  30p trước      đã chạm limit, còn 4h12p
# max3  kang+max3@gmail.com   max  chưa dùng      sẵn sàng

ccx next                       # tự chọn max3 (nghỉ lâu nhất)
ccx use max1 --model opus      # chỉ định account, tham số truyền thẳng cho claude
CCX_DANGER=0 ccx use max1      # hôm nay muốn sống thận trọng: KHÔNG kèm --dangerously-skip-permissions
ccx limit max1                 # vừa dính "5-hour limit reached" → cho nghỉ 5 giờ, qua max2/max3
```

`ccx sh <tên>` mở một shell con đã có sẵn biến môi trường — tiện khi cần chạy nhiều
lệnh `claude` liên tiếp bằng cùng một account. (Trong shell đó gõ `claude` thì tự thêm
cờ nhé, `ccx` không đọc được suy nghĩ của bạn.)

## Token dài hạn (tuỳ chọn, cho người lười mở trình duyệt)

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
| `CCX_DANGER` | `1` | luôn chạy `claude` với `--dangerously-skip-permissions`; đặt `0` nếu muốn Claude hỏi trước khi làm gì |

## Lưu ý (phần nghiêm túc duy nhất)

- Script chỉ để quản lí các tài khoản **của chính bạn**. Dùng chung một tài khoản cho
  nhiều người là vi phạm điều khoản dịch vụ của Anthropic — giàu thì giàu, đừng lách.
- `--dangerously-skip-permissions` đúng nghĩa đen là *dangerously*: Claude sẽ chạy lệnh,
  sửa file, xoá file mà không hỏi. Chạy trong thư mục nào thì tự chịu trách nhiệm với
  thư mục đó. Sợ thì `CCX_DANGER=0`.
- `ccx limit` là đánh dấu **thủ công** — Claude Code không có API để script tự biết đã
  chạm limit; cứ thấy "5-hour limit reached" thì gõ `ccx limit <tên>`, coi như bấm thẻ.
- Đã kiểm tra `bash -n` và `shellcheck -S warning` sạch. Nhẹ và đủ dùng, thế thôi.
