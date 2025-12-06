# AgriVision – Hệ thống nhận dạng và phân loại độ chín trái mít

Ứng dụng AI phục vụ Nông nghiệp Thông minh.  
Kết hợp YOLOv8, Streamlit (frontend) và FastAPI (backend) để phân tích hình ảnh, video và quản lý dữ liệu.

---

## 🌿 Cấu trúc thư mục (mô tả)

```
mit_detection_demo/
│
├── frontend/
│   ├── app.py                 # Entrypoint Streamlit
│   ├── assets/                # Hình ảnh, logo, CSS, font
│   ├── fonts/
│   ├── pages/                 # Các trang Streamlit (home, analysis, ...)
│   │   ├── home_page.py
│   │   ├── login_page.py
│   │   ├── analysis_page.py
│   │   ├── video_page.py
│   │   ├── stats_page.py
│   │   ├── compare_page.py
│   │   ├── chat_page.py
│   │   └── account_page.py
│   └── utils/
│       └── helpers.py
│
├── backend/
│   ├── main.py                # Entrypoint FastAPI (uvicorn main:app)
│   ├── auth.py                # Đăng ký / đăng nhập
│   ├── config.py              # Cấu hình (env)
│   ├── model_loader.py        # Load YOLOv8 / tiện ích model
│   ├── predictor.py           # Endpoint /predict xử lý ảnh / video
│   ├── mongodb_connection.py  # Kết nối DB
│   └── utils.py               # Helpers chung (thumbnail, logging,...)
│
├── yolov8/
│   └── best.pt                # Model YOLOv8 (weights) hoặc tham chiếu
│
└── requirements.txt
```

---

## 📦 requirements.txt (chính)

```txt
# FRONTEND
streamlit
streamlit-option-menu
pandas
numpy
matplotlib
plotly
opencv-python
pillow
requests
python-dotenv
google-generativeai

# BACKEND
fastapi
uvicorn
pydantic
python-multipart
pymongo
pandas
numpy
opencv-python
ultralytics
pillow
python-dotenv

# COMMON
tqdm
typing-extensions
```

---

## ⚙️ Cài đặt môi trường

```bash
# 1. Tạo môi trường ảo
python -m venv venv
source venv/bin/activate   # Linux/Mac
venv\Scripts\activate      # Windows

# 2. Cài thư viện
cd mit_detection_demo
pip install -r requirements.txt
```

Tạo file `.env` (frontend & backend) theo phần "Biến môi trường" bên dưới.

---

## 🚀 Chạy hệ thống

### 1) Chạy backend (FastAPI)

Từ thư mục `backend/` hoặc gốc chứa `main.py`:

```bash
uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

API mặc định: http://127.0.0.1:8000

Các endpoint chính:

| Method | Endpoint | Mô tả |
|---:|---|---|
| POST | /auth/register | Đăng ký user (email/password) |
| POST | /auth/login | Đăng nhập (trả token/session) |
| POST | /predict | Dự đoán từ ảnh (multipart/form-data) |
| POST | /predict/video | Dự đoán từ video (upload) |
| GET  | /health | Kiểm tra trạng thái service |

Ví dụ curl tới `/predict`:

```bash
curl -X POST "http://127.0.0.1:8000/predict" \
  -H "accept: application/json" \
  -H "Authorization: Bearer <TOKEN>" \
  -F "file=@/path/to/image.jpg;type=image/jpeg"
```

---

### 2) Chạy frontend (Streamlit)

Từ thư mục `frontend/`:

```bash
streamlit run app.py
```

Mở: http://localhost:8501

Frontend sẽ gọi API backend để upload ảnh/video, hiển thị kết quả, vẽ biểu đồ và quản lý người dùng.

---

## 🔑 Biến môi trường (mẫu .env)

Tạo file `.env` cho backend & frontend (hoặc chung):

```env
# Backend
MONGO_URI=mongodb+srv://user:pass@cluster.mongodb.net/dbname
JWT_SECRET=your_jwt_secret
YOLO_WEIGHTS_PATH=./yolov8/best.pt

# Frontend / Chat
GEMINI_API_KEY=your_google_gemini_key_here

# Chung
API_BASE_URL=http://127.0.0.1:8000
```

Lưu ý: Không commit `.env` lên git.

---

## 🧭 Thiết kế API & luồng chính

- Upload ảnh/video → Backend nhận file → predictor tiền xử lý (resize, normalize) → chạy YOLOv8 → trả nhãn, bounding boxes, confidence.
- Lưu kết quả: metadata vào MongoDB (user_id, image_url/video_url, label, confidence, created_at).
- Chat AgriVision (Streamlit): gửi text/ảnh => gọi Google Generative API (Gemini) để phân tích/nội dung tư vấn.
- So sánh model: frontend upload CSV kết quả training (YOLOv8 runs) → backend parse/compute F1 → lưu `compare_history`.

---

## 🗄️ Mô hình dữ liệu (gợi ý schema MongoDB)

- users
  - _id, email, password_hash, created_at, profile...
- image_history
  - _id, user_id, image_url, labels: [{label, confidence, bbox}], created_at
- video_history
  - _id, user_id, video_url, thumbnail_url, labels, created_at
- compare_history
  - _id, user_id, model_a: {precision, recall, map50,...}, model_b: {...}, created_at
- chat_messages
  - _id, user_id, role (user/ai), text, file_url?, created_at

---

## 💡 Tính năng chính

- Phân tích ảnh: nhận dạng độ chín & sâu bệnh bằng YOLOv8
- Phân tích video/webcam: phát hiện real-time (trích frame hoặc streaming)
- Thống kê & biểu đồ: lưu lịch sử, filter theo nhãn/thời gian, biểu đồ (Plotly/Matplotlib)
- So sánh YOLOv8: upload CSV (results.csv) để so sánh chỉ số, tính F1
- Chat AgriVision: trợ lý AI (Gemini) có thể trả lời text & phân tích ảnh
- Quản lý tài khoản: đăng ký, đăng nhập, phiên làm việc

---

## 🧩 Công nghệ sử dụng (tóm tắt)

| Thành phần | Vai trò |
|---|---|
| Streamlit | Frontend UI nhanh, interactive dashboard |
| FastAPI | Backend REST API, xử lý file & inference |
| YOLOv8 (Ultralytics) | Model object detection |
| Google Generative AI (Gemini) | Chat & phân tích ảnh theo ngữ cảnh |
| MongoDB | Lưu lịch sử & metadata |
| OpenCV / Pillow | Tiền xử lý ảnh & frame |
| Plotly / Matplotlib | Visualize kết quả & biểu đồ |

---

## 🔧 Gợi ý triển khai / production

- Chạy backend bằng uvicorn + reverse proxy (nginx) hoặc containerize bằng Docker.
- Sử dụng GPU (nếu cần tốc độ inference) trên backend — cài CUDA & Pytorch tương thích.
- Lưu trữ file (ảnh/video) trên object storage (S3 / DigitalOcean Spaces) hoặc bucket riêng; lưu URL trong DB.
- Bảo mật: dùng HTTPS, JWT tokens, rate limit, và RLS (nếu dùng Supabase thay vì Mongo).
- Logging & monitoring (Prometheus / Sentry) cho production.

---

## 📝 Tài liệu tham khảo & ghi chú

- Model YOLOv8: https://github.com/ultralytics/ultralytics
- FastAPI docs: https://fastapi.tiangolo.com
- Streamlit docs: https://docs.streamlit.io
- Google Generative AI: kiểm tra chính sách và quota trước khi dùng

---

## 📸 Demo giao diện (mô tả nhanh)

| Trang | Mô tả |
|---|---|
| Trang chủ | Giới thiệu hệ thống & chỉ dẫn sử dụng |
| Đăng nhập | Quản lý user (streamlit form) |
| Phân tích ảnh | Upload ảnh, xem bounding box, kết quả |
| Video / Webcam | Tải video hoặc bật webcam để phân tích |
| Thống kê | Biểu đồ tổng quan, filter theo thời gian |
| So sánh YOLOv8 | Upload CSV kết quả training, so sánh chỉ số |
| Chat AgriVision | Trò chuyện & gửi ảnh để AI phân tích |
| Tài khoản | Thông tin user, lịch sử, logout |

---

## 🧪 Kiểm thử nhanh

- Gửi ảnh sample tới `/predict` và kiểm tra JSON trả về (labels, confidences, bboxes).
- Kiểm tra upload CSV ở trang So sánh → xem biểu đồ & lưu lịch sử.
- Gửi ảnh qua Chat → nhận phân tích từ Gemini.

---

## 🧾 Giấy phép

MIT License © 2025 — AgriVision

---

## Nhóm thực hiện
- Nguyễn Văn Duy – 2151220251  
- Lê Nam – 2151220149
