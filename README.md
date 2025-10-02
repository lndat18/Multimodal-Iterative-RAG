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
- **README.md**
    
    Trang giới thiệu repo: mục tiêu, cách cài, chạy baseline, roadmap.
    
- **requirements.txt**
    
    Danh sách package để `pip install`. (W1: PaddleOCR, ultralytics, chromadb, faiss, sentence-transformers, FlagEmbedding, datasets, pandas, tqdm…)
    
- **.env.example**
    
    Mẫu biến môi trường (API keys, PATH dữ liệu…). Bạn copy thành `.env` để chạy local, tránh commit secret.
    

**`src/`** – mã nguồn chính

- **`src/dataio/`** – nạp & chuẩn hóa dữ liệu
    - `viocrvqa.py`: tải/đọc ViOCRVQA → chuẩn record `{image_path, question, answer, split, meta}`.
    - `wikipedia.py`: tải Wikipedia Vi/En (HF) → tách `chunk` → lưu JSONL.
    - `schemas.py`: dataclass/Pydantic schema cho sample, chunk, evidence.
- **`src/vision/`** – tín hiệu từ ảnh (Visual)
    - `ocr.py`: gọi PaddleOCR → trả `spans` (text, bbox, conf).
    - `detect.py`: YOLOv8 → trả `objects` (label, bbox, conf).
    - `caption.py` *(tùy chọn W2+)*: sinh mô tả ảnh.
- **`src/kb/`** – dựng Knowledge Base
    - `build_visual_kb.py`: gom `ocr_text`, `objects` thành 1 doc/ảnh → `data/visual_kb.jsonl`.
    - `build_text_kb.py`: từ Wikipedia đã chunk → `data/text_kb.jsonl`.
- **`src/index/`** – lập chỉ mục (vector stores)
    - `build_index.py`:
        - Visual KB → **Chroma** (tiện metadata/filter).
        - Text KB → **FAISS** (nhiều tài liệu, nhanh).
    - `searchers.py`: hàm `search_visual(query,k)`, `search_text(query,k)` dùng lại khi truy vấn.
- **`src/retriever/`** – truy xuất + hợp nhất
    - `embed.py`: load **bge-m3** (embedding).
    - `fusion.py`: **RRF**/Weighted merge giữa visual & text.
    - `rerank.py`: **bge-reranker-v2-m3** sắp xếp lại top-N.
    - `router.py`: nhận biết ngôn ngữ (heuristic/fastText) → route Vi/En.
    - `planner.py` *(W2+)*: sinh multi-query (paraphrase, sub-qs, pivot EN…).
- **`src/rag/`** – pipeline trả lời
    - `baseline.py`: **RAG 1 vòng** (W1) → retrieve (visual+text) → fuse → rerank → in `context + citations`.
    - `generator.py` *(W3–W4)*: gọi LLM sinh câu trả lời ràng buộc citation.
- **`src/utils/`** – tiện ích
    - `logging.py`: ghi **JSONL** theo stage `ingest/query/answer` vào `logs/runs/…`.
    - `timing.py`: decorator đo thời gian từng bước.
    - `text.py`: chuẩn hóa Unicode, strip, lọc ký tự rác từ OCR.
