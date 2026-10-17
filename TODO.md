# TODO — Cấu hình bảo mật trước khi publish EasyTrip

Checklist các bước cần làm **trên Firebase / Google Cloud Console** (không nằm trong repo).
Làm xong đánh dấu `[x]`.

## 1. Bật Anonymous Authentication (BẮT BUỘC)

> Nếu bỏ bước này, app vẫn đọc được nhưng mọi thao tác ghi sẽ thất bại âm thầm.

- [ ] Mở https://console.firebase.google.com → chọn project **easytrip-55faf**
- [ ] Menu trái: **Build → Authentication**
- [ ] Nếu chưa bật lần nào, bấm **Get started**
- [ ] Tab **Sign-in method** → tìm **Anonymous** trong danh sách Native providers → bấm vào
- [ ] Gạt công tắc **Enable** → **Save**

## 2. Deploy Firestore Security Rules

> Rules mới có phân quyền chủ/editor, link sửa (`tripSecrets`, `joinRequests`). Phải deploy
> bản mới nhất trong repo, nếu không thì tạo link sửa sẽ báo lỗi quyền.

### Cách A — Dán trực tiếp trên console (dễ nhất, không cần cài gì)

- [ ] Firebase Console → **Build → Firestore Database → tab Rules**
- [ ] Xóa nội dung cũ
- [ ] Dán toàn bộ nội dung file `firestore.rules` trong repo
- [ ] Bấm **Publish**

### Cách B — Dùng Firebase CLI

- [ ] `npm install -g firebase-tools`
- [ ] `firebase login`
- [ ] Project mặc định đã khai báo trong `.firebaserc` (`easytrip-55faf`), không cần `firebase use`
- [ ] `firebase deploy --only firestore:rules`  (chạy trong `D:\Work\EasyTrip`)

## 3. Giới hạn API key (giảm rủi ro lạm dụng quota/billing)

- [x] Mở https://console.cloud.google.com/apis/credentials?project=easytrip-55faf
- [x] Chọn key **Browser key (auto created by Firebase)** (key bắt đầu bằng `AIzaSyDKoaSt`)
- [x] **Application restrictions** = **Websites**, thêm:
  - `https://easytrip-55faf.web.app/*`
  - `https://easytrip-55faf.firebaseapp.com/*`
  - Link preview channel đang dùng, ví dụ `https://easytrip-55faf--test-hgc7szgg.web.app/*`
    (Google không cho `*` ở giữa tên miền; mỗi channel mới phải thêm địa chỉ của nó)
  - `http://localhost:8000/*` (chạy local bằng `Start.bat`)
- [x] **API restrictions** = **Restrict key**, chỉ tick: Identity Toolkit API, Token Service API, Cloud Firestore API
- [x] Save, đợi ~5 phút rồi mở app kiểm tra vẫn tạo được chuyến đi

## 4. Dọn bản Hosting cũ (từng lộ `.git/`)

- [ ] Firebase Console → **Hosting** → bảng **Release history** của site `easytrip-55faf`
- [ ] Với bản deploy đầu tiên ngày 17/10/2026 (bản 53 file): menu ⋮ → **Delete**
- [ ] (Tùy chọn) **Release storage settings** → giữ tối đa vài bản để bản cũ tự bị xóa

## 5. Quy tắc về Gemini API key

- [ ] **KHÔNG** dán Gemini API key trực tiếp vào `index.html` (key trong JS tĩnh sẽ bị bot GitHub lấy ngay)
- [ ] Muốn dùng AI khi publish: gọi Gemini qua backend trung gian (Cloud Functions / proxy)
      hoặc để người dùng tự nhập key vào `localStorage`
- [ ] Xác nhận `.gitignore` đã chặn `*.local` và các file `.env*`

## 6. Kiểm tra sau khi deploy

- [ ] Mở app, bấm **F12** → Console, không thấy log `Anonymous sign-in failed`
- [ ] Thử tạo 1 chuyến đi và lưu được
- [ ] Mở chuyến đi → Chia sẻ → **Sao chép link sửa** chạy được (nếu lỗi quyền → rules chưa publish bản mới)
- [ ] Mở link xem ở trình duyệt khác → chỉ xem, không sửa được
- [ ] Nếu log báo `Missing or insufficient permissions` → Anonymous chưa bật hoặc rules chưa publish

## 7. Chuyến đi cũ

- [ ] Chuyến đi tạo trước khi có phân quyền sẽ không còn hiện ở trang chủ. Mở bằng link
      `#trip=<id>` (lấy id trong Firebase Console → Firestore → `trips`) và bấm **Nhận quyền sở hữu**
