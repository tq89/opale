# Bài học & quyết định kỹ thuật

Tài liệu ghi nhận các quyết định kỹ thuật và bài học của dự án theme Opale.
Rà soát tài liệu này trước khi bắt đầu một phiên làm việc mới.

## Baseline đã kiểm chứng

Môi trường và trạng thái gốc của repo, đã xác minh bằng lệnh thực tế:

| Hạng mục | Lệnh | Kết quả |
|---|---|---|
| Cài dependencies | `npm ci` | exit 0 |
| Build | `npm run build` | Done, 8 stylesheet, không warning |
| Lint | `npm run lint` (stylelint) | Pass, 0 lỗi |
| Toolchain | — | Node v22.22.2, Dart Sass 1.101.0 |

**Build có tính tái lập (reproducible)**: sau `npm run build` trên commit gốc,
`git status` sạch hoàn toàn. CSS đã commit khớp 100% với kết quả build.

Hệ quả: mọi khác biệt xuất hiện trong `stylesheets/` hoặc `plugins/` sau khi
build đều là hệ quả trực tiếp của thay đổi SCSS, không phải nhiễu do khác
phiên bản trình biên dịch. Đây là cơ chế kiểm chứng chính của dự án.

## Kiến trúc theme

- Nguồn: `src/sass/`, đầu ra: `stylesheets/application.css` và `plugins/**/*.css`.
- Build bằng Grunt: `copy` (Tabler icons + webfonts) → `sass` (style compressed)
  → `postcss` (autoprefixer). Xem `Gruntfile.js`.
- **Cả nguồn lẫn đầu ra đều được commit.** Redmine nạp trực tiếp file CSS đã
  build, nên mỗi thay đổi SCSS phải kèm kết quả build tương ứng trong cùng commit.
- Toàn bộ theming là **Sass compile-time**: biến trong `src/sass/_variables.scss`
  với cờ `!default` để người dùng ghi đè. Theme **không** dùng CSS custom
  properties (`:root`), không có `prefers-color-scheme` hay `data-theme`.
- `src/sass/components/` chia theo thành phần giao diện Redmine
  (issue, top, lists, forms, login, wiki, gantt, calendar, responsive, print...).

## Tương thích Redmine 7.0.0

Đối chiếu các thay đổi CSS của Redmine 7.0.0 với mã nguồn theme:

| Thay đổi trong Redmine 7.0.0 | Ảnh hưởng tới Opale |
|---|---|
| Gỡ các class `icon-*` khỏi core, chuyển sang `legacy-icons-compat.css` | **Không rủi ro.** Opale tự định nghĩa `icon-*` qua `$icon-map` + Tabler webfont trong `components/_icons.scss`, không phụ thuộc core. |
| Header mới, gom link người dùng vào user menu dropdown | Đã xử lý (commit `ea0619c` "Fix new user menu"). |
| Thay physical properties bằng logical properties, gỡ `rtl.css` | **Cần lưu ý.** Theme còn 408 khai báo `margin/padding/border-left|right` và 153 `float/text-align: left\|right`, chỉ 13 logical property. Theme sẽ không tự hoạt động đúng ở bố cục RTL. |
| Tích hợp Open Color, quản lý màu tập trung bằng CSS variables | **Cần lưu ý.** Opale ghi đè bằng giá trị màu cứng sinh từ Sass, không dùng cơ chế biến CSS mới. Muốn dùng chung cơ chế của core thì phải chuyển đổi có chủ đích. |

## Bài học

- Không suy đoán rủi ro tương thích từ changelog. Changelog Redmine 7.0.0 nêu
  việc gỡ `icon-*` khỏi core, thoạt nhìn giống lỗi tương thích nghiêm trọng,
  nhưng kiểm tra `components/_icons.scss` cho thấy theme tự cung cấp các class
  này. Luôn đối chiếu changelog với mã nguồn thực tế trước khi kết luận.
