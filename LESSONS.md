# Bài học & quyết định kỹ thuật

Tài liệu ghi nhận các quyết định kỹ thuật và bài học của dự án theme Opale.
Rà soát tài liệu này trước khi bắt đầu một phiên làm việc mới.

> Tài liệu phải nằm ở thư mục gốc của repo, cạnh `README.md`. Lý do ở mục
> "Không tạo thư mục con mới trong repo theme" phần Bài học.

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

## Quyết định: màu chủ đạo PCCC

`$brand-primary` đổi từ `#628db6` sang **`#af2b1e` (RAL 3000 Flame Red)**.

Căn cứ lựa chọn:

- **Đúng chuẩn ngành.** RAL 3000 (*Feuerrot*) là màu tiêu chuẩn cho phương tiện
  và thiết bị PCCC. TCVN 4879:1989 (ISO 6309:1987) quy định dấu hiệu an toàn
  PCCC dùng nền đỏ với ký hiệu trắng; hình dạng và màu tuân theo TCVN 5053:1990.
- **Khả năng tiếp cận.** Tương phản với chữ trắng đạt **6.59:1**, vượt WCAG AA
  (4.5:1) và gần AAA (7:1). Màu cũ `#628db6` chỉ đạt **3.50:1**, tức dưới chuẩn
  AA — thay đổi này đồng thời sửa một lỗi tiếp cận có sẵn.
- **Tách bạch ngữ nghĩa.** Đủ trầm để không lẫn với `$red: #e5123d`, vốn đang
  gánh ngữ nghĩa danger/error/priority khẩn (`$brand-danger`, `$link-hover-color`,
  `$flash-error-bg`, `$sidebar-link-active-side`). Hai sắc đỏ phục vụ hai vai trò
  khác nhau và phải phân biệt được.

Các ứng viên bị loại và lý do:

| Ứng viên | Tương phản | Lý do loại |
|---|---|---|
| `#c8102e` (ISO safety red) | 5.88:1 | `color.adjust($saturation: 25%)` đẩy saturation lên **110.18%**, vượt gamut |
| `#cc0605` (RAL 3020) | — | `shade()` sinh saturation **105.22%**, vượt gamut |
| `#a52019` (RAL 3001) | 7.45:1 | Đạt yêu cầu nhưng RAL 3000 sát định danh thiết bị PCCC hơn |
| `#9b2423` | 7.86:1 | Ngả nâu, giảm nhận diện đỏ PCCC |
| `#b71c1c` (Material Red 800) | 6.57:1 | Không thuộc chuẩn an toàn nào |

## Quyết định: giao diện theo phong cách Woffice

Thiết kế lại theo ảnh mẫu giao diện Woffice (skin *celestial*), giữ nguyên
kiến trúc SCSS sẵn có và chỉ thay design token + tinh chỉnh component.

Ràng buộc nền tảng: theme Redmine **chỉ can thiệp được CSS**, không sửa được
HTML. Cấu trúc Redmine 7.0.0 là cố định:

```
#wrapper › #top-menu › #header › #main › #sidebar › #content › #footer
```

Vì vậy tái tạo được phong cách thị giác (khung tối, card, bo góc, kiểu chữ)
nhưng không tái tạo được các widget riêng của Woffice (Poll, Groups, Upcoming
Birthdays) — Redmine không có dữ liệu tương ứng.

Token đã áp dụng:

| Token | Giá trị |
|---|---|
| Primary / accent | `#3146c5` / `#f161a1` |
| Nền trang / bề mặt | `#f4f5f6` / `#fff` |
| Chữ / chữ phụ | `#131313` / `#667085` |
| Success / danger / warning | `#21bd98` / `#eb5757` / `#f78915` |
| Bo góc: base / large / pill | 10px / 15px / 25px |
| Shadow card | `0 10px 50px 0 rgba(19,19,19,.1)` |

## Bài học

- **Kiểm tra subset tiếng Việt trước khi chọn webfont.** Ảnh mẫu dùng Poppins,
  nhưng Google Fonts chỉ phát hành Poppins với subset `latin`, `latin-ext` và
  `devanagari`. Toàn bộ nguyên âm có dấu tiếng Việt nằm ở khối U+1EA0–U+1EF9,
  không thuộc `latin-ext`, nên chúng sẽ rơi sang font dự phòng và phá vỡ nhịp
  chữ. Đã thay bằng **Be Vietnam Pro** — cùng phong cách geometric sans, có
  subset `vietnamese`. Cách kiểm chứng nhanh một font bất kỳ:
  `curl "https://fonts.googleapis.com/css2?family=<Ten>:wght@400" | grep vietnamese`
  (phải kèm User-Agent của trình duyệt, nếu không Google trả về định dạng cũ).
  Font được self-host trong `webfonts/` để bản cài nội bộ không phụ thuộc CDN.

- **Phân biệt `$body-bg` với `$surface-bg`.** Trước đây nền trang là màu trắng
  nên `$body-bg` bị dùng lẫn cho hai vai trò: nền trang *và* nền của các bề mặt
  nổi (dropdown, dialog, tooltip, datepicker, tab đang mở). Khi nền trang
  chuyển sang xám, 26 chỗ trong `components/` lập tức sai màu. Nay `$body-bg`
  chỉ dành cho `body`, mọi bề mặt nổi dùng `$surface-bg`. Khi đổi nền trang,
  luôn rà `grep -rn 'variables.\$body-bg' components/` trước.

- **Bảng màu thiết kế không đương nhiên đạt chuẩn tiếp cận.** Bảng màu Woffice
  có 3 cặp dưới WCAG AA: chữ phụ `#8590a6` trên nền trang chỉ đạt 2.94:1, và
  `#f161a1` với chữ trắng chỉ 3.02:1. Cách xử lý giữ được cả hai mục tiêu: giữ
  màu gốc cho **bề mặt và ký hiệu**, dùng biến thể tối hơn (`#667085`,
  `#d81b60`) cho **mọi chỗ có chữ**. Nhận diện không đổi, mọi cặp đạt AA.

- **Thêm shadow thì phải reset trong bản in.** `components/_print.scss` không
  reset `box-shadow`, nên card mới sẽ in ra bóng xám tốn mực. Bản in đã ẩn sẵn
  `#top-menu`, `#header`, `#sidebar` nên khung tối không thành mảng đen, nhưng
  card thì cần tự xử lý.

- **Nền tối phải reset ở cả trạng thái `.nosidebar`.** `#main.nosidebar #sidebar`
  reset `margin`, `padding`, `border` nhưng không reset `background`, nên nền
  tối vẫn dính lại trên những trang không có sidebar.

- **Lệnh lint của dự án từng bỏ sót 16/58 file.** `stylelint src/sass/**/*.scss`
  không được đặt trong dấu nháy, nên shell (không bật `globstar`) rút `**`
  thành `*` và chỉ quét đúng một cấp thư mục con — bỏ qua cả `_variables.scss`,
  `application.scss` và toàn bộ `plugins/`. Đã thêm dấu nháy để stylelint tự
  phân giải glob. Hệ quả: `npm run lint` báo pass trong khi husky lại chặn
  commit, vì lint-staged kiểm tra đúng file được stage.



- **Không tạo thư mục con mới trong repo theme.** Redmine cài theme bằng cách
  đặt nguyên repo vào `{redmine}/public/themes/opale`, và asset pipeline
  precompile **các thư mục con** của theme. Đặt tài liệu ở `docs/lessons.md`
  khiến `rake assets:precompile` sinh ra `themes/opale/lessons-<hash>.md`, tức
  tài liệu nội bộ bị publish thành asset công khai. Các file ở thư mục gốc
  (`README.md`, `AUTHORS.md`, `CONTRIBUTING.md`) không bị quét, nên tài liệu
  mới phải đặt cạnh chúng ở gốc repo. Đã chuyển thành `LESSONS.md`.

- **Kiểm tra dư địa saturation trước khi chọn màu chủ đạo.** Theme dùng
  `color.adjust($saturation: 25%)` ở `$input-border-focus` và
  `$bubble-target-border`, còn `shade()` tự cộng saturation cho các nấc tối.
  Màu đầu vào đã bão hòa cao sẽ vượt 100% và khiến Sass xuất `hsl()` thay vì
  `rgb()`/hex như phần còn lại của theme. Ngưỡng an toàn: saturation đầu vào
  dưới ~75%. Cách kiểm chứng đã dùng:
  `npx sass --load-path=src/sass probe.scss` với `@use "variables" with (...)`
  để lấy giá trị thật từ chính trình biên dịch, thay vì tính tay.
- **Đối chiếu trước/sau bằng tập hợp màu, không đọc diff của CSS đã nén.**
  `stylesheets/application.css` là một dòng duy nhất nên `git diff` vô dụng.
  Cách hiệu quả: trích `#hex` và `rgb()` từ `git show HEAD:<file>` và từ bản mới,
  `sort -u` rồi `comm` hai tập. Một thay đổi đúng phạm vi phải cho kết quả cân
  bằng 1:1 — lần này đúng 1 hex và 8 rgb đổi chỗ, tổng số màu giữ nguyên 29.
- **Phân biệt lỗi có sẵn với lỗi mới sinh.** CSS chứa nhiều `hsl()` có
  saturation > 100% (từ `$orange`, `$teal`, `$green`, `$pink`). Đối chiếu với
  `git show HEAD:` cho thấy chúng tồn tại từ trước và không liên quan tới thay
  đổi này. Luôn so với baseline trước khi kết luận mình gây ra lỗi.
- Không suy đoán rủi ro tương thích từ changelog. Changelog Redmine 7.0.0 nêu
  việc gỡ `icon-*` khỏi core, thoạt nhìn giống lỗi tương thích nghiêm trọng,
  nhưng kiểm tra `components/_icons.scss` cho thấy theme tự cung cấp các class
  này. Luôn đối chiếu changelog với mã nguồn thực tế trước khi kết luận.
