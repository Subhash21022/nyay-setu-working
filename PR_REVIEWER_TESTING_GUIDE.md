PR Reviewer & Testing Guide
===========================

Summary
-------
This short guide explains how to run the lawgpt-service backend and the nyaysetu frontend locally, verify the document-generation flows (PDF/DOCX export and clipboard copy), and run the automated frontend tests added in this patch.

Quick checklist for reviewers
---------------------------
- [ ] Start the backend (with DEV fallback if desired)
- [ ] Start the frontend and point it at the backend
- [ ] Manually exercise: select document type -> fill fields -> Generate Preview -> Download PDF / Download DOCX / Copy to clipboard
- [ ] Run the frontend tests (Vitest) and confirm they pass
- [ ] Inspect code changes and commit history for clarity

Prerequisites
-------------
- Node 18+ and npm
- Python 3.10+ (or the project's supported Python version)
- git

Backend (lawgpt-service)
------------------------
1. Create and activate a Python virtualenv in the repo root (recommended):

```powershell
cd lawgpt-service
python -m venv .venv
.\.venv\Scripts\activate
```

2. Install dependencies:

```powershell
pip install -r requirements.txt
```

3. Optional DEV fallback for local testing (allows running without upstream LLMs/retriever):

- Set environment variable `LAWGPT_FAKE_LLM=1` to enable the DummyLLM fallback used for local QA only.

Important: DO NOT set `LAWGPT_FAKE_LLM=1` in production.

4. Start the backend (example):

```powershell
# from repo root
cd lawgpt-service
.\.venv\Scripts\python.exe -m uvicorn main:app --host 127.0.0.1 --port 8001
```

Backend will be available at http://127.0.0.1:8001

Frontend (nyaysetu-frontend)
---------------------------
1. Install node deps and run dev server:

```powershell
cd frontend/nyaysetu-frontend
npm install
# set the backend base URL for the frontend
$env:VITE_LAWGPT_BASE = 'http://127.0.0.1:8001'
npm run dev
```

2. In production-like workflows use the usual build flow (`npm run build`) and host the output.

Example manual generation flow
-----------------------------
1. Open the frontend app in a browser (the dev server URL printed by Vite).
2. Select a document type (e.g. Affidavit).
3. Fill the fields (Petitioner name, address, incident date, respondent, facts, relief sought).
4. Click "Generate Preview" — the app should call `/generate` and render a preview.
5. Click "Download PDF" — the app should call `/generate/pdf` and prompt a download.
6. Click "Download DOCX" — similar behavior for `/generate/docx`.
7. Click "Copy to clipboard" — clipboard receives the generated content.

Automated tests (frontend)
--------------------------
- Run all frontend tests:

```powershell
cd frontend/nyaysetu-frontend
npx vitest
```

- Run the specific page test added in this patch:

```powershell
npx vitest run src/pages/litigant/DocumentGeneratePage.test.jsx
```

Notes on the DummyLLM / Dev fallback
-----------------------------------
- The `LAWGPT_FAKE_LLM=1` switch enables a local DummyLLM fallback to avoid requiring LLM API keys or retriever dependencies during manual QA.
- Keep this disabled when verifying integration against real LLMs and retrievers.

What to review in the PR
------------------------
- Tests: confirm the new PDF download test covers generatePdf invocation and the download blob/filename flow.
- UX: validate the error mapping (validation & prompt-injection) surfaces user-friendly messages.
- Safety: ensure `LAWGPT_FAKE_LLM` is only mentioned as a dev-only switch and is gated from production usage.

Optional: Screenshots / GIFs
---------------------------
If useful, record a short GIF showing: select Affidavit -> fill fields -> Generate Preview -> Download PDF. Place screenshots under `docs/screenshots/` in the PR for reviewer convenience.

If you want me to:
- open a feature branch and commit these changes (I added this file locally),
- push the branch and open a PR draft with a testing summary and suggested reviewers,
I can run git commands for those actions — tell me to proceed and I'll create the branch and show the push/PR commands.
