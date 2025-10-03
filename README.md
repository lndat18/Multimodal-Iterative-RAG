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


---

## 🏠 Root Directory
- **`README.md`** – Hướng dẫn sử dụng nhanh: cài đặt, chạy demo.  
- **`requirements.txt`** – Danh sách thư viện Python (`pip install -r requirements.txt`).  
- **`.env.example`** – Mẫu biến môi trường (đường dẫn dữ liệu, API key).  
  > Copy thành `.env` và điền giá trị thật.  

---

## ⚙️ `src/` – Code chính theo từng công đoạn

### `src/dataio/` – **Nhập dữ liệu**
- Tải/đọc **ViOCRVQA, Wikipedia (Vi/En)**  
- Tiền xử lý (clean text, tách trường, chuẩn format)  
- ✅ Kết quả: dữ liệu có cấu trúc `{question, image_path, ocr_spans, …}`  

### `src/vision/` – **Xử lý ảnh**
- **OCR**: lấy text trong ảnh  
- **Object detection**: tên/box vật thể  
- **Captioning**: mô tả ảnh  
- ✅ Kết quả: metadata từ ảnh (ocr text, boxes, labels, caption)  

### `src/kb/` – **Xây dựng Knowledge Base**
- Gom thông tin từ `vision/` (visual) và `dataio/` (text)  
- **Visual KB**: vector/metadata ảnh, OCR, objects, caption  
- **Text KB**: đoạn Wiki/docs → chunk + embeddings  
- ✅ Kết quả: dữ liệu sẵn sàng đưa vào chỉ mục  

### `src/index/` – **Dựng chỉ mục**
- Xây **FAISS/Chroma index** cho visual & text (song song)  
- API: thêm/tìm vector, lưu/đọc index  

### `src/retriever/` – **Truy xuất & xếp hạng**
- **Dual retriever**: gọi cả visual & text KB → lấy top-k  
- **Fusion + Rerank**: loại nhiễu, xếp hạng theo BM25/RRF/cross-encoder  
- ✅ Kết quả: tập ứng viên tốt nhất cho model trả lời  

### `src/rag/` – **Pipeline sinh câu trả lời**
- Baseline: `OCR + Detect → Embed → Retrieve → Rerank → Generate → Citation`  
- Gọi retriever, ráp ngữ cảnh, gọi LLM sinh đáp án **có trích dẫn**  

### `src/utils/` – **Tiện ích chung**
- Logging (ghi thời gian, top-k, rerank score, …)  
- Config loader (`.env`, YAML)  
- Hàm tiện ích: seed, timer, …  

---

## 📦 `data/` – Kho dữ liệu
- **`data/viocrvqa/`**: ảnh, câu hỏi/trả lời, annotation (OCR-VQA).  
- **`data/wiki/`**: dump Wikipedia đã tách nhỏ & chuẩn hoá (JSONL/Parquet).  

> ⚠️ Không commit dữ liệu thật (nặng & riêng tư). Chỉ commit **script** và **.env.example**.  

---

## 📝 `logs/`
- **`logs/runs/`**: chứa log mỗi lần chạy  
  - thời gian, thông số, top-k, rerank score, citation, latency  
- Dùng để debug & so sánh thử nghiệm  

---

## 📒 `notebooks/`
Notebook demo/sanity check:
- Test nhanh OCR 1 ảnh  
- Build thử 10 wiki chunks  
- Gọi retriever lấy top-k  
- Chạy pipeline end-to-end cho 1–2 câu hỏi  

---

✨ Với cấu trúc này, bạn có thể dễ dàng mở rộng:  
- Thêm dataset mới → đặt file vào `data/` + viết loader trong `src/dataio/`  
- Đổi OCR/detector → `src/vision/`  
- Thêm nguồn tri thức mới → `src/kb/`  
- Đổi index (FAISS ↔ Chroma) → `src/index/`  
- Điều chỉnh fusion/rerank → `src/retriever/`  
- Tinh chỉnh pipeline → `src/rag/`  

---

Bạn có muốn mình viết thêm một **bảng tóm tắt (1 dòng / thư mục)** để cuối README trông “gọn” và tiện nhìn hơn không?
