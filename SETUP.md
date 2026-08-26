# Hướng dẫn cài đặt

## 1. Tạo repository profile

Tạo một repo public **có tên trùng đúng với username GitHub của bạn**.

Ví dụ: username là `phanthanhdat` → tên repo phải là `phanthanhdat`.

Khi tạo, tick chọn **Add a README file**. Đây là cơ chế "magic repo" của GitHub:
nội dung `README.md` trong repo này sẽ hiện lên trang profile của bạn.

## 2. Tạo Personal Access Token

`lowlighter/metrics` cần token riêng, **không dùng được `GITHUB_TOKEN` mặc định**
vì nó chỉ có quyền trong phạm vi repo hiện tại.

1. Vào **Settings → Developer settings → Personal access tokens → Tokens (classic)**
2. Bấm **Generate new token (classic)**
3. Đặt tên: `metrics`
4. Chọn scope:
   - `public_repo` — bắt buộc
   - `read:user` — bắt buộc
   - `read:org` — nếu muốn hiện dữ liệu organization
   - `repo` — chỉ chọn nếu muốn tính cả private repo vào thống kê
5. Copy token (chỉ hiện 1 lần)

## 3. Lưu token vào repo secret

Trong repo profile: **Settings → Secrets and variables → Actions → New repository secret**

- Name: `METRICS_TOKEN`
- Secret: dán token vừa copy

Tên phải chính xác là `METRICS_TOKEN` vì workflow đang gọi
`${{ secrets.METRICS_TOKEN }}`.

## 4. Thay placeholder

Tìm và thay toàn bộ trong cả 2 file:

| Placeholder | Thay bằng |
|---|---|
| `YOUR_USERNAME` | username GitHub của bạn (có 5 chỗ trong `metrics.yml`, 1 chỗ trong `README.md`) |
| `YOUR_LINKEDIN` | slug LinkedIn của bạn |

## 5. Push và chạy

```bash
git add .
git commit -m "Add profile README and metrics workflow"
git push origin main
```

Sau đó vào tab **Actions** → chọn workflow **Metrics** → bấm **Run workflow**
để chạy lần đầu ngay, không cần đợi đến 07:00 hôm sau.

## 6. Kiểm tra

Workflow sẽ commit các file `.svg` trực tiếp vào nhánh `main`. Sau lần chạy đầu
thành công, bạn sẽ thấy trong repo:

- `github-metrics.svg`
- `metrics.plugin.languages.indepth.svg`
- `metrics.plugin.isocalendar.fullyear.svg`
- `metrics.plugin.habits.charts.svg`
- `metrics.plugin.achievements.compact.svg`

Ảnh trong README sẽ **hiện lỗi (broken) cho tới khi workflow chạy xong lần đầu** —
đây là bình thường, không phải lỗi cấu hình.

## Xử lý sự cố thường gặp

**Workflow fail với lỗi 401 / 403**
Token sai, hết hạn, hoặc thiếu scope. Tạo lại token và cập nhật secret.

**Workflow fail với "Resource not accessible by integration"**
Thiếu `permissions: contents: write`. File workflow đã có sẵn dòng này — kiểm tra
xem có bị xoá không. Cũng kiểm tra **Settings → Actions → General → Workflow
permissions** đã chọn *Read and write permissions*.

**Ảnh vẫn broken sau khi workflow xanh**
Kiểm tra tên file trong README có khớp chính xác với `filename:` trong workflow không.
Phân biệt chữ hoa/thường.

**Plugin `languages_indepth` chạy rất lâu hoặc timeout**
Nó phải clone repo về để phân tích. Tăng `plugin_languages_analysis_timeout` lên
20–25, hoặc bỏ `plugin_languages_indepth: yes` nếu không cần.

## Ghi chú

Workflow đang chạy 5 job `lowlighter/metrics` riêng biệt. Mỗi job đều gọi GitHub API,
nên nếu bạn giảm cron xuống chạy quá thường xuyên (ví dụ mỗi giờ) có thể bị rate limit.
Chạy 1 lần/ngày là hợp lý.

Danh sách plugin đầy đủ và các tuỳ chọn khác xem tại:
https://github.com/lowlighter/metrics#-plugins
