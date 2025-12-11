I've been working on a problem that anyone dealing with enterprise documents knows: you have hundreds of technical PDFs, Excel specifications, and emails - and you need to systematically extract answers to structured question sets while proving those answers are grounded in actual source material.

Today I'm open-sourcing **PrismRAG**, a document intelligence platform built on Azure AI.

**The real problem this solves:**

Organizations often need to answer the same set of domain-specific questions across large document collections that are inherently unstructured. Think bid analysis, compliance reviews, technical due diligence, or specification extraction. Doing this manually doesn't scale. Doing it with basic RAG produces answers you can't trust.

PrismRAG approaches this differently.

**How the extraction actually works:**

Documents go through hybrid extraction. For PDFs, PyMuPDF4LLM handles text and tables locally - it's fast, free, and preserves structure. But pages containing embedded images, engineering diagrams, or schematics get validated by GPT-5 Vision. The system automatically detects which pages need vision processing by analyzing drawing complexity and text density. Repeated elements like headers and logos are filtered to avoid wasting API calls.

Excel files are extracted with openpyxl (including hidden sheets and formulas), then restructured by an AI agent to optimize for search retrieval. Emails get parsed with extract-msg, with AI categorizing email types and extracting action items, deadlines, and stakeholder references.

Everything outputs to structured markdown that preserves the original document hierarchy. You can apply your own custom retrieval instructions at runtime.

**How agentic retrieval works:**

Once documents are chunked and embedded, PrismRAG creates Azure AI Search Knowledge Agents. These aren't simple vector lookups.

When you ask a question, the Knowledge Agent:
1. Plans the query - breaking complex questions into focused subqueries
2. Executes searches in parallel across the index
3. Synthesizes answers from multiple retrieved chunks
4. Returns citations with document names, page numbers, and relevance scores

The agent operates under strict grounding instructions: only use explicitly stated document content, distinguish between "not found" and "explicitly excluded," mark any assumptions, cite everything. This prevents the hallucination problem that makes standard RAG unreliable for serious document work.

**How workflow automation works:**

Define your question sets in JSON - grouped into sections with templates and per-question instructions. Run them against your knowledge base. Each answer is saved with citations and raw response data.

Then the evaluation system kicks in. Using Azure AI Evaluation SDK, every answer is automatically scored on:
- **Relevance**: Does it address the question?
- **Coherence**: Is it logically consistent?
- **Fluency**: Is the language natural?
- **Groundedness**: Is it supported by the retrieved context?

No ground truth required. You get quantified confidence in your answers, not just the answers themselves.

**What you get:**
- FastAPI backend + Vue 3 frontend
- Full IaC with Bicep - deploy with `azd up`
- Docker Compose for local development (uses Azurite, no Azure account needed to experiment)
- Project isolation - each project gets its own index, knowledge source, and agent
- Rollback support at each pipeline stage

**What this is and isn't:**

This is a reference implementation demonstrating how to combine Azure AI services for document intelligence workflows. It showcases production patterns but isn't production-hardened out of the box - authentication is basic, and you'll want to add enterprise security features before deploying for real workloads. The architecture documentation and productionizing guide explain what's needed.

If you're building systems that need to extract structured knowledge from document collections - and need to prove that knowledge is actually grounded in sources - this might be useful.

Code is on GitHub under MIT license.

GitHub: [link to repo]

#OpenSource #Azure #DocumentAI #RAG #EnterpriseAI
