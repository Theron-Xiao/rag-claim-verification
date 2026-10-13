# 基于 RAG 的学术论文证据核验系统

端到端的「预处理 → 混合召回 → 重排序 → 大模型核验」事实核验流程，面向「AI / 数据产品 / NLP 应用」方向。

## 问题定义

给定论文中的一条事实陈述，判断其是否被原文证据支持，定位支持证据，并输出判定理由。

## 数据来源

- QASPER 学术论文事实验证数据（文档 + 标注）。

## 方法

- **预处理**：全文按 section 切分，构造 `Title / Section / Content` 富语义文本。
- **向量化与索引**：BGE-M3 1024 维向量化，FAISS 建立向量索引。
- **混合召回**：BM25 与 BGE-M3 同时检索取并集做粗排。
- **重排序**：BGE-Reranker 精排，DeBERTa 返回 Top-1、LLM 返回 Top-K。
- **核验**：DeBERTa 分类（Entailment）或 DeepSeek-V3 输出 JSON（`reasoning` / `label` / `best_evidence_id`）。

## 结果与评估

- 构建 Top-1 检索准确率与严格匹配评估流程。
- 通过错误案例分析定位召回与重排环节的问题。

## 运行

```bash
pip install -r requirements.txt
jupyter notebook notebooks/01_preprocess.ipynb       # 生成文档与 FAISS 索引
jupyter notebook notebooks/02_llm_verification_deepseek.ipynb  # 填 API_KEY / BASE_URL 后运行
```

- 需加载 BGE-M3、bge-reranker-base、BM25 模型。
- DeBEATa 分类版 notebook 未随仓库提供，说明见 `reports/code_notes.pdf`。

## 项目结构

```
rag-claim-verification/
├── notebooks/
│   ├── 01_preprocess.ipynb
│   └── 02_llm_verification_deepseek.ipynb
├── reports/
│   ├── project_report.pdf
│   └── code_notes.pdf
├── requirements.txt
└── README.md
```

## 局限性

- 全量运行约 12–16 小时；LLM 版依赖外部 API。
- DeBEATa 分类实现未包含在当前仓库中。
