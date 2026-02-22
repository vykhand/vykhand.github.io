A while back I released a [Streamlit app](https://azure-di-showcase.streamlit.app/) and a [GitHub repo](https://github.com/vykhand/azure-di-showcase) implementing an Azure Document Intelligence playground that you can run on your machine or self-host.

The need is simple — at Aprova GmbH, we're in the business of AI-assisted data extraction from complex documents. When I have a new document type, I want to quickly run it through different APIs and see what I get.

This weekend I built the same kind of playground for GCP Document AI. Plug in your credentials, pick a processor, upload a document (or use one of 12 built-in samples), and explore the results visually — color-coded bounding boxes, hover tooltips with extracted content and confidence scores, Layout Parser with hierarchical structure and chunking for RAG pipelines. It auto-discovers all Document AI processors in your GCP project and supports 12 types out of the box: OCR, invoices, receipts, bank statements, W-2s, IDs, passports, and more.

As I'm quite rusty in GCP, a quick comparison with Azure revealed a few rough edges:

- Setup is heavier — you need to configure the project, enable the API, create IAM & credentials, then create each processor individually. On Azure I just create a resource group & Foundry, grab the keys, and go.
- Some things are conceptually different. For example, the Layout Parser in GCP doesn't return bounding boxes at all, while other models do. In Azure, that's exactly what the layout extractor is for.
- Not sharing this as critique — just the experience. My GCP friends out there, tell me if I'm doing things completely wrong :)

What's coming next:

- Playground for AWS Textract
- Tool for side-by-side comparison of different APIs

Feel free to use these playgrounds for your demos and PoCs and let me know if you found them useful!

Streamlit app: https://gcp-doc-ai-showcase.streamlit.app/
GitHub repo: https://github.com/vykhand/gcp-doc-ai-showcase
Blog post: https://vykhand.github.io/GCP-Document-AI-Playground-Streamlit/

#GoogleCloud #DocumentAI #Azure #Streamlit #OpenSource #DocumentProcessing #Python