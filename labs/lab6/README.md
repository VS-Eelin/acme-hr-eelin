# Lab 6 — Structured Data Extraction: document intake (30 min)

**Goal:** turn MCs and expense claims, as PDF files, into checked data. Claude gives the shape; C# decides if it is right; a person reviews the rest.

**Branch:** `git checkout lab6-start` · Solution: `final`. Prompts: [PROMPTS.md](../../PROMPTS.md#lab-6--document-intake).

## Before you start
- The 20 samples are PDF files in `seed/documents/`: text PDFs and image-only scans (03, 06, 09, 13, 16, 20), all synthetic. Each PDF goes to Claude as a document block; Claude reads its text and its pages as images.
- C# checks every upload before any Claude call: PDF only (`%PDF`), max 5 MB, max 5 pages. Others get 400.
- An MC is medical data (POL-12): the stored PDF opens only for the employee who sent it and for HR. Others get 403.
- Cost: one ExtractScore run is up to 40 Claude calls (20 documents × up to 2 attempts). PDFs cost more than plain text, because every page is also read as an image.

## Steps
6.1 `git checkout lab6-start`, then `git switch --no-track -c lab6-work`. (`--no-track` stops a bare `git push` from going to `lab6-start` or `main`.) Stop the API, run `dotnet test`: 5 extractor tests fail.

6.2 Read `docs/feature-spec-documents.md`, `src/Api/Documents/DocumentExtractor.cs` and `DocumentValidator.cs`. Switch to plan mode, paste **Finish the extractor**, review the plan, then let it build.

6.3 Start the API, then run `dotnet run --project tools/ExtractScore`. Each wrong field is printed by name, e.g. `wrong date: expected 2026-10-03, got 2026-03-10`.

6.4 Paste **Improve the score** once. The prompt is compiled into the API, so the order is: stop the API, edit, run the tests, start the API, run the score.

6.5 With the API and web app running locally: as Farah, upload sample `08-mc.pdf` on **Documents**. Sign in as Dina (`dina.ismail4@acme-hr.example`, HR) and open **Review**: it lists only documents that need a person (try 08, 10, 17, 19 or 20; a Valid document does not appear). Open the PDF from the row and compare. Farah cannot open Review.

6.6 Commit, push with `git push -u origin lab6-work`, open a pull request to `main`. The gates from Lab 5 apply: merge when both checks are green, then check the deploy printed a `dep-` id.

6.7 On the live site: as Farah upload one sample at `/app/documents/`, then as Dina open `/app/review/` (your private password). The live database is separate from your local SQLite data; the API adds the new tables at start-up.

## Done when
- [ ] 20/20 documents end as `Valid` or `NeedsReview` (no errors)
- [ ] Samples 08, 10 and 17 end in `NeedsReview` with their values as written, not "fixed" by Claude
- [ ] It works on the live site

## Stuck?
- HTTP 500 on every document: `ANTHROPIC_API_KEY` is not set in `.env`, or the API was not restarted.
- HTTP 400 "Only PDF files are accepted" or "The file is not a PDF": upload the files from `seed/documents/`, not a `.txt` or an image.
- `no such table: Documents` or `DocumentFiles`: restart the API; it adds missing tables at start-up.
- Claude "fixes" the wrong totals: the retry message must say copy values as written; the check is in C#.
- Score did not change after a prompt edit: the API still runs the old build. Restart it.
- Windows: "Tests failing" with no test names after an edit: the running API locks the build. Stop it first.

## Debrief
Which fields failed on the scans but not on the text PDFs? Which errors should a person review, and which can code reject on its own? The schema forces the shape of the output; only the prompt can say how to read an ambiguous input.
