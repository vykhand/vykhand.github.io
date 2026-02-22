# Building a GCP Document AI Playground with Streamlit

**Andrey Vykhodtsev** ([@vykhand](https://github.com/vykhand))

---

## Why another playground?

At [Aprova GmbH](https://aprova.ch), we do AI-assisted data extraction from complex documents. When a new document type lands on my desk, I want to throw it at the API and immediately see what comes back — entities, tables, bounding boxes, confidence scores, everything.

A while back I built an [Azure Document Intelligence playground](https://azure-di-showcase.streamlit.app/) ([repo](https://github.com/vykhand/azure-di-showcase)) for exactly this purpose. This weekend I decided to build the same thing for GCP Document AI.

The result is the **[GCP Document AI Showcase](https://gcp-doc-ai-showcase.streamlit.app/)** — a Streamlit app where you plug in your credentials, pick a processor, drop in a document, and explore everything the API returns, visually, on top of the original pages.

## Demo

<video width="100%" controls>
  <source src="/images/gcp-docai-demo.mp4" type="video/mp4">
</video>

## What the app actually does

You get a single browser window where you can:

- **Auto-discover** all Document AI processors in your GCP project (no hardcoding processor IDs)
- **Upload** any PDF or image, or pick from 12 built-in sample documents so you can demo it without having files handy
- **Process** with any of the 12 supported processor types — OCR, invoices, receipts, bank statements, W-2s, IDs, passports, and more
- **Visually inspect** results with color-coded bounding box overlays — hover over any box to see the extracted content, confidence score, and type-specific details
- **Browse** structured results across tabs: Entities, Tables, Form Fields, Text, Raw JSON
- **Explore** Layout Parser output — hierarchical document structure and document chunks for RAG pipelines
- **Navigate** multi-page documents with zoom controls

The bounding box overlay is probably the most useful part. Each element type gets its own color (blue for text lines, green for tables, orange for form fields, crimson for entities, etc.), and hovering shows a rich tooltip with everything the API returned for that element. This is rendered as an HTML/JS overlay via `streamlit.components.v1.html` — plain Streamlit images don't support mouse hover events, so I had to get a bit creative there.

## The tech stack

I kept it deliberately minimal:

```
Streamlit          → UI framework
GCP Document AI    → Document processing (REST API)
pdf2image + PIL    → PDF rendering and annotation
OAuth2 (SA keys)   → Authentication
```

### Why REST instead of the Python SDK?

The official `google-cloud-documentai` SDK pulls in a heavy dependency tree — `grpcio`, `proto-plus`, `google-auth`, and friends. For a lightweight app that needs to run on Streamlit Community Cloud, I wanted fewer dependencies, faster installs, and full visibility into what goes over the wire.

So the entire GCP client is a single file (`gcp_docai_client.py`) that handles OAuth2 token refresh, processor discovery, and document processing — all via plain HTTPS requests. It's about as simple as a Document AI client can get.

### Project structure

```
app.py                  # Main Streamlit entry point
config.py               # Processor definitions, categories, colors
gcp_docai_client.py     # REST client + DocumentAnalysisResult parser
document_processor.py   # File validation, PDF-to-image, coordinate math
ui_components.py        # Reusable Streamlit UI components
simple_annotator.py     # PIL-based image annotation
```

## A few things I noticed coming from Azure

I'm quite rusty in GCP, so take this with a grain of salt — but a quick comparison revealed some interesting differences.

**Setup is heavier.** On GCP, you need to configure the project, enable the Document AI API, set up IAM & credentials, and then create each processor individually. On Azure, I create a resource group and an AI Foundry resource, grab the keys, and I'm off. Not a dealbreaker, just more steps.

**Some things are conceptually different.** The Layout Parser in GCP doesn't return bounding boxes at all — it gives you hierarchical document structure and chunks, which is great for RAG but means you can't visualize where things are on the page. Other GCP models (OCR, invoice parser, etc.) do return bounding boxes just fine. In Azure, the layout extractor is specifically the model that gives you spatial information. Neither approach is wrong, they just solve different problems.

**Processor auto-discovery is nice.** The `list` endpoint that returns all processors in your project is a good API design choice. The app calls it on startup and dynamically populates the sidebar:

```python
processors = client.list_processors()
# Returns: [{"display_name": "My Invoice Parser", "type": "INVOICE_PROCESSOR", "state": "ENABLED", ...}]
```

If auto-discovery fails (missing permissions, for example), the app falls back to manual processor ID entry with a type-reference dropdown. This way it still works even with minimal IAM setup.

My GCP friends out there — if I'm doing things completely wrong, tell me :)

## Getting started

```bash
# Clone
git clone https://github.com/vykhand/gcp-doc-ai-showcase
cd gcp-doc-ai-showcase

# Install (using uv)
uv sync

# Configure
export GCP_DOCAI_ENDPOINT="https://us-documentai.googleapis.com/v1/projects/YOUR_PROJECT/locations/us"
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"

# Run
uv run streamlit run app.py
```

You'll need:
- A GCP project with the Document AI API enabled
- At least one processor created (Invoice Parser is a good first choice)
- A service account with `roles/documentai.apiUser`

See the [QUICKSTART.md](https://github.com/vykhand/gcp-doc-ai-showcase/blob/main/QUICKSTART.md) for step-by-step GCP setup. The app also ships with 12 sample documents (sourced from Google's public buckets), so you can start exploring immediately.

## What's next

- **AWS Textract playground** — same idea, third cloud
- **Side-by-side comparison tool** — feed the same document to Azure, GCP, and AWS, and compare results

## Try it out

The code is open source under the MIT License:

- **Live app:** [gcp-doc-ai-showcase.streamlit.app](https://gcp-doc-ai-showcase.streamlit.app/)
- **GitHub:** [github.com/vykhand/gcp-doc-ai-showcase](https://github.com/vykhand/gcp-doc-ai-showcase)
- **Azure version:** [azure-di-showcase.streamlit.app](https://azure-di-showcase.streamlit.app/)

Feel free to use these playgrounds for your demos and PoCs. If you find them useful, let me know — and contributions are always welcome.
