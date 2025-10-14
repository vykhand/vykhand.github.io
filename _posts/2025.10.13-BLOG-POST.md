# Building a Better Azure Document Intelligence Playground: AI-Assisted Development in Production

## Context: Document Intelligence in Swiss Insurance

At our firm, we develop LLM-based document data extraction solutions for the Swiss insurance market. Azure Document Intelligence is a critical component of our stack, handling structured extraction from diverse document types: claims forms, policy documents, medical reports, and correspondence across multiple languages (German, French, Italian).

The challenge with document intelligence systems isn't the API—Azure's REST interface is well-designed and comprehensive. The challenge is **rapid iteration during development**. We need to test various models against different document types, validate extraction accuracy, debug edge cases, and demonstrate capabilities to stakeholders—often within the same day.

---

## The Tooling Gap

For experimentation and testing, we had two options:

**Option 1: Azure Document Intelligence Studio**
- Microsoft's official web-based playground
- Excellent for basic testing, but limited customization
- Lacks workflow integration and automation capabilities
- No version control or programmatic access

**Option 2: Form Recognizer Toolkit**
- Open-source React application: [microsoft/Form-Recognizer-Toolkit](https://github.com/microsoft/Form-Recognizer-Toolkit)
- Full-featured but architecturally complex
- Heavy TypeScript/React stack with multiple dependencies
- Customization requires navigating React components, state management, and build configurations
- Setup time measured in hours, not minutes

Neither option aligned with our needs: a lightweight, customizable testing environment that could be deployed quickly, modified easily, and integrated into development workflows.

The solution: build a purpose-built alternative using Streamlit and Python.

---

## The AI-Assisted Development Experiment

This project became an opportunity to evaluate modern AI coding assistants in a real-world scenario. I decided to build the same application using three different tools in parallel:

1. **Gemini CLI** (free version, 2.5 Flash model)
2. **OpenAI Codex CLI** (GPT-4.1 via Plus subscription)
3. **Claude Code CLI** (Claude 3.7 Sonnet—this was before the 4.5 release)

The methodology was pure "vibe-coding": natural language prompts, no manual code edits, iterative refinement through conversation. The AI writes the specification, generates the code, debugs issues, and produces documentation.

---

## The Race: Three AIs, One Task

### Initial Prompt

All three systems received the same instruction:

> " I am writing a self-contained demo to showcase all the capabilities of azure document intelligence 4.0. build me a stream lit app. I envision it like this  1) in the sidebar, there will be a
  dropdown allowing to choose different Azure DI models (layout, general, receipts etc.). Later, we will add auto mode (choosing model automatically with LLM). When we choose a model in the sidebar,
   below will appear ui controls for all the parameters of that model available in the API. on the right of the sidebar, there will be an interface similar to what is available in Azure DI
  studio(see img.png). It will display the anotated document (all formats supported by azure DI) and also parsing results in three forms (nicely formatted, markdown if available from response, and
  json with ability to collapse. Azure Document Intelligence must be used via REST API (no python SDK). Start by writing a detailed specification and put it into SPEC.md
"

### The Divergence

All three AI assistants followed the instruction to write SPEC.md first. However, the quality and usability of the specifications varied significantly:

**Gemini 2.5 Flash:**
- Generated a basic specification document
- Implementation diverged from the spec during multi-file development
- Context loss during API integration discussions required repeated clarification
- Model configuration generation was incomplete

**OpenAI GPT-4.1:**
- Produced a reasonable specification
- Initial code structure was clean
- Implementation struggled with async polling patterns specific to Azure DI
- Lost momentum during multi-file coordination

**Claude 3.7 Sonnet:**
- Generated the most comprehensive [SPEC.md](https://github.com/vykhand/azure-di-showcase/blob/main/SPEC.md) (383 lines)
- Consistently referenced the spec during implementation
- Maintained architectural coherence across all modules
- UI implementation matched the specification from the first iteration

### The Specification Quality Gap

Claude's specification document included details that proved critical for implementation success:

- Complete definitions for all 20+ Azure DI models with feature mappings
- API parameter configurations with validation rules and UI widget types
- ASCII art wireframes showing exact UI layout
- REST API patterns including operation polling and error handling
- Credential management strategies

The key difference wasn't just writing a specification—it was **using the specification as a consistent reference throughout development**. While all three AIs generated specs, Claude maintained specification fidelity during implementation, preventing architectural drift.

After multiple attempts with Gemini and GPT-4.1 that resulted in incomplete or inconsistent implementations, I focused exclusively on Claude's output.

---

## The Build: 4-5 Hours to Production

### Phase 1: Architecture and Core (Hours 0-2)

Claude generated the foundational structure:

```
azure-di-showcase/
├── app.py                 # Main Streamlit application
├── config.py             # Models and parameters configuration
├── azure_di_client.py    # REST API client with async polling
├── ui_components.py      # Reusable UI components
├── document_processor.py # Document handling utilities
├── logging_config.py     # Centralized logging
└── SPEC.md              # Technical specification
```

Key achievements:
- Async REST client with proper operation polling
- All 20+ models configured with correct parameters
- Dynamic UI generation based on selected model
- Type hints and docstrings throughout

### Phase 2: Features and Integration (Hours 2-4)

Implemented complete feature set:
- Three upload methods: file upload, URL input, sample documents
- Document viewer with PDF page navigation
- Multi-tab results display: Fields, Markdown, Raw JSON
- Annotated document visualization (bounding boxes)
- Configurable logging (INFO/DEBUG/ERROR levels)
- Comprehensive error handling

### Phase 3: Deployment and Documentation (Hour 4-5)

Prepared for Streamlit Community Cloud:
- Multiple credential sources: Streamlit secrets, environment variables, UI input
- Created deployment guide (STREAMLIT_DEPLOYMENT.md)
- Added MIT license with attribution
- Fixed system dependencies (poppler-utils for PDF processing)
- Generated QUICKSTART.md, README.md, LOGGING.md, CODE_QUALITY.md

**Total development time: 4-5 hours, zero manual code edits.**

---

## The Result

**Live Application:** [azure-di-showcase.streamlit.app](https://azure-di-showcase.streamlit.app/)
**Source Code:** [github.com/vykhand/azure-di-showcase](https://github.com/vykhand/azure-di-showcase)

### Capabilities

**Supported Models (20+):**
- Core: Read OCR, Layout Analysis
- Business: Receipts, Invoices, Contracts, Business Cards, ID Documents
- Financial: Bank checks, statements, pay stubs, credit cards
- Government: Tax forms (W-2, W-4, 1040, 1098, 1099, 1095), mortgage documents
- Healthcare: US Health Insurance Cards

**Technical Features:**
- Async REST API integration with operation polling
- Dynamic parameter configuration based on model selection
- Interactive document viewer with bounding box annotations
- Multi-format output: structured fields, markdown, raw JSON
- Configurable logging with DEBUG mode for API introspection
- Multiple credential sources: Streamlit secrets, environment variables, UI input
- System dependency management for cloud deployment

**Use Cases:**
- Rapid model testing during development
- Document extraction accuracy validation
- Stakeholder demonstrations
- API behavior debugging
- Training data generation
- Integration prototyping

---

## What Made the Difference

### Specification Fidelity

The winning factor was maintaining consistency between specification and implementation. Claude generated a detailed SPEC.md and then **actually followed it** during code generation. This prevented architectural drift across the multi-file Python project.

### Context Retention

Complex projects require maintaining awareness across multiple files and conversations. The Azure DI integration required:
- Model-specific parameter configurations
- REST API polling patterns with operation IDs
- Async operation handling
- Multi-format document processing
- Cross-file dependency management

Claude maintained this context throughout the 4-5 hour development session, while the other assistants required re-explanation of previously established patterns.

### UI Implementation Quality

The most visible difference was in the initial UI implementation. Claude's first iteration produced a functional interface that closely matched the Azure DI Studio reference, with:
- Proper sidebar layout with dynamic controls
- Document viewer with annotation support
- Three-tab results display (Fields, Markdown, JSON)
- Working file upload and URL input

The other assistants required multiple iterations to achieve similar layouts.

### Autonomous Problem-Solving

During deployment preparation, Claude proactively:
- Implemented multiple credential sources (secrets, env vars, UI input)
- Created comprehensive deployment documentation
- Identified system dependencies (poppler-utils) before deployment
- Generated troubleshooting guides

These weren't explicitly requested but emerged naturally from understanding the deployment context.

---

## Practical Implications for Development Workflows

### When AI-Assisted Development Works

This experiment demonstrates that current-generation AI coding assistants (Claude 3.7+) are production-ready for:

**Specification-First Projects:**
- Well-defined requirements
- Standard architectural patterns
- Clear API contracts
- Documented third-party services

**Rapid Prototyping:**
- Internal tooling
- Testing interfaces
- Demo applications
- Proof-of-concept implementations

**Documentation-Heavy Work:**
- Deployment guides
- API references
- Technical specifications
- Troubleshooting documentation

### When Human Oversight Remains Critical

AI-assisted development still requires human judgment for:
- Security review (credential handling, data validation)
- Performance optimization (caching strategies, async patterns)
- Business logic validation (domain-specific rules)
- Production monitoring (observability, error tracking)

The tool writes production-ready code, but production deployment requires human verification.

---

## Future Experiments: Claude 4.5 vs GPT-5

With the recent release of Claude 4.5 Sonnet (February 2025) and the upcoming GPT-5, I plan to repeat this experiment with a different task. Areas of interest:

**Complexity Factors:**
- Multi-service orchestration (Azure + other APIs)
- State management in distributed systems
- Real-time data processing
- Custom training workflows

**Evaluation Criteria:**
- Code quality and maintainability
- Error handling comprehensiveness
- Documentation accuracy
- Deployment success rate
- Debug efficiency

Initial impressions suggest Claude 4.5's improvements focus on extended context and reasoning depth. GPT-5's capabilities remain to be evaluated at release.

---

## Recommendations for Azure Document Intelligence Users

If you're working with Azure Document Intelligence and find the existing tooling insufficient:

**For Development Teams:**
- Fork [azure-di-showcase](https://github.com/vykhand/azure-di-showcase) as a starting point
- Customize for your document types and workflows
- Extend with custom models if needed
- Deploy to internal infrastructure or Streamlit Cloud

**For Evaluation/Testing:**
- Use the [live demo](https://azure-di-showcase.streamlit.app/) directly
- Test your documents against multiple models
- Compare extraction accuracy across model versions
- Export results for further processing

**For Integration:**
- Reference `azure_di_client.py` for REST API patterns
- Adapt async polling logic for your applications
- Use `config.py` as a reference for model parameters
- Leverage logging patterns for debugging

The MIT license permits commercial use with attribution.

---

## Conclusion

This experiment demonstrates that AI-assisted development has reached a maturity level suitable for production tooling when:

1. **Requirements are well-defined**: Clear scope and success criteria
2. **Specification precedes implementation**: Architecture documented before code generation
3. **Tasks match AI strengths**: Standard patterns, documented APIs, multi-file projects
4. **Human oversight is maintained**: Security review, production validation

The comparative evaluation revealed that specification fidelity and context retention are critical differentiators for complex multi-file projects. While all three AI assistants could generate code, maintaining architectural coherence across a 4-5 hour development session separated successful implementations from incomplete ones.

For teams working with Azure Document Intelligence in LLM-based extraction workflows, this project demonstrates that custom tooling can be developed rapidly without sacrificing code quality or maintainability. The key is selecting AI tools that maintain context and follow architectural specifications consistently.

---

## Resources

**Project Links:**
- Live Application: https://azure-di-showcase.streamlit.app/
- GitHub Repository: https://github.com/vykhand/azure-di-showcase
- Technical Specification: [SPEC.md](https://github.com/vykhand/azure-di-showcase/blob/main/SPEC.md)
- Deployment Guide: [STREAMLIT_DEPLOYMENT.md](https://github.com/vykhand/azure-di-showcase/blob/main/STREAMLIT_DEPLOYMENT.md)

**Azure Documentation:**
- [Document Intelligence Overview](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/)
- [REST API Reference](https://learn.microsoft.com/en-us/rest/api/aiservices/document-models)
- [Prebuilt Models](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-model-overview)

**AI Tools Used:**
- Claude Code (Anthropic): https://claude.ai/code
- Gemini CLI (Google): https://ai.google.dev/
- OpenAI API (OpenAI): https://platform.openai.com/

---

**License:** MIT with attribution requirement
**Author:** Andrey Vykhodtsev
**Tags:** #AzureAI #DocumentIntelligence #LLM #MachineLearning #Python #Streamlit #AIAssistedDevelopment #Claude #InsurTech
