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
