# 📚 Student Library

**Student Library** is a free-tier-friendly Multi-Tool AI Assistant built with:

- **Streamlit** — web interface
- **Google Colab** — development environment
- **Groq API** — primary LLM backend
- **Gemini API** — optional alternative backend
- **FAISS + TF-IDF** — local document retrieval
- **PDF/DOCX/PPTX/TXT parsers** — document extraction

It contains three separate AI modes:

1. 📖 Study Assistant
2. 💼 Job Analyzer
3. 🔬 Research Assistant

---

## 1. Project files

Keep these three files in the same folder:

```text
student-library/
├── app.py
├── requirements.txt
└── README.md
```

---

## 2. What the application does

### 📖 Study Assistant

Upload:

- PDF
- DOCX
- PPTX
- TXT

Then ask for:

- Document-grounded questions
- Quick summaries
- Standard summaries
- Deep summaries
- MCQs
- True/False questions
- Short-answer questions
- Fill-in-the-blank questions
- Flashcards
- Glossary extraction
- Study plans
- Alternative explanations when you are still confused

The assistant is instructed to distinguish uploaded source material from the user's request and avoid fabricating source facts.

### 💼 Job Analyzer

Upload:

- Resume
- One or more job descriptions

It analyzes:

- Technical skills
- Soft skills
- Experience
- Education
- Certifications
- Required qualifications
- Nice-to-have qualifications
- Keyword/ATS gaps

It produces three separate scores:

1. Technical Skills Match
2. Experience Match
3. Keyword/ATS Match

It does **not** merge these into one overall score.

You can also request:

- ATS formatting analysis
- Tailored cover letter
- Mock interview questions
- Sample interview answers
- Multi-job comparison
- Skill-gap learning plan

### 🔬 Research Assistant

Upload one or more:

- Research PDFs
- DOCX papers
- TXT papers

Ask for:

- Paper Q&A
- Methodology explanations
- Summaries
- Citation generation
- APA 7
- MLA 9
- IEEE
- Chicago
- Harvard
- Paper comparison
- Literature-review drafting
- Research-gap analysis
- BibTeX/EndNote-style metadata

The assistant is instructed never to invent DOI, page numbers, publisher details, or other missing citation metadata.

---

# 3. API keys

The application supports **either Groq or Gemini**.

You do not need to put an API key into the source code.

## Groq

Set:

```text
GROQ_API_KEY
```

## Gemini

Set:

```text
GEMINI_API_KEY
```

The Streamlit application reads these from environment variables or Streamlit secrets.

### Important

A free-tier API key is still an API key. "Free" means you are using the provider's available free quota; it does not mean the API works without credentials.

Do not place your real API key directly inside `app.py`.

---

# 4. Google Colab setup

Open a new Google Colab notebook.

Upload:

```text
app.py
requirements.txt
README.md
```

Install the dependencies:

```bash
!pip install -r requirements.txt
```

Set your key for the current Colab runtime.

### Groq

```python
import os
os.environ["GROQ_API_KEY"] = "YOUR_GROQ_KEY"
```

### Gemini

```python
import os
os.environ["GEMINI_API_KEY"] = "YOUR_GEMINI_KEY"
```

Only set the provider you intend to use, or set both if you want both options.

---

# 5. Run Streamlit in Colab

Start the application:

```bash
!streamlit run app.py --server.port 8501 &
```

Then expose port 8501 with a tunnel.

For example, if you use a tunneling package available in your environment:

```bash
!pip install pyngrok
```

Then:

```python
from pyngrok import ngrok

public_url = ngrok.connect(8501)
print(public_url)
```

If your tunnel provider requires authentication, follow that provider's current setup instructions.

The public URL is temporary when the Colab runtime is temporary.

---

# 6. Recommended initial architecture

```text
                     ┌─────────────────────┐
                     │   Student Library   │
                     │     Streamlit UI    │
                     └──────────┬──────────┘
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
              ▼                 ▼                 ▼
        Study Assistant    Job Analyzer     Research Assistant
              │                 │                 │
              └─────────────────┼─────────────────┘
                                ▼
                     Document Extraction
                     PDF / DOCX / PPTX / TXT
                                │
                                ▼
                     Local Chunking + FAISS
                                │
                                ▼
                         Context Retrieval
                                │
                     ┌──────────┴──────────┐
                     │                     │
                     ▼                     ▼
                   Groq                  Gemini
```

---

# 7. Why FAISS is included

The application extracts text first.

It then:

1. Splits text into chunks.
2. Converts chunks to TF-IDF vectors.
3. Stores vectors in a local FAISS index.
4. Searches for chunks relevant to the user's request.
5. Sends the selected context to the LLM.

This is a lightweight retrieval-augmented generation (RAG) approach.

The initial implementation intentionally uses local TF-IDF vectors rather than a paid embedding API.

---

# 8. Free-tier development approach

For an initial university/FYP prototype, the main components can be:

| Component | Initial approach |
|---|---|
| UI | Streamlit |
| Development | Google Colab |
| LLM | Groq or Gemini |
| Retrieval | FAISS + TF-IDF |
| PDF extraction | pdfplumber |
| DOCX extraction | python-docx |
| PPTX extraction | python-pptx |
| TXT extraction | Python |
| PDF export | ReportLab |
| Word export | python-docx |
| Storage | Session memory |
| Database | Not required initially |

This avoids requiring a paid database or paid hosting for the first prototype.

---

# 9. API key security

For local/Colab development:

```python
import os
os.environ["GROQ_API_KEY"] = "..."
```

For a deployed Streamlit application, prefer Streamlit secrets or environment variables.

Example Streamlit secrets:

```toml
GROQ_API_KEY = "your-key"
GEMINI_API_KEY = "your-key"
```

Do **not** commit this file to GitHub.

Do not write:

```python
client = Groq(api_key="my-real-secret-key")
```

inside public source code.

---

# 10. Important limitation of the first version

This version is designed as a strong MVP rather than a complete production SaaS platform.

Some advanced features are represented through specialized prompts rather than separate database-backed subsystems.

For example:

- Flashcards are generated through Study Assistant requests.
- Study plans are generated through Study Assistant requests.
- Cover letters are generated through Job Analyzer requests.
- Mock interviews are generated through Job Analyzer requests.
- Literature-review drafts are generated through Research Assistant requests.

A later production version can turn each into dedicated UI workflows with structured JSON.

---

# 11. Notion export

Notion export is intentionally not hard-coded into the first MVP because a real Notion integration requires:

- Notion integration credentials
- Page/database permissions
- A destination page/database
- API calls to the Notion service

You can add it later without changing the three-mode architecture.

---

# 12. Live web research

The initial version does not automatically browse the internet.

This is intentional because the MVP should remain simple and free-tier friendly.

For a production Research Assistant, add a separate live-search layer for:

- Related papers
- Current courses
- Certifications
- Current research
- Literature discovery

When adding live search, keep web results clearly separated from uploaded-paper evidence.

---

# 13. Testing checklist

## Study Assistant

Test:

- Upload one PDF.
- Ask a question whose answer exists in the PDF.
- Ask a question whose answer does not exist.
- Request a quick summary.
- Request a deep summary.
- Generate 5 MCQs.
- Upload a DOCX.
- Upload a PPTX.
- Upload a TXT file.

Expected behavior:

- No fabricated source facts.
- No invented page numbers.
- Source-grounded answers.

## Job Analyzer

Test:

- Upload a resume.
- Upload one JD.
- Check three separate percentages.
- Check matching skills.
- Check missing skills.
- Check keyword gaps.
- Ask for 2–3 rewritten bullets.

Expected behavior:

- No invented experience.
- No invented metrics.
- No invented certifications.

## Research Assistant

Test:

- Upload one paper.
- Ask about methodology.
- Generate APA 7 citation.
- Upload two papers.
- Compare methodology and findings.
- Ask for research gaps.

Expected behavior:

- Explicit paper claims remain separate from synthesis.
- Missing citation metadata is not fabricated.
- Contradictions are flagged.

---

# 14. Troubleshooting

## "GROQ_API_KEY is missing"

Set:

```python
import os
os.environ["GROQ_API_KEY"] = "YOUR_KEY"
```

Then rerun the Streamlit process.

## "GEMINI_API_KEY is missing"

Set:

```python
import os
os.environ["GEMINI_API_KEY"] = "YOUR_KEY"
```

## PDF extraction is empty

Some PDFs are scanned images instead of text PDFs.

This MVP uses text extraction and does not include OCR.

For scanned PDFs, add OCR later.

## FAISS installation fails

Try:

```bash
!pip install -U faiss-cpu
```

Then restart the Colab runtime if necessary.

## Streamlit page does not open

Check:

```bash
!streamlit run app.py --server.port 8501
```

Make sure the tunnel points to:

```text
localhost:8501
```

---

# 15. Recommended development phases

### Phase 1 — MVP

Build and test:

- Three explicit modes
- File extraction
- Groq/Gemini provider switch
- Local FAISS retrieval
- Study Q&A
- Study summaries
- Study quizzes
- Resume/JD comparison
- Research Q&A
- Citation generation
- Paper comparison
- PDF/Word download
- Session history

### Phase 2 — Structured features

Add:

- JSON schemas
- Interactive quiz controls
- Flashcard UI
- Leitner boxes
- Adaptive quiz scoring
- ATS checker UI
- Cover-letter workflow
- Mock interview workflow
- BibTeX export
- EndNote export

### Phase 3 — Advanced research

Add:

- Live web search
- Related-paper finder
- Current course/certification search
- Literature discovery
- Research-gap engine
- Multi-source evidence tracking

### Phase 4 — Production

Add:

- Authentication
- Persistent database
- User accounts
- Cloud storage
- Rate limiting
- Background jobs
- Monitoring
- Production hosting
- Stronger moderation
- Document deletion controls
- Audit logs

---

# 16. Final development principle

Keep the three modes separate.

Do not turn Student Library into one generic chatbot.

The core flow should remain:

```text
User
  ↓
Select Mode
  ↓
Upload Source
  ↓
Extract Text
  ↓
Chunk + Retrieve
  ↓
Mode-Specific System Prompt
  ↓
Groq / Gemini
  ↓
Grounded Response
  ↓
Export / History / Feedback
```

Accuracy, source grounding, anti-hallucination behavior, usability, privacy, structured outputs, and reliable document processing should remain the main priorities.
