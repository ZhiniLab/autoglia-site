# Beads Deep Dive — Implementation Patterns for Autoglia

**Date:** March 13, 2026
**Source:** GitHub steveyegge/beads, docs, agent instructions, Medium article

---

## Why This Matters

Beads (18.7k stars) is the most prominent open-source AI agent memory system. It's NOT just a competitor — it's a **pattern library** we can learn from. Steve Yegge (ex-Amazon/Google/Groupon) built this after burning down 350k LOC — learn from his mistakes.

---

## 1. Hash-Based IDs (Critical for Multi-Agent)

### The Problem
Sequential IDs collide when multiple agents work in parallel:
```
Branch A: bd create "Add OAuth" → ID: bd-10
Branch B: bd create "Add Stripe" → ID: bd-10  # COLLISION!
```

### Beads' Solution
Hash-based IDs prevent collision:
```
bd-a1b2  (from random UUID, 4 chars for <500 issues)
bd-f14c3 (5 chars for 500-1,500 issues)  
bd-3e7a5b (6 chars for 1,500+ issues)
```

### What Autoglia Should Do
**Implement content-addressable IDs:**
```python
# Instead of auto-increment (1, 2, 3...)
# Use hash of content + timestamp
import hashlib
def generate_memory_id(content: str, timestamp: str) -> str:
    data = f"{content}:{timestamp}"
    hash = hashlib.sha256(data.encode()).hexdigest()[:6]
    return f"mem-{hash}"
```

**Why:**
- Multiple agents can write to same DB without collision
- ID is deterministic — same content = same ID
- Enables safe sync/merge from multiple sources

---

## 2. Dependency Graph (Memory Relationships)

### Beads Has Four Dependency Types
| Type | Meaning | Use Case |
|------|---------|----------|
| `blocks` | A blocks B | "Can't test until auth is done" |
| `related` | A relates to B | "This bug relates to that bug" |
| `parent-child` | A contains B | Epic → Task → Subtask |
| `discovered-from` | A discovered B | Agent work discovery |

### Autoglia Implementation
```sql
-- Add to memory_proposals or new table
CREATE TABLE memory_relationships (
    id INTEGER PRIMARY KEY,
    source_id TEXT NOT NULL,      -- mem-a1b2c3
    target_id TEXT NOT NULL,       -- mem-x9y8z7
    relationship_type TEXT NOT NULL, -- blocks, related, parent, discovered
    metadata TEXT,                  -- JSON: why, when, confidence
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Query: What does this memory depend on?
SELECT * FROM memory_relationships 
WHERE source_id = 'mem-a1b2c3';

-- Query: What's blocked by this memory?
SELECT * FROM memory_relationships 
WHERE target_id = 'mem-a1b2c3' AND relationship_type = 'blocks';
```

**Why This Matters:**
- Agents can reason about memory connections
- "What do I need to know before answering this?"
- Enables "work discovery" — find related memories automatically

---

## 3. Semantic Memory Decay (Auto-Summarization)

### Beads Does This
Old closed tasks get summarized to save context. The system keeps the "essence" but drops details.

### Autoglia Should Do This (Matches OpenClaw Video!)
```python
def memory_decay(memory_id: str, db_path: str):
    """
    When memory is old and context is full, summarize it.
    """
    memory = get_memory(memory_id)
    
    if should_summarize(memory):
        summary = summarize(memory.content)  # Use LLM
        update_memory(memory_id, {
            'content': summary,
            'is_summary': True,
            'original_version': memory.content,
            'summary_of': memory_id
        })
    
    # Keep metadata for retrieval
    # But reduce token cost in context
```

### When to Trigger Decay
- Memory is >7 days old
- Context window is >80% full
- Memory has been retrieved >10 times (stable)
- Explicit user request

---

## 4. Project Isolation (Multi-Tenant)

### Beads Architecture
- Each project has `.beads/` directory with own Dolt database
- `bd` walks up directory tree to find project (like git)
- Shared server mode available for machines with many projects

### Autoglia Already Does This
We have `~/memory.db` — but consider:
```python
# Per-project memory
~/.autoglia/projects/{project_name}/memory.db

# Agent-specific memory  
~/.autoglia/agents/{agent_id}/memory.db
```

---

## 5. Offline-First Architecture

### Beads is Offline-First
- Works without network
- Syncs via git/Dolt when online
- Conflict resolution built-in

### Autoglia Should Emphasize This
**Current strength:** Already SQLite, already local-only.

**Enhancement:** Add git sync option for backup:
```python
def backup_to_git(db_path: str, repo_path: str):
    """Export memory to git for backup/versioning."""
    # Export to JSONL
    # Commit with timestamp
    # Push if remote configured
```

---

## 6. JSON-First APIs (Agent-Native)

### Beads Pattern
Every CLI command has `--json` output:
```bash
bd ready --json        # Machine-readable queue
bd show bd-a1b2 --json # Full issue data
bd create "Title" -p 0 --json  # Created issue
```

### Autoglia Should Do This
Current: Limited JSON output.

**Enhancement:**
```bash
# Query memories as JSON
autoglia query --json --topic "project-x"
autoglia recent --json --limit 10
autoglia search "context loss" --json

# Output format:
{
  "memories": [
    {
      "id": "mem-a1b2c3",
      "content": "...",
      "topic": "project-x",
      "created_at": "2026-03-13T...",
      "retrieval_count": 5
    }
  ],
  "total": 42,
  "query_time_ms": 12
}
```

---

## 7. MCP Server (Model Context Protocol)

### Beads Has MCP
`beads-mcp` provides MCP server for Claude Desktop, Amp, etc.

### Autoglia Should Have This
```python
# autoglia-mcp/server.py
from mcp.server import Server
from mcp.types import Tool, TextContent

app = Server("autoglia")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="query_memory",
            description="Query Autoglia memory database",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {"type": "string"},
                    "limit": {"type": "number", "default": 10}
                }
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "query_memory":
        return query_memory(arguments["query"], arguments.get("limit", 10))
```

---

## 8. Agent Instructions (Prompt Engineering)

### Beads Has Specific Agent Instructions
From `AGENT_INSTRUCTIONS.md`:
- Never pollute production DB with test data
- Use `BEADS_DB=/tmp/test.db` for testing
- Include issue ID in git commits: `git commit -m "Fix bug (bd-abc)"`
- "Land the plane" workflow: always push to remote

### Autoglia Should Have This
Create `/home/johnny/autoglia-agent/AGENT_INSTRUCTIONS.md`:
```markdown
# Autoglia Agent Instructions

## Writing to Memory
- Always include source and timestamp
- Tag with relevant topics
- Deduplicate before insert

## Reading Memory
- Query relevant memories BEFORE answering
- Check memory before each significant action
- Use exact match + semantic search

## Testing
- Use test database: AUTOGLIA_DB=/tmp/test.db
- Never write test data to production

## Context Management
- Run checkpoint before context compaction
- Monitor context usage: 80% = checkpoint time
- Archive old memories after 30 days
```

---

## 9. Git Integration Patterns

### Beads Uses Git for:
1. **Auto-commit** — Hooks commit on every write
2. **Sync** — Push/pull via Dolt
3. **Branching** — Per-branch task state
4. **Conflict resolution** — Hash IDs prevent most conflicts

### Autoglia Could Add:
```bash
# Optional: Enable git sync
autoglia git init
autoglia git sync  # Push/pull memory changes

# Backup mechanism
autoglia export --jsonl memory_backup.jsonl
autoglia import --jsonl memory_backup.jsonl
```

---

## 10. Quality Gates (Reliability)

### Beads Has:
- Linting: `golangci-lint run ./...`
- Testing: `make test` + `make test-full-cgo`
- Doctor: `bd doctor` detects orphaned issues
- Issue ID in every commit for traceability

### Autoglia Should Add:
```bash
# Health check
autoglia doctor  # Check DB integrity, find orphaned memories

# Consistency check
autoglia verify  # Verify relationships, find broken links

# Audit
autoglia audit   # Find duplicate memories, vacuum old data
```

---

## Summary: What to Implement

| Priority | Feature | Why | Complexity |
|----------|---------|-----|------------|
| 🔴 Critical | Hash-based IDs | Multi-agent safety | Low |
| 🔴 Critical | JSON APIs | Agent integration | Medium |
| 🟡 Important | Memory decay | Context management | High |
| 🟡 Important | Relationship graph | Memory connections | Medium |
| 🟢 Nice | MCP server | Claude Desktop support | Medium |
| 🟢 Nice | Git backup | Version control | Low |
| 🟢 Nice | Doctor/audit | Reliability | Low |

---

## Key Takeaways from Beads

1. **Steve Yegge burned 350k LOC** before building Beads — learn from his mistakes
2. **Hash IDs are essential** for multi-agent/multi-branch workflows
3. **Offline-first** is a feature, not a limitation
4. **JSON everywhere** — agents need machine-readable output
5. **Memory decay** prevents context bloat (matches our video learnings)
6. **Relationships** between memories enable reasoning
7. **Project isolation** keeps memory organized
8. **Quality gates** (doctor, audit) catch problems early

---

*Last updated: 2026-03-13*
