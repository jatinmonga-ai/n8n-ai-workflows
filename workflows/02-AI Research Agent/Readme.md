# Workflow 2: AI Research Agent

> Multi-agent coordination. Conditional logic. Persistent memory. Production-grade automation.

## 📋 Overview

This workflow represents the leap from simple automation to intelligent systems. It doesn't just execute tasks—it makes decisions, maintains memory, and coordinates multiple AI agents to build a growing knowledge base.

**Perfect for:** Builders ready for production systems  
**Complexity:** ⭐⭐⭐ Advanced  
**Setup Time:** 45 minutes  
**Nodes:** 10+

## 🎯 What This Workflow Does

**The Intelligence:**
1. You input a research topic
2. System checks Google Sheets: Does this topic exist?
3. **If NEW topic:**
   - AI Agent 1 searches and analyzes from scratch
   - Generates bullet-point summary
   - Emails you the insights
   - Saves to Google Sheets for future reference
4. **If EXISTING topic:**
   - AI Agent 2 finds new angles and fresh insights
   - Updates the existing Google Sheets row
   - Emails you the updated research

**The Result:** A self-improving research system that builds institutional knowledge over time and keeps you informed via email.

## 🏗️ Architecture

```
┌─────────────────┐
│ Manual Trigger  │  ← Start research pipeline
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Edit Fields    │  ← Define research topic
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Get Rows       │  ← Load existing research
│ (Google Sheets) │     from knowledge base
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ IF Conditional  │  ← Decision point
│ Topic exists?   │
└────┬────────┬───┘
     │        │
 NO  │        │ YES
     │        │
     ▼        ▼
┌─────────┐  ┌──────────┐
│ Branch 1│  │ Branch 2 │
│  (NEW)  │  │(EXISTING)│
└────┬────┘  └────┬─────┘
     │            │
     ▼            ▼
┌─────────────┐ ┌──────────────┐
│ AI Agent 1  │ │ AI Agent 2   │
│ Research &  │ │ Find fresh   │
│ Analyze     │ │ insights     │
└────┬────────┘ └────┬─────────┘
     │               │
     ▼               ▼
┌─────────────┐ ┌──────────────┐
│ Generate    │ │ Update       │
│ Summary     │ │ Existing Row │
└────┬────────┘ └────┬─────────┘
     │               │
     ▼               ▼
┌─────────────┐ ┌──────────────┐
│ Send Email  │ │ Send Email   │
│ (Gmail)     │ │ (Gmail)      │
└────┬────────┘ └────┬─────────┘
     │               │
     ▼               ▼
┌─────────────┐ ┌──────────────┐
│ Save New    │ │ Update Sheet │
│ (G Sheets)  │ │ (G Sheets)   │
└─────────────┘ └──────────────┘
```

## 🔧 Setup Guide

### Prerequisites

- n8n account (cloud or self-hosted)
- Groq API key ([Get one here](https://groq.com/))
- Gmail account
- Google Sheets account

### Step 1: Create Google Sheet

1. Create a new Google Sheet named "AI Research Database"
2. Add these column headers in Row 1:
   - `A1`: Topic
   - `B1`: Research Summary
   - `C1`: Last Updated
   - `D1`: Status

**Example:**
| Topic | Research Summary | Last Updated | Status |
|-------|-----------------|--------------|--------|
| AI in healthcare | • Key insight 1<br>• Key insight 2 | 2025-02-11 | Active |

### Step 2: Import Workflow

1. Download `workflow.json` from this folder
2. Open n8n → **Workflows** → **Import from File**
3. Select the downloaded JSON file

### Step 3: Configure Google Sheets Connection

1. Open the **Get Rows (Google Sheets)** node
2. Click **Create New Credential**
3. Follow OAuth2 authentication
4. Select your spreadsheet: "AI Research Database"
5. Repeat for **Save to Google Sheets** and **Update Google Sheets** nodes

### Step 4: Configure Groq API

**For both AI Agent nodes:**

1. Open each **Groq Chat Model** node
2. Add authentication:
   - Type: `Header Auth`
   - Name: `Authorization`
   - Value: `Bearer YOUR_GROQ_API_KEY`

### Step 5: Configure Gmail

1. Open the **Send Email** node
2. Create Gmail credential (OAuth2)
3. Update recipient email address

### Step 6: Set Up Conditional Logic

The **IF** node checks if your topic already exists:

**Condition:**
```javascript
{{ $json.exists === true }}
```

**Logic:**
- If topic found in Google Sheets → Route to Branch 2 (Update)
- If topic NOT found → Route to Branch 1 (Create New)

### Step 7: Test Run

1. Open **Edit Fields** node
2. Enter a test topic: "AI agents in healthcare"
3. Click **Execute Workflow**
4. First run → Creates new research
5. Second run → Updates existing research

## 📝 Configuration Details

### AI Agent 1 (New Research)

**Purpose:** Generate comprehensive research summary for new topics

**Prompt Template:**
```
Research the following topic and provide detailed insights:

Topic: {{$json.topic}}

Please provide:
- 5-7 key insights in bullet points
- Current trends and developments
- Practical applications
- Future outlook

Format your response as clean bullet points.
```

**Parameters:**
- Model: `mixtral-8x7b-32768`
- Temperature: `0.5` (balanced creativity)
- Max Tokens: `1500`

### AI Agent 2 (Update Existing)

**Purpose:** Find fresh angles and new insights on existing topics

**Prompt Template:**
```
Here is existing research on {{$json.topic}}:

{{$json.existing_summary}}

Your task:
- Identify what's missing or outdated
- Find 3-5 NEW insights not covered above
- Focus on recent developments (last 30 days)
- Keep it actionable

Format as bullet points.
```

**Parameters:**
- Model: `mixtral-8x7b-32768`
- Temperature: `0.6` (slightly more creative)
- Max Tokens: `1000`

### Conditional Logic (IF Node)

**Expression:**
```javascript
// Check if topic exists in any row
{{ $json.sheet_data.filter(row => 
     row.Topic.toLowerCase() === $json.topic.toLowerCase()
   ).length > 0 }}
```

**True:** Topic exists → Update flow  
**False:** New topic → Create flow

## 💡 Use Cases

### 1. Continuous Market Research
**Scenario:** Track competitor updates weekly

**Setup:**
- Topics: Competitor names
- Schedule: Every Monday
- Output: Updated competitive analysis in Sheets

**Benefit:** Never lose context on competitive movements

### 2. Knowledge Base Building
**Scenario:** Research team needs centralized insights

**Setup:**
- Topics: Industry trends, technologies, strategies
- Schedule: Daily or on-demand
- Output: Growing, searchable knowledge base

**Benefit:** Institutional knowledge that compounds

### 3. Investment Research
**Scenario:** Track developments in investment areas

**Setup:**
- Topics: Specific companies, sectors, markets
- Schedule: Daily for active topics
- Output: Time-series research updates

**Benefit:** See how narratives evolve over time

### 4. Content Research
**Scenario:** Content creators need fresh angles

**Setup:**
- Topics: Content themes and subjects
- Schedule: Before each content cycle
- Output: Updated insights + what's already covered

**Benefit:** Avoid redundancy, find new angles

## 🔄 Advanced Modifications

### Add Web Scraping

Insert **HTTP Request** node before AI agents:
- Scrape specific news sources
- Extract competitor blog posts
- Pull data from APIs
- Feed real-time data to AI agents

### Multi-Source Research

Add multiple AI agent pairs:
- Agent A: General research
- Agent B: Technical deep-dive
- Agent C: Business implications

Merge outputs in Google Sheets with different columns.

### Automated Triggers

Replace **Manual Trigger** with:

**Cron Trigger** (Scheduled):
```
0 9 * * 1-5  → Weekdays at 9 AM
0 */6 * * *  → Every 6 hours
```

**Webhook Trigger** (API-driven):
```
POST /webhook/research
Body: { "topic": "your-topic" }
```

### Slack Integration

Add **Slack** node after email:
```
Channel: #research-updates
Message:
📊 New research on {{$json.topic}}

{{$json.summary}}

View full: [Google Sheets Link]
```

### Export to Notion/Airtable

Replace Google Sheets with:
- **Notion** → For rich formatting and databases
- **Airtable** → For relational data and views
- **PostgreSQL** → For production-scale storage

## 🎯 Understanding the Intelligence

### What Makes This "Multi-Agent"?

**Agent Specialization:**
- **Agent 1:** Breadth-focused (comprehensive overview)
- **Agent 2:** Depth-focused (new angles, fresh insights)

**Why Two Agents?**
- Different prompts = different thinking modes
- New research needs breadth
- Updates need depth and freshness
- Specialization improves quality

### What Makes This "Conditional"?

**Decision-Making:**
The workflow *decides* what to do based on data state.

**Without Conditionals:**
- Always creates new research
- Loses context
- Duplicates work
- No memory

**With Conditionals:**
- Checks existing knowledge
- Updates intelligently
- Maintains continuity
- Builds over time

### What Makes This "Persistent Memory"?

**Google Sheets as Database:**
- Each row = one topic
- Accumulates over time
- Queryable and searchable
- Visual interface for humans

**Memory Benefits:**
- Context awareness
- Historical tracking
- Pattern recognition
- Institutional knowledge

## 🐛 Troubleshooting

### Issue: "IF Node Always Routes to New"
**Solution:** Check conditional expression
```javascript
// Debug: Print what's being compared
console.log("Sheet Topics:", $json.sheet_data);
console.log("Input Topic:", $json.topic);
```

Ensure topic names match exactly (case-sensitive).

### Issue: "Google Sheets Not Updating"
**Solution:** 
- Verify sheet permissions (Edit access)
- Check sheet name matches exactly
- Ensure row exists for updates
- Re-authenticate Google credential

### Issue: "AI Agents Return Empty Results"
**Solution:**
- Increase `max_tokens` parameter
- Simplify the prompt
- Check Groq API credits
- Try different model

### Issue: "Workflow Times Out"
**Solution:**
- Add **Wait** nodes between heavy operations
- Reduce batch sizes
- Check n8n execution timeout settings
- Upgrade n8n plan if needed

## 📊 What You'll Learn

By building this workflow, you'll master:

- ✅ Conditional logic in automation
- ✅ Multi-agent AI coordination
- ✅ Persistent data storage strategies
- ✅ Database-style operations in Sheets
- ✅ Intelligent routing and branching
- ✅ Production-grade error handling
- ✅ System design thinking

## 🎓 Next Steps

### Mastery Checklist

- [ ] Successfully run both branches (new + update)
- [ ] Add a third topic and verify routing
- [ ] Customize AI prompts for your domain
- [ ] Add a new data source (web scraping)
- [ ] Schedule automated runs
- [ ] Export to a different database
- [ ] Handle edge cases (empty inputs, API failures)

### Advanced Challenges

1. **Multi-language support** - Research in multiple languages
2. **Sentiment tracking** - Track how sentiment changes over time
3. **Source attribution** - Track which sources AI used
4. **Confidence scoring** - Rate research quality
5. **Alert system** - Notify on significant changes

## 💬 Questions?

**Complex concepts unclear?** Review [Workflow 1](../01-simple-email-automation/) first  
**Want to collaborate?** Open an issue with your use case  
**Built something cool?** Submit a PR with your modifications


## 🎯 Success Metrics

You've mastered this workflow when you can:

- [ ] Explain why two AI agents are better than one
- [ ] Debug conditional routing issues
- [ ] Design your own branching logic
- [ ] Add new data sources independently
- [ ] Optimize for your specific use case
- [ ] Deploy to production with confidence

---

**This is where automation becomes intelligence.**

You're not just running tasks—you're orchestrating decisions.

**The gap between intermediate and advanced is smaller than you think.**

---

[← Back to Main Repo](../../) | [← Previous: Email Automation](../01-simple-email-automation/)
