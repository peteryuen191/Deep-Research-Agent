---
name: deep-science-writer
description: "End-to-end scientific research pipeline combining Exa Search, Playwright, deep-research, text-humanization, and iterative Remi peer review."
---

# Deep Science Writer (End-to-End Pipeline)

This skill orchestrates a multi-stage pipeline for scientific research, combining neural search, browser automation, academic synthesis, and humanized writing. It merges the capabilities of Exa Search, Playwright, Google Science Skills, Deep Research, and Text Humanization into one cohesive workflow.

## 🎯 Trigger Conditions
- User asks to write a scientific article, literature review, or deep-dive research report.
- User requests comprehensive research using web searching and scraping (Exa + Playwright).
- The output requires rigorous academic citations (APA 7th) but a highly natural, human-like reading flow.

## 📋 Workflow Steps

### Phase 0: Asset Loading & Research Plan Discussion
1. Incorporate academic rigor from `nature-skills`, structured literature parsing from `academic-research-skills`, and cross-validation techniques from `superpowers`. Keep their principles active in context.
2. **Mandatory Pre-Flight Discussion:** Before launching any web searches or fetching any papers, formulate a preliminary research plan (proposed keywords, target databases, expected article structure, and goals).
3. Present this blueprint to the user to discuss the research direction.
   - Focus on delegating the heavy lifting (scraping, multi-page data extraction) to parallel subagents via `delegate_task` to protect the main context window.
4. **Halt & Wait:** You MUST wait for the user's "Explicit Approval" before proceeding to Phase 1.

### Phase 0.5: Background Sourcing & Full-Text Deep Reading
1. **Background Process Sourcing:** Do NOT use main thread or short-lived subagents for extraction. You MUST write a local script to fetch exactly 30 high-impact papers (Q1/Q2) via APIs.
2. **Full-Text Reading:** The script MUST download or scrape the *entire full text* of these 30 papers (not just abstracts).
3. **Long-Running Execution:** Execute the script using `terminal(background=true, notify_on_complete=true)`. This prevents timeouts and saves tokens while the script runs autonomously. Wait for the background notification before proceeding.
4. **Synthesis:** Once the background task finishes, read the locally saved full texts to generate the final Synthesis Reports.
4. **Subagent C (Master Synthesizer):** After A and B complete, you MUST spawn a third subagent, **Subagent C**. Pass both the Scopus and Open Science synthesis reports (which contain ~60 cited papers combined) to Subagent C. Its sole goal is to merge, critically analyze, and curate a final master synthesis that directly addresses the overarching research direction, explicitly selecting and citing at least 20 best papers for the final manuscript.

### Phase 1: Neural & Academic Discovery (`exa-search` + `scopus-mcp` + `google-science-skills`)
1. **Journal Quality Filter**: ONLY include Q1 and Q2 papers. If a Q3 paper provides crucial evidence, you MUST explicitly mark it in the text/table with `[Q3]`. STRICTLY EXCLUDE any Q4 papers and ANY papers published by MDPI.
2. **Query Formulation**: Break down the user's topic into 3-5 distinct semantic search queries.
3. **Proactive Scopus API Search (High Priority)**: Actively deploy the `scopus-mcp` tools (`search_scopus` and `get_abstract_details`) to query the Elsevier Scopus database directly. This is the primary and most authoritative source. Bypass web UI scraping entirely by using these direct API tools.
4. **Neural Search**: Call the **Exa Search MCP** to supplement with contextually relevant articles, technical blogs, and open-access papers.
4. **Academic Databases**: Call relevant `google-science-skills` (e.g., PubMed, Semantic Scholar) to pull additional peer-reviewed metadata.
4. **Exhaustive Mapping (User Rule)**: Do NOT sample (e.g., just looking at 5 out of 20). Process the *exhaustive* set of relevant findings into a structured Markdown table: `| Title | Authors/Year | Key Finding | URL/DOI |`.

### Phase 2: Deep Extraction (`cloakbrowser` + `deep-research`)
1. **Dynamic Scraping**: For high-value sources that require JavaScript rendering, interaction, or are heavily dynamically loaded, use the **CloakBrowser** tool/CLI to navigate, bypass anti-bot protections, wait for specific DOM selectors, and extract the full text. Do NOT use standard Playwright for this, as CloakBrowser is the primary scraper.
2. **Synthesis**: Cross-reference extracted findings across multiple sources to validate claims and identify consensus vs. controversy. 

### Phase 3: Structural Drafting (`article-writing`)
1. Outline the article based on `article-writing` principles: strong hook, logical progression, evidence-backed claims, and clear logical headings.
2. Integrate data explicitly. Use block equations or Mermaid diagrams if explaining complex systems or workflows.
3. Format all inline citations and the final reference list strictly in **APA 7th** format. When generating via `python-docx`, programmatically implement APA hanging indents (`p.paragraph_format.first_line_indent = Inches(-0.5)` and `p.paragraph_format.left_indent = Inches(0.5)`) and ensure journal/book titles and volume numbers are properly italicized.

### Phase 4: Humanization & Polish (`text-humanizer`)
1. Review the generated draft and ruthlessly strip AI-isms, applying the strict **"No Fluff"** style profile.
2. **Banned AI Vocabulary**: "delve", "tapestry", "in conclusion", "crucial", "testament", "realm", "fosters", "underscores", "moreover".
3. **Style Adjustments**: 
   - Use varied sentence lengths (short, punchy sentences mixed with complex ones).
   - Prefer active voice over passive voice.
   - Remove introductory meta-commentary ("Here is the article...").
   - Ensure the prose flows like a subject-matter expert speaking directly to a peer.

### Phase 4.5: Anti-Hallucination & Evidence Verification
**Strict Requirement:** This phase MUST be completed before Remi (Phase 5) is allowed to review the manuscript.
1. **Extraction**: Scan the draft and extract every single in-text citation (e.g., Smith, 2024) and its corresponding Reference List entry.
2. **DOI/URL Liveness Test**: Use the `terminal` tool (via `curl -I`), Exa Search MCP, or a Python script (e.g., `requests.get` / `urllib`) to ping every DOI or URL in the reference list. For bulk or programmatic URL resolution when links are missing, execute `scripts/verify_urls.py` (uses `ddgs` and `requests`).
   - If a DOI/URL is dead (404) or fake: **Delete the citation and rewrite the affected claim**, or pause to find the correct, live DOI.
3. **Claim Grounding Check**: Cross-reference the specific claims made in the draft against the raw data/abstracts collected in Phases 1 & 2. 
   - **Strict Literalism (就事論事)**: Never over-extend findings. If a claim overstates the actual findings, down-modulate it. 
   - **No Concept Stitching**: Do not stitch a macro/global finding (e.g., "planetary boundaries") with a localized context (e.g., "European summer extremes") in the same citation unless the source explicitly connects them. Stick strictly to the literal facts.
4. **Pass Condition**: Zero dead links, zero fake DOIs, and zero ungrounded claims. Only then proceed to Phase 5.

### Phase 5: Peer Review & Iteration (`remi`)
1. Load the `remi` skill (`skill_view(name='remi')`) to act as a strict Nature/Science-level peer reviewer.
2. Submit the Phase 4 draft to Remi for a rigorous evaluation against evidence backing, APA 7th formatting, logical flow, and academic tone.
3. **Iteration Loop**: Analyze Remi's critique. If there are any flaws, fluff, or stylistic complaints, apply the fixes and rewrite the draft.
4. Repeat the Remi review process until Remi approves the manuscript with zero critical concerns.

### Phase 6: Data Visualization & Final Document Generation (.docx)
1. **Infographic & Statistical Plotting**: 
   - For highly engaging, data-driven storytelling and statistical plots, you MUST utilize the **AntV Infographic** (`@antv/infographic`) framework. 
   - Write a short Node.js/HTML script that uses the AntV declarative infographic syntax to render the key research takeaways or statistics.
   - Use `playwright-mcp` or a local headless browser script to take a high-quality screenshot (`.png`) of the rendered AntV infographic. Save it to `assets/`.
   - You may still use `mermaid.ink` for simple flowcharts, but all major data visualizations should leverage the AntV infographic capabilities for professional aesthetics.
2. **Document Compilation (`docx` or `python-docx`)**: You MUST NOT just output Markdown as the final product. Write a Node.js script using the `docx` library (`npm install docx`) or a Python script using `python-docx` to programmatically build the final Word document. Node.js is heavily preferred if Python environments (like MSYS bash on Windows) lack `pip` or are unstable. (See `templates/generate_docx.js` for a Node.js starter).
3. **Embed Assets**: Insert the generated AntV Infographic images and any Mermaid charts into the `.docx` file at the appropriate logical sections. Ensure formatting aligns with APA 7th standards (e.g., proper figure captions).
4. **Final Delivery**: Save the generated `.docx` file directly to the `D:\` drive (e.g., `D:\Research_Report.docx`). Deliver this absolute D:\ path to the user.

### Phase 7: Knowledge Base & NotebookLM Integration
1. **Obsidian Wiki Update**: Synthesize the core findings, literature insights, and strategic takeaways. Write or append this synthesis directly into the user's Obsidian Vault (`C:\Users\User\Documents\Obsidian Vault\Hermes\`) to maintain an ongoing, centralized knowledge base.
2. **NotebookLM Source Ingestion**: Utilize the `notebooklm` MCP tools to create a dedicated notebook for this research project. You MUST explicitly upload **every single cited reference** as an **individual source** into NotebookLM (do not just upload one compiled document). Upload the raw abstracts or full-texts for each cited paper so NotebookLM can accurately cross-reference and map individual citations.

## 📚 Linked References
- `references/python-docx-manipulation.md`: Patterns for reading, creating, and safely removing XML paragraphs from `.docx` files.
- `references/academic-api-patterns.md`: Reliable curl and Python patterns for hitting Crossref and OpenAlex APIs, including critical URL-encoding fixes.
- `scripts/verify_urls.py`: Python script utilizing `ddgs` and `requests` to programmatically search and verify evidence URLs for Phase 4.5.
- `templates/fetch_openalex_background.js`: Node.js template for background fetching from the OpenAlex API, including the logic to decode `abstract_inverted_index` and respect rate limits.

## ⚠️ Pitfalls & Strict Rules

- **STRICT COMPLIANCE REQUIRED:** You MUST strictly follow EVERY step of this pipeline in order. Skipping phases, ignoring instructions, or taking shortcuts (such as writing drafts without full-text verification) is strictly forbidden. There are no exceptions.
- **English Output ONLY:** All generated drafts, reports, and final documents MUST be written strictly in English, regardless of the conversational language.
- **FULL-TEXT READING IS MANDATORY (NO ABSTRACT-ONLY SHORTCUTS):** You MUST NOT rely solely on abstracts. You MUST download, scrape, or extract the *entire full text* of the papers before synthesizing or extracting mechanisms. If full text is inaccessible, you MUST drop the paper or ask the user for help. DO NOT proceed to drafting based only on OpenAlex/Scopus abstracts.
- **No Hallucinated Citations:** Every claim MUST map to a real URL/DOI found during Phase 1/2.
- **MDPI Exclusion & 403 Errors:** MDPI servers (DOI prefix `10.3390`) aggressively block headless requests (curl/node), resulting in 403 Forbidden errors during Phase 4.5 URL verification. Do NOT just fetch them and fail later. When writing the Phase 0.5 background fetch script (e.g., using OpenAlex), **programmatically filter out MDPI** by checking `host_organization_name` (must not include "mdpi") and `doi` (must not include "10.3390") *before* processing or presenting the results to the user.
- **Paywalled Literature (Elsevier/Scopus):** Standard APIs (Crossref/Exa) cannot fetch full text from closed databases. To bypass academic paywalls, spawn a subagent using **CloakBrowser** to navigate to the article URL directly—if the host machine is on a university network, this automatically leverages IP-based access while bypassing bot protections.
- **Action Over Planning:** Do not tell the user what you *will* do. Immediately start executing Exa Search and Playwright tool calls.
- **Table First:** Always build and present the literature/evidence table *before* writing the final prose.
- **Intersection of Zero (Query Formulation):** When cross-analyzing highly specific variables (e.g., comparing 4 distinct cities simultaneously), NEVER combine them into a single academic API search query (e.g., Crossref/Semantic Scholar). This yields zero results. Deconstruct the research into atomic, independent searches (e.g., "Paris drought NPP", "Singapore UHI NPP") and synthesize the results post-retrieval.
- **API Rate Limits:** Semantic Scholar public APIs aggressively rate-limit (HTTP 429). Implement delays, or use the Crossref API (`https://api.crossref.org/works?query=...`) as a more resilient fallback for bulk DOI metadata retrieval.
- **Windows MSYS Execution Context:** On Windows hosts, the `terminal` tool runs MSYS bash where `pip` and `python` commands may fail (e.g., Error 49) due to Windows Store aliases. If you must run Python, use `py script.py`. However, for background async tasks (like API fetching in Phase 0.5), **strictly prefer writing scripts in Node.js (`node`)**, as it bypasses these execution aliases reliably and natively handles async loops well in the MSYS terminal.
