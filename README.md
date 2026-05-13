# Build Page 3 — Investigation Board for the AI Incident Management System.
This page connects to the existing Page 2 dashboard and updates ticket 
statuses in real time.

## Context
- Page 2 is the dashboard with 3 AWT tickets already loaded
- Page 3 receives the selected ticket from Page 2 via shared state / context
- Any status change on Page 3 must reflect immediately on Page 2 dashboard
- Use the existing wiki loader already in the codebase for Step 3
- Use the existing state management / context pattern already established

## Default Selected Ticket
Pre-select this AWT ticket as the working example:
{
  id: "AWT-001",
  title: "Why did my portfolio value drop by 40% overnight?",
  priority: "High",
  type: "AWT",
  affectedAccounts: 3,
  reportedAt: "9:00 AM EST",
  notes: "No trades executed. Transaction service restarted at 2AM."
}

## Agent Execution Flow (8 Steps — Sequential)

### Step 1: Incident Analyzer
- Parse the ticket and extract:
  { problemStatement, affectedSystem, severity, missingInfo[] }
- Call the AI model currently configured in the project

### Step 2: Problem Statement Finder
- From Step 1 output, generate a concise 2-3 line summary:
  what broke / when / who is affected / business impact

### Step 3: Wiki & Documentation Search
- Use the EXISTING wiki loader in the codebase to search
- If wiki search returns no results or fails → fall back to doc search
- Output: top 3 results [{ title, source, relevantSnippet, score }]
- Show which source was used: "wiki" or "docs fallback"

### Step 4: Historical Memory Search (RAG)
- Search past resolved tickets for similar patterns
- Output: top 3 matches [{ ticketId, summary, resolution, similarityScore }]
- Use existing RAG setup in the codebase if available

### Step 5: Query Generator
- Based on all context so far, generate:
  - 2 SQL queries targeting the relevant DB tables
  - 1 log query (grep/filter pattern)
- Display each in a syntax-highlighted code block
- Call the AI model to generate these based on actual ticket context

### Step 6: SQL Executor — CLI Visual
- When this step is selected/active, show a CLI terminal panel:
  - Animate the SQL command being typed character by character
  - Show a blinking cursor while "running"
  - Then stream the results in as if printing to terminal
  - Then render the same results as a clean data table below the terminal
- Use realistic mock data matching the AWT-001 ticket context
  (portfolio transactions, account balances, timestamps around 2AM)

### Step 7: Log Query Executor — CLI Visual
- Same CLI terminal panel pattern as Step 6:
  - Animate the log grep command being typed
  - Stream mock log lines one by one (15-20 lines)
  - Highlight matching lines in yellow/amber
  - Show: timestamp | level | service | message
- Mock logs should reference: transaction-service, restart events, 
  portfolio-calculator, 2AM timeframe

### Step 8: Final Answer Generator
- Synthesize all step outputs into:
  - Root cause (2-3 sentences)
  - Recommended fix steps (numbered)
  - Confidence score (0-100%)
- For AWT-001, show action: "Generate Final Report"
  - Clicking it opens a report preview panel
  - Report includes: ticket summary, root cause, queries run, 
    findings, recommended actions, resolved by AI badge
  - Report has a [Send Report] button
  - On send: ticket status → "Under Review" 
    and this syncs to Page 2 dashboard instantly

## CLI Terminal Component (reuse for Steps 6 & 7)
- Black terminal background, green monospace font
- Show a header bar: "● AWT Investigation Terminal"
- Character-by-character typing animation for the command (50ms per char)
- After command: show a loading spinner for 1.5s
- Then stream output lines with 80ms delay between lines
- Show execution time at the bottom: "Executed in 1.24s — 847 rows scanned"

## Status Sync to Page 2
- Use the same global state / context / store already used in the project
- Status values: Open → Investigating → Under Review → Escalated → Complete
- When agent starts: ticket status → "Investigating" (syncs to Page 2)
- When report sent: ticket status → "Under Review" (syncs to Page 2)
- Page 2 ticket card must reflect the new status badge in real time
  without a page refresh

## Step UI Pattern
Each of the 8 steps is a card with:
- Step number + name on the left
- Status badge on the right: 
  pending (gray) | running (blue pulse) | done (green) | error (red)
- Time taken shown after completion e.g. "2.1s"
- Collapsed after done, expandable to re-view output
- On error: show [Retry] and [Skip] buttons

## Layout
- Top bar: ticket ID, title, priority badge, current status, [Investigate] button
- Main area: vertical step cards (left 60%) + active step detail panel (right 40%)
- The right panel shows the expanded output of whichever step is active
- CLI terminal renders inside the right panel for Steps 6 and 7

## Do Not
- Do not add any comments referencing specific model names
- Do not create new mock data patterns — follow whatever mock data 
  approach is already used in the codebase
- Do not create a new state management system — use what exists
- Do not create a new wiki loader — use the existing one


## RAG Memory System — Solved Ticket Vector Store

### On Ticket Completion
When a ticket is marked as Complete or "Under Review" after 
the final report is sent, automatically store the full 
resolution in the vector DB:

Document to store:
{
  ticketId: "AWT-001",
  title: "original ticket title",
  problemStatement: "extracted in Step 1",
  rootCause: "from Step 8",
  affectedSystem: "from Step 1",
  sqlQueriesUsed: ["query1", "query2"],
  logQueryUsed: "log grep pattern",
  sqlFindings: "summary of table results",
  logFindings: "summary of log results",
  fixSteps: ["step1", "step2", ...],
  wikiSourcesUsed: [{ title, url }],
  confidenceScore: 92,
  resolvedAt: "ISO timestamp",
  resolvedBy: "AI Agent",
  tags: ["portfolio", "transaction-service", "AWT"]
}

- Generate an embedding for the combined text of:
  title + problemStatement + rootCause + tags
- Store embedding + full document in the existing vector DB
- Use the same vector DB client/config already in the project
- Collection name: "resolved_incidents"
- If collection does not exist: create it on first store

### Step 4 — Historical Memory Search (RAG)
This step now queries the "resolved_incidents" collection:

- Generate embedding from current ticket's problem statement
- Run similarity search: top 3 results with score > 0.75
- If results found:
  - Show each as a card:
    [AWT-002] Portfolio NAV mismatch — resolved 3 days ago
    Similarity: 89% | Fix: rerun portfolio calculator job
  - Highlight the most similar one with a "Best Match" badge
- If no results found (first time this type appears):
  - Show: "No similar past incidents found — 
            this resolution will be stored for future use"

### Step 1 Enhancement — Memory-Aware Analysis
After extracting the problem statement in Step 1:
- Immediately do a quick RAG lookup (top 1 result, score > 0.85)
- If a strong match is found:
  - Show an "Similar incident resolved before" banner at the top
  - Pre-fill the agent with context from the past resolution
  - Agent can skip or fast-track steps 3-7 using cached findings
  - Show: "Fast Track Mode — using memory from [ticket ID]"

### Agent Decision Logic
After Step 4 RAG results come in, the agent decides:

If similarityScore > 0.85 (very similar):
  → Show: "High confidence match found"
  → Offer [Fast Track] button — skips to Step 8 using past resolution
  → Still show all steps as "skipped with memory" in green

If similarityScore 0.65-0.85 (partial match):
  → Continue full flow but pre-populate queries in Step 5
    with the ones that worked last time
  → Show: "Partial match — using historical queries as base"

If similarityScore < 0.65 or no match:
  → Run full 8-step flow as normal
  → New resolution will be stored after completion

### Memory Panel (UI)
Add a collapsible "Memory" sidebar panel on the right:
- Shows all retrieved similar tickets from RAG
- Each entry: ticket ID, title, similarity %, resolution summary,
  date resolved, [View Full Resolution] button
- A small brain/memory icon in the step 4 card header
- After ticket is stored: show a 
  "✓ Stored in memory" toast notification
  at the bottom of the page

### Storage Trigger
Store to vector DB at these points:
- When [Send Report] is clicked and status → "Under Review"
- When ticket status → "Complete"
- Do NOT store if the agent flow errored out or was incomplete
- Do NOT store duplicates — check ticketId before inserting

### Retrieval at Page Load
When Investigation Board loads with a selected ticket:
- Immediately run a background RAG search silently
- If match found: show a subtle banner before user clicks Investigate:
  "💡 Similar incident found in memory — 
     investigation may be faster"
- This gives the engineer a heads-up before starting

### Use existing setup
- Use the vector DB client already configured in the project
- Use the same embedding model already in use
- Follow the same collection/index naming conventions in the project
- If no vector DB exists yet in the project: use ChromaDB with 
  the JS client, store locally, and add a note in the code 
  showing where to swap in a production DB

