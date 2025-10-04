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
  mi_rag/
    ├── env/
    │   ├── requirements.txt
    │   └── config.json                 # tham số chung: top_k, num_rounds, paths...
    ├── data/                           # ảnh/tài liệu thô (visual_raw, texts_raw)
    ├── indices/                        # FAISS/Chroma lưu sẵn để tái dùng
    ├── artifacts/                      # checkpoint retriever/reranker (nếu có)
    ├── logs/                           # jsonl: query, top-k, điểm rerank, thời gian
    └── notebooks/
        ├── 00_setup.ipynb              # mount Drive + pip install + kiểm tra GPU + load config
        ├── 10_build_kb.ipynb           # TẠO KB: OCR/object/caption + tách văn bản
        ├── 20_index_embed.ipynb        # EMBED + LẬP CHỈ MỤC: Visual KB & Text KB
        ├── 30_mi_rag.ipynb             # PIPELINE CHÍNH: planner → retrieval (2–3 vòng) → rerank → fusion+citation
        ├── 40_eval.ipynb               # ĐÁNH GIÁ: Recall@k/MRR/nDCG + EM/F1 + kiểm citation
        └── 50_demo_colab.ipynb         # DEMO nhanh (Gradio trên Colab)
</pre>

