# BVTV AI - App Android chẩn đoán sâu bệnh & đề xuất thuốc

Repo này build ra **file APK tự động bằng GitHub Actions** — mỗi lần bạn push code
lên nhánh `main`, tab **Actions** sẽ tự chạy, build APK và để sẵn cho bạn tải về,
không cần cài Android Studio trên máy.

Lưu ý: tôi (AI) không có tài khoản GitHub/Cloudflare của bạn nên không tự push/deploy
giúp được — nhưng mọi thứ đã viết sẵn, bạn chỉ copy lệnh bên dưới và chạy.

## Cấu trúc repo

```
bvtv-ai-android/
├── www/                    # App web (giao diện chụp ảnh, kết quả...)
├── android/                # Project Android (Capacitor) - nơi Gradle build ra APK
├── cloudflare-worker/      # Máy chủ trung gian giấu API key Gemini
├── .github/workflows/build-apk.yml   # Tự build APK mỗi khi push
├── capacitor.config.ts
└── package.json
```

---

## Bước 1 — Deploy Cloudflare Worker (giấu API key Gemini)

Lấy API key miễn phí tại https://aistudio.google.com/apikey, rồi:

```bash
cd cloudflare-worker
npm install
npx wrangler login
npx wrangler secret put GEMINI_API_KEY     # dán API key vào khi được hỏi
npx wrangler deploy
```

Terminal sẽ in ra URL dạng `https://postbvtv-ai-proxy.<ten-cua-ban>.workers.dev` — **copy lại**.

## Bước 2 — Gắn URL worker vào app

Mở `www/index.html`, tìm dòng:
```js
const DEFAULT_WORKER_URL = "https://postbvtv-ai-proxy.YOUR-SUBDOMAIN.workers.dev";
```
Thay bằng URL thật ở Bước 1, lưu lại.

## Bước 3 — Đẩy toàn bộ repo lên GitHub

```bash
cd ..    # về thư mục gốc bvtv-ai-android
git init
git add .
git commit -m "BVTV AI - app + worker + auto build APK"
git branch -M main
git remote add origin https://github.com/<username-cua-ban>/bvtv-ai-android.git
git push -u origin main
```

## Bước 4 — Lấy file APK từ tab Actions

1. Vào repo trên GitHub → tab **Actions**
2. Sẽ thấy workflow **"Build Android APK"** tự chạy (mất khoảng 3-5 phút lần đầu)
3. Khi chạy xong (dấu ✅ xanh), bấm vào lần chạy đó → kéo xuống mục **Artifacts**
   → tải file `BVTV-AI-apk.zip` về, giải nén ra được `BVTV-AI.apk`
4. Ngoài ra workflow cũng tự tạo mục **Releases** (ở trang chính repo, cột phải)
   với bản phát hành tên **"latest"** — mỗi lần push code mới, file APK trong đó
   cũng được cập nhật, tiện để lấy link tải cố định gửi cho người khác

Chép file `.apk` vào điện thoại Android → mở lên cài (Android có thể hỏi "Cho phép
cài từ nguồn không xác định", bấm Cho phép).

> Đây là bản APK **debug** (chưa ký release, không cần keystore) — cài và dùng
> bình thường trên điện thoại của bạn. Nếu sau này muốn đăng lên Google Play,
> cần thêm bước tạo keystore ký release, có thể hỏi tôi khi tới lúc đó.

## Từ lần sau: chỉ cần push code là có APK mới

Mỗi khi bạn sửa gì trong `www/` (giao diện) hoặc `android/` rồi:
```bash
git add . && git commit -m "cập nhật" && git push
```
GitHub Actions sẽ tự build lại APK mới, không cần làm gì thêm.

---

## Cách hoạt động của app

1. Nông dân chụp ảnh cây bị bệnh, chọn loại cây + giai đoạn sinh trưởng (có thể để trống)
2. App gửi ảnh lên Cloudflare Worker
3. Worker gọi Gemini Vision chẩn đoán bệnh/sâu hại + đề xuất hoạt chất + tư vấn phân bón
4. Worker đối chiếu hoạt chất AI đề xuất với `data.json` (danh mục thuốc thật của bạn
   trên GitHub `POSTBVTV/data.json`) → trả về **tên thuốc thật + công ty**
5. App hiển thị: chẩn đoán, mức độ nặng nhẹ, hoạt chất, danh sách thuốc khớp, gợi ý phân bón

## Giới hạn cần lưu ý

- Công cụ hỗ trợ tham khảo, không thay thế chẩn đoán trực tiếp của kỹ sư nông nghiệp
  với ca bệnh nặng/lạ hoặc ảnh không rõ
- Gemini free tier giới hạn số lượt gọi/phút — dùng nhiều nên nâng cấp gói trả phí
- App cần internet để gọi AI (chụp ảnh/xem giao diện thì không cần)
