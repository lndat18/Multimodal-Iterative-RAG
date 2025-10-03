# MI-RAG for ViOCRVQA

## 🔎 Giới thiệu
Hệ thống **MI-RAG (Multimodal Iterative Retrieval-Augmented Generation)** cho ảnh có chữ/đối tượng.  

Đặc điểm chính:
- Kết hợp **Visual KB** (OCR, Object, Caption) và **Text KB** (Wikipedia tiếng Việt/Anh, tài liệu nội bộ).
- Hỗ trợ **router đa ngữ**, **multi-query planner**, **multi-step reasoning**, và **knowledge fusion**.
- Trả lời kèm **citation** rõ ràng (ảnh, bbox, hoặc đoạn văn bản).

Dataset: [ViOCRVQA](https://github.com/qhnhynmm/ViOCRVQA-Dataset) (~28k ảnh, 124k QA).

---

## 🏗 Cấu trúc repo
<pre>
  mi-rag/
├─ README.md
├─ requirements.txt
├─ .env.example
├─ src/
│ ├─ dataio/ # load ViOCRVQA, Wikipedia
│ ├─ vision/ # OCR, object detection, caption
│ ├─ kb/ # build visual/text KB
│ ├─ index/ # FAISS/Chroma
│ ├─ retriever/ # dual retriever, rerank, fusion
│ ├─ rag/ # baseline pipeline
│ └─ utils/ # logging, config
├─ data/
│ ├─ viocrvqa/
│ └─ wiki/
├─ logs/
│ └─ runs/
└─ notebooks/ # demo, sanity check
</pre>

Trong đó
Thư mục gốc

README.md: “bảng hướng dẫn sử dụng nhanh” – cách cài đặt, cách chạy demo.

requirements.txt: danh sách thư viện Python để pip install -r requirements.txt.

.env.example: mẫu biến môi trường (đường dẫn dữ liệu, API key…). Bạn copy thành .env rồi điền giá trị thật.

src/ – Code chính theo từng “công đoạn”

src/dataio/ – Nhập hàng & đóng gói dữ liệu

Code để tải/đọc ViOCRVQA, Wikipedia (Vi/En), tiền xử lý (clean text, tách trường, chuẩn format).

Kết quả: trả về các mẫu dữ liệu có cấu trúc (question, image path, ocr_spans, …) để các bước sau dùng.

src/vision/ – Nhìn ảnh để lấy thông tin

Chạy OCR (lấy text trong ảnh), object detection (tên/box các vật thể), captioning (mô tả ảnh).

Kết quả: sinh ra feature/metadata từ ảnh (ocr text, boxes, labels, caption).

src/kb/ – Xây “kho tri thức”

Dùng thông tin từ vision/ (phần visual) và dataio/ (Wiki, docs – phần text) để build Knowledge Base:

Visual KB: vector/metadata của ảnh, OCR, objects, caption.

Text KB: đoạn văn Wiki, docs đã tách nhỏ (chunks) + embeddings.

Kết quả: dữ liệu đã “đóng thùng” sẵn sàng đưa vào chỉ mục.

src/index/ – Dựng chỉ mục để tìm cho nhanh

Tạo FAISS/Chroma index cho visual và text (tách riêng để truy xuất song song).

Cung cấp API: add/search vectors, lưu/đọc index từ đĩa.

src/retriever/ – Lấy hàng đúng & xếp hạng lại

Dual retriever: gọi cả hai kho (visual & text) để lấy top-k ứng viên.

Fusion (hợp nhất) & Rerank: loại nhiễu, sắp xếp lại theo điểm phù hợp (BM25/RRF/cross-encoder…).

Trả về gói “tài liệu/visual chunks” tốt nhất để model trả lời.

src/rag/ – Dây chuyền lắp ráp câu trả lời

Baseline pipeline: OCR+Detect → Embed → Retrieve song song → Rerank → Generate → Citation.

Gọi retriever, ráp ngữ cảnh, gọi LLM sinh câu trả lời kèm trích dẫn nguồn.

src/utils/ – Dụng cụ & cấu hình chung

logging (ghi thời gian, top-k, điểm rerank…), config loader (đọc .env, YAML), tiện ích (seed, timer…).

data/ – Kho dữ liệu

data/viocrvqa/: ảnh câu hỏi/trả lời, annotation… dùng cho OCR-VQA.

data/wiki/: dump/đoạn văn Wikipedia đã tách và chuẩn hoá (JSONL/Parquet…).

Bạn không commit dữ liệu thật lên git (nặng & riêng tư); chỉ commit script và .env.example.

logs/

logs/runs/: nơi rơi file log mỗi lần chạy (thời gian, thông số, top-k, nguồn trích dẫn, latency…).
Dùng để debug và so sánh các lần thử.

notebooks/

Notebook demo/sanity check:

test nhanh OCR một ảnh,

thử build 10 mẫu wiki chunks,

gọi retriever lấy top-k,

chạy pipeline end-to-end cho 1–2 câu hỏi.
