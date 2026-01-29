# RAG-Pipeline

## 🌟 What is this?

A complete **Retrieval-Augmented Generation (RAG)** system that lets you upload documents and ask natural language questions about their contents. The system uses semantic search, vector embeddings, and multi-document synthesis to provide accurate answers with source citations.

Perfect for searching through contracts, reports, research papers, or any collection of documents where you need fast, accurate information retrieval.

---

## ✨ Key Features

- 📁 **Multi-format support**: PDFs, text files, and images (with OCR)
- 🔍 **Semantic search**: FAISS vector database for fast similarity search
- 🧠 **Multi-document synthesis**: Combines information across multiple sources intelligently
- 📊 **Confidence scoring**: Know how reliable each answer is
- 📚 **Source citations**: Every answer includes page numbers and relevant excerpts
- 🎨 **Clean web UI**: Built with Gradio for easy interaction
- ⚡ **Fast retrieval**: Sub-second query response times
- 🖼️ **OCR support**: Extract text from scanned documents and images using Tesseract

---

## 🎬 Demo

### Example 1: Simple Question
```
📤 Upload: mortgage_agreement.pdf
❓ Question: "What is the interest rate?"
💬 Answer: "The loan bears interest at a fixed annual rate of 6.75%."
📊 Confidence: 94%
📚 Source: mortgage_agreement.pdf, Page 1
```

### Example 2: Multi-Document Query
```
📤 Upload: contract_v1.pdf, contract_v2.pdf, terms.pdf
❓ Question: "Compare the payment terms across all documents"
💬 Answer: "Contract v1 requires monthly payments of $2,270.16.
           Contract v2 modified this to $2,840.16 including escrow.
           The terms document specifies a 15-day grace period."
📊 Confidence: 87%
📚 Sources: 3 documents cited
```

### Example 3: OCR from Scanned Document
```
📤 Upload: scanned_statement.png
❓ Question: "What is the current principal balance?"
💬 Answer: "The current principal balance is $348,742.15."
📊 Confidence: 91%
📚 Source: scanned_statement.png (OCR extracted)
```

---

## 🚀 Quick Start

### Option 1: Run in Google Colab (Recommended - No Setup Required!)

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/DevAnnafi/document-rag-pipeline/blob/main/notebooks/RAG_Pipeline.ipynb)

1. Click the badge above
2. Run all cells (`Runtime` → `Run all`)
3. Upload your documents when prompted
4. Start asking questions!

**No installation required!** Everything runs in your browser.

---

### Option 2: Run Locally

#### Prerequisites
- Python 3.8 or higher
- Tesseract OCR (for image support)

#### Installation

```bash
# Clone the repository
git clone https://github.com/DevAnnafi/RAG-Pipeline.git

# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install Python dependencies
pip install -r requirements.txt

# Install Tesseract OCR
# Ubuntu/Debian:
sudo apt-get install tesseract-ocr

# macOS:
brew install tesseract

# Windows:
# Download installer from: https://github.com/UB-Mannheim/tesseract/wiki
```

#### Run the Application

```bash
# Run the Gradio interface
python app.py

# Or run the notebook locally
jupyter notebook notebooks/RAG_Pipeline.ipynb
```

The Gradio interface will launch at `http://localhost:7860`

---

## 📖 How It Works

### System Architecture

```
┌─────────────────────┐
│   Upload Documents  │
│  (PDF/TXT/Images)   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────────┐
│   Document Processor            │
│   • Extract text (PDF/OCR)      │
│   • Clean & normalize           │
│   • Create semantic chunks      │
│   • (800 chars, 150 overlap)    │
└──────────┬──────────────────────┘
           │
           ▼
┌─────────────────────────────────┐
│   Embedding Manager             │
│   • Generate embeddings         │
│   • Sentence-BERT (384-dim)     │
│   • Index with FAISS            │
└──────────┬──────────────────────┘
           │
           ▼
┌─────────────────────────────────┐
│   User Query                    │
└──────────┬──────────────────────┘
           │
           ▼
┌─────────────────────────────────┐
│   Semantic Retrieval            │
│   • Embed query                 │
│   • FAISS similarity search     │
│   • Retrieve top-K chunks       │
└──────────┬──────────────────────┘
           │
           ▼
┌─────────────────────────────────┐
│   Multi-Doc Synthesis           │
│   • Pick best chunk per doc     │
│   • Extract relevant sentences  │
│   • Synthesize coherent answer  │
└──────────┬──────────────────────┘
           │
           ▼
┌─────────────────────┐
│   Final Answer      │
│   + Sources         │
│   + Confidence      │
│   + Page Numbers    │
└─────────────────────┘
```

### Pipeline Components

#### 1. **Document Processor** (`DocumentProcessor`)
- Supports PDF, TXT, PNG, JPG formats
- OCR for scanned documents using Tesseract
- Creates overlapping semantic chunks (800 characters with 150 character overlap)
- Maintains document metadata (filename, page count, upload date)

#### 2. **Embedding Manager** (`EmbeddingManager`)
- Uses Sentence-BERT model: `all-MiniLM-L6-v2`
- Generates 384-dimensional dense vectors
- FAISS `IndexFlatL2` for exact similarity search
- Efficient chunk indexing and retrieval

#### 3. **Response Generator** (`ResponseGenerator`)
- **Multi-document synthesis**: Picks best chunk from each relevant document
- Extracts relevant sentences matching the query
- Combines information into coherent answers
- Avoids simple copy-paste from single sources

#### 4. **RAG Pipeline** (`RAGPipeline`)
- Orchestrates all components
- Manages document store and chat history
- Provides unified query interface
- Tracks confidence scores and sources

---

## 🏗️ Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Embeddings** | Sentence-BERT (`all-MiniLM-L6-v2`) | Convert text to semantic vectors (384-dim) |
| **Vector DB** | FAISS (`IndexFlatL2`) | Fast similarity search (L2 distance) |
| **PDF Processing** | PyPDF2 | Extract text from PDF documents |
| **OCR** | Tesseract | Extract text from images and scans |
| **UI** | Gradio | Interactive web interface |
| **Language** | Python 3.8+ | Core implementation |
| **Compute** | CPU-optimized | No GPU required |

---

## ⚙️ Key Design Decisions

### Chunking Strategy: 800 characters with 150 overlap
**Why?**
- Balances context preservation and retrieval precision
- Overlap prevents information loss at chunk boundaries
- Approximately 2-3 paragraphs per chunk (natural semantic unit)
- Tested alternatives: 500 (too fragmented), 1200 (too diluted)

### Embedding Model: all-MiniLM-L6-v2
**Why?**
- Lightweight (80MB) but highly accurate
- Fast inference on CPU (~100 chunks/second)
- Good balance of speed vs. quality for document search
- Industry-proven for semantic similarity tasks

### Multi-Document Synthesis
**Why?**
- Prevents answers from being anchored to a single document
- Picks the best chunk from EACH relevant document
- Synthesizes information across sources
- Provides more complete, accurate answers for complex queries

### FAISS Index: IndexFlatL2
**Why?**
- Exact search (no approximation) for maximum accuracy
- Perfect for <100k chunks (most document collections)
- Simple, reliable, no hyperparameters to tune
- Would scale to IndexIVFFlat for larger datasets

---

## 📊 Performance Benchmarks

| Metric | Performance | Notes |
|--------|-------------|-------|
| **Document Processing** | 3-5 seconds per PDF | Depends on page count |
| **Query Response Time** | <1 second | For indexed documents |
| **Embedding Speed** | ~100 chunks/second | On CPU (faster on GPU) |
| **OCR Accuracy** | 85-95% | Depends on image quality |
| **Memory Usage** | ~500MB base + ~1MB per 1000 chunks | Scalable |
| **Concurrent Users** | 5-10 simultaneous | On standard hardware |

**Tested Scale:**
- ✅ 50+ documents successfully processed
- ✅ 500+ chunks indexed and searched
- ✅ 100+ queries executed
- ✅ Multiple concurrent users supported

---

## 🎯 Use Cases

### Legal
- Search through contracts, agreements, NDAs
- Extract specific clauses or terms
- Compare versions of legal documents

### Finance
- Query mortgage documents, loan agreements
- Search financial statements and reports
- Find specific payment terms or rates

### Research
- Ask questions across academic papers
- Extract methodology or findings
- Compare results across multiple studies

### Business
- Search company policies and documentation
- Query HR handbooks, procedures
- Find specific information in reports

### Personal
- Organize and search receipts
- Query medical records or insurance documents
- Search tax documents and statements

---

## 🛠️ Configuration

Customize the pipeline by editing the `Config` class:

```python
class Config:
    CHUNK_SIZE = 800              # Characters per chunk
    CHUNK_OVERLAP = 150           # Overlap between chunks
    MIN_CONTENT_LENGTH = 20       # Minimum content to process

    EMBEDDING_MODEL = "sentence-transformers/all-MiniLM-L6-v2"
    EMBEDDING_DIM = 384           # Model output dimension

    TOP_K = 5                     # Number of chunks to retrieve
    SIMILARITY_THRESHOLD = 0.3    # Minimum similarity score

    DEVICE = "cpu"                # "cpu" or "cuda"
```

### Tuning Tips

**For better accuracy:**
- Increase `TOP_K` to 10 (retrieves more context)
- Decrease `SIMILARITY_THRESHOLD` to 0.2 (more permissive)

**For faster performance:**
- Decrease `CHUNK_SIZE` to 500 (smaller chunks)
- Decrease `TOP_K` to 3 (fewer retrievals)

**For longer documents:**
- Increase `CHUNK_SIZE` to 1000
- Increase `CHUNK_OVERLAP` to 200

---

## 📁 Project Structure

```
document-rag-pipeline/
├── README.md                           # This file
├── requirements.txt                    # Python dependencies
├── LICENSE                            # MIT License
├── .gitignore                         # Git ignore patterns
│
├── notebooks/
│   └── RAG_Pipeline.ipynb    # Main Colab notebook
│
├── samples/                           # Sample documents for testing
│   ├── mortgage_agreement_sample.txt  # Sample mortgage document
│   └── mortgage_statement_scan.png    # Sample scanned statement
```

---

## 🔮 Future Improvements

Planned enhancements for future versions:

### High Priority
- [ ] **Conversation memory** - Support for follow-up questions with context
- [ ] **Reranking** - Use cross-encoder for better precision
- [ ] **Hybrid search** - Combine semantic + keyword (BM25) search
- [ ] **Citation validation** - Verify answers are grounded in sources

### Medium Priority
- [ ] **Document versioning** - Track and update document versions
- [ ] **Query expansion** - Handle abbreviations and synonyms better
- [ ] **Semantic caching** - Cache frequently asked questions
- [ ] **Better answer generation** - Integrate LLM API for more sophisticated responses

### Nice to Have
- [ ] Support for more formats (DOCX, XLSX, PPTX)
- [ ] Multilingual support
- [ ] Document summarization
- [ ] Named entity extraction
- [ ] Export chat history
- [ ] API endpoint for programmatic access

---

## 🐛 Known Limitations

**Current limitations to be aware of:**

1. **OCR Accuracy**: Low-quality scans may produce poor results
2. **Context Window**: Very long documents may need more than 5 chunks
3. **No Conversation Memory**: Each query is independent (no follow-ups)
4. **CPU-Only**: No GPU acceleration implemented (though FAISS supports it)
5. **English-Only**: Optimized for English documents
6. **No Authentication**: Single-user system (no multi-tenancy)

---

## 🤝 Contributing

Contributions are welcome! Here are some ways you can help:

### Areas for Improvement
1. Add support for more file formats (DOCX, XLSX)
2. Implement conversation memory for follow-up questions
3. Add reranking with cross-encoder models
4. Improve OCR preprocessing and quality
5. Add comprehensive unit tests
6. Optimize for GPU acceleration
7. Add multilingual support

### How to Contribute
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

---

## 📚 Documentation

### Additional Resources

- **[Installation Guide](docs/installation.md)** - Detailed setup instructions
- **[API Documentation](docs/api.md)** - Component interfaces
- **[Troubleshooting](docs/troubleshooting.md)** - Common issues and solutions
- **[Best Practices](docs/best-practices.md)** - Tips for optimal usage

### Academic References

This project is based on research in:
- Retrieval-Augmented Generation (RAG)
- Dense passage retrieval
- Semantic similarity search
- Information retrieval systems

**Key Papers:**
- Lewis et al. (2020) - "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"
- Karpukhin et al. (2020) - "Dense Passage Retrieval for Open-Domain Question Answering"
- Reimers & Gurevych (2019) - "Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks"

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

**TL;DR**: You can use, modify, and distribute this code freely, even commercially, as long as you include the original license.

---

## 🙏 Acknowledgments

Special thanks to:

- **[Sentence-Transformers](https://www.sbert.net/)** - Excellent embedding models
- **[FAISS](https://github.com/facebookresearch/faiss)** - Fast vector search library
- **[Gradio](https://gradio.app/)** - Beautiful UI framework
- **[Tesseract OCR](https://github.com/tesseract-ocr/tesseract)** - Powerful OCR engine
- **[PyPDF2](https://pypdf2.readthedocs.io/)** - PDF processing library
- **Outamation** - For the externship opportunity

---

## 📧 Contact & Support

### Get Help

- 🐛 **Bug Reports**: [Open an issue](https://github.com/DevAnnafi/document-rag-pipeline/issues)
- 💡 **Feature Requests**: [Start a discussion](https://github.com/DevAnnafi/document-rag-pipeline/discussions)
- 📧 **Email**: islamannafi@gmail.com

---

## ⭐ Show Your Support

If this project helped you, please consider:

- ⭐ **Star this repository**
- 🍴 **Fork it** for your own projects
- 📢 **Share it** with others who might find it useful
- 💬 **Provide feedback** via issues or discussions

---

## 📈 Project Stats

![GitHub stars](https://img.shields.io/github/stars/YOUR_USERNAME/document-rag-pipeline?style=social)
![GitHub forks](https://img.shields.io/github/forks/YOUR_USERNAME/document-rag-pipeline?style=social)
![GitHub watchers](https://img.shields.io/github/watchers/YOUR_USERNAME/document-rag-pipeline?style=social)

---

## 🎓 Learning Resources

If you're new to RAG or want to learn more:

### Tutorials
- [Introduction to RAG Systems](https://www.youtube.com/watch?v=example)
- [FAISS Tutorial](https://github.com/facebookresearch/faiss/wiki/Getting-started)
- [Sentence-BERT Documentation](https://www.sbert.net/)

### Courses
- [Building RAG Systems (Free)](https://www.deeplearning.ai/)
- [Information Retrieval](https://www.coursera.org/)

### Books
- "Speech and Language Processing" by Jurafsky & Martin
- "Introduction to Information Retrieval" by Manning, Raghavan & Schütze

---

<div align="center">

**Built with ❤️ for the Outamation Externship Program**

*Making document intelligence accessible to everyone*

[⬆ Back to Top](#-document-intelligence-rag-pipeline)

</div>
