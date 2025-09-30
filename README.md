# MI-RAG for ViOCRVQA

## 🔎 Giới thiệu
Hệ thống **MI-RAG (Multimodal Iterative Retrieval-Augmented Generation)** cho ảnh có chữ/đối tượng.  

Đặc điểm chính:
- Kết hợp **Visual KB** (OCR, Object, Caption) và **Text KB** (Wikipedia tiếng Việt/Anh, tài liệu nội bộ).
- Hỗ trợ **router đa ngữ**, **multi-query planner**, **multi-step reasoning**, và **knowledge fusion**.
- Trả lời kèm **citation** rõ ràng (ảnh, bbox, hoặc đoạn văn bản).
- Baseline tuần 1: **1 vòng retrieval + rerank + citation logging**.

Dataset: [ViOCRVQA](https://github.com/qhnhynmm/ViOCRVQA-Dataset) (~28k ảnh, 124k QA).

---

## 🏗 Cấu trúc repo
<pre>
  mi-rag/
├─ README.md
├─ requirements.txt / pyproject.toml
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
