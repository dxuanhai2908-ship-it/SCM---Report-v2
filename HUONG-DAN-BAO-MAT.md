# Chuyển SCM-report sang đăng nhập bằng Firebase Authentication

Bản cũ chỉ che dữ liệu bằng một màn hình đăng nhập: dữ liệu vẫn được tải về trước khi đăng nhập, và Firestore cho phép bất kỳ ai có link đọc/ghi. Bản mới chỉ đọc dữ liệu sau khi đăng nhập bằng email, và Firestore Rules chặn mọi người không có trong danh sách thành viên.

Làm theo đúng thứ tự dưới đây để team không bị gián đoạn. Người làm cần quyền Owner/Editor trên project Firebase `scm-project-ea359`.

## Bước 1 — Bổ sung email cho mọi thành viên (trên bản cũ, trước khi deploy)

Mở app bản hiện tại → tab **Quản trị** → **Sửa** từng thành viên, điền **email công ty**. Bản mới dùng email để đăng nhập và để phân quyền; thành viên không có email sẽ không vào được.

## Bước 2 — Bật đăng nhập bằng email

Firebase Console → **Authentication** → **Get started** (nếu chưa bật) → tab **Sign-in method** → **Email/Password** → bật **Enable** → **Save**.

## Bước 3 — Tạo tài khoản cho thành viên hiện có

Firebase Console → **Authentication** → tab **Users** → **Add user**. Nhập email (trùng với email ở Bước 1) và một mật khẩu tạm cho từng người. Làm ít nhất cho tài khoản Admin trước.

Sau khi bản mới chạy, Admin có thể thêm thành viên mới ngay trong app (tab Quản trị) — app tự tạo tài khoản đăng nhập. Mỗi người tự đổi mật khẩu bằng nút **Quên mật khẩu?** ở màn đăng nhập.

## Bước 4 — Deploy bản mới

Gộp nhánh `fix/bao-mat-va-so-lieu` vào `main`. Vercel tự deploy.

## Bước 5 — Admin đăng nhập một lần

Admin mở app, đăng nhập bằng email + mật khẩu ở Bước 3. App sẽ tự:
- tạo tài liệu `system/access` (danh sách email được vào, admin, chỉ xem), và
- xoá mật khẩu kiểu cũ (salt/hash) khỏi `system/members`.

Kiểm tra: Firebase Console → **Firestore Database** → collection `system` → phải có document `access` chứa đúng các email.

## Bước 6 — Bật Firestore Rules

Firebase Console → **Firestore Database** → tab **Rules** → xoá nội dung cũ, dán toàn bộ file `firestore.rules` trong repo này → **Publish**.

## Bước 7 — Kiểm tra

1. Mở app trong cửa sổ ẩn danh, **chưa đăng nhập**: chỉ thấy màn đăng nhập, bảng task trống.
2. Đăng nhập bằng tài khoản thành viên: thấy dữ liệu, tạo/sửa task được, không thấy tab Quản trị.
3. Đăng nhập bằng tài khoản Admin: thấy tab Quản trị, sửa được thành viên.

## Nếu có sự cố

- **App lỗi sau khi deploy:** Vercel → project → **Instant Rollback** về bản trước.
- **Không ai đọc được dữ liệu sau khi bật Rules:** kiểm tra `system/access` đã có và chứa email đúng (chữ thường). Có thể tạm khôi phục rules cũ trong tab Rules (Firebase lưu lịch sử các lần Publish).

## Khuyến nghị thêm

- Chuyển các repo GitHub chứa app sang **Private**.
- Google Cloud Console → **APIs & Services** → **Credentials** → API key của app → giới hạn **HTTP referrers** chỉ cho `scm-report-v2.vercel.app/*`.
