# Workflow 3: Elite AI Job Matcher

> Your personal job hunting assistant. Finds jobs, matches your skills, scores your chances, and only emails you the winners.

## 📋 Overview

This workflow is your 24/7 job hunting agent. It searches for data analyst jobs (remote or in Canada), analyzes if your skills match, scores your likelihood of getting hired (0-10), and only bothers you when it finds great matches (7+). Everything else just saves to Google Sheets for later review.

**Perfect for:** Job seekers who want quality over quantity  
**Complexity:** ⭐⭐⭐⭐ Expert  
**Setup Time:** 1 hour  
**Nodes:** 15+

## 🎯 What This Workflow Does

**The Intelligence:**
1. **Searches** for "data analyst" jobs (remote or in Canada)
2. **Checks** Google Sheets: Have you already seen this job?
3. **If job already exists** → Skip it (no duplicates)
4. **If job is new:**
   - Analyzes job description vs. your skills
   - Filters for beginner-level positions
   - Scores your match (0-10 scale)
   - Saves ALL jobs to Google Sheets
   - **If score ≥ 7** → Sends you an email alert
   - **If score < 7** → Just saves, no email

**The Result:** You only get emails for jobs worth applying to. No noise, just signal.

## 🏗️ Architecture

```
┌─────────────────┐
│Schedule Trigger │  ← Runs automatically (daily/hourly)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Code (JavaScript)│  ← Defines your skills & preferences
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  HTTP Request   │  ← Scrapes job boards for "data analyst"
│  (Job Search)   │     (remote OR Canada locations)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│     Merge       │  ← Combines multiple job sources
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Code (JavaScript)│  ← Deduplicates job listings
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Get Rows       │  ← Loads jobs you've already seen
│ (Google Sheets) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Code (JavaScript)│  ← Checks: Is this job already saved?
└────────┬────────┘
         │
         ▼
    IF Job Exists?
         │
    ┌────┴────┐
    │         │
   YES       NO
    │         │
 SKIP      CONTINUE
           │
           ▼
    ┌─────────────────┐
    │ Code (JavaScript)│  ← Filters for beginner level
    └────────┬────────┘
            │
            ▼
    ┌─────────────────┐
    │ Build AI Prompt │  ← Prepares skill matching request
    └────────┬────────┘
            │
            ▼
    ┌─────────────────┐
    │  HTTP Request   │  ← AI analyzes job vs. your skills
    │  (Groq API)     │     Returns score 0-10
    └────────┬────────┘
            │
            ▼
    ┌─────────────────┐
    │ Code (JavaScript)│  ← Extracts score from AI response
    └────────┬────────┘
            │
            ▼
    ┌─────────────────┐
    │ Append/Update   │  ← Saves job to Google Sheets
    │   Row (Sheets)  │
    └────────┬────────┘
            │
            ▼
       IF Score ≥ 7?
            │
       ┌────┴────┐
       │         │
      YES       NO
       │         │
       ▼         ▼
   ┌────────┐  END
   │ Gmail  │  (saved only)
   │ Alert  │
   └────────┘
```

## 🔧 Setup Guide

### Prerequisites

- n8n account (cloud or self-hosted)
- Groq API key ([Get one here](https://groq.com/))
- Gmail account
- Google Sheets

### Step 1: Create Your Job Tracking Sheet

1. Create a new Google Sheet named "Job Tracker"
2. Add these column headers in Row 1:
   - `A1`: Job Title
   - `B1`: Company
   - `C1`: Location
   - `D1`: Job URL
   - `E1`: Match Score
   - `F1`: Date Found
   - `G1`: Status
   - `H1`: Notes

### Step 2: Import Workflow

1. Download `workflow.json` from this folder
2. Open n8n → **Workflows** → **Import from File**
3. Select the downloaded JSON file

### Step 3: Configure Your Profile

Open the first **Code (JavaScript)** node and customize:

```javascript
// Your skills profile
const mySkills = {
  technical: [
    "Python",
    "SQL", 
    "Excel",
    "Power BI",
    "Tableau",
    "Data Analysis",
    "Statistics"
  ],
  experience: "beginner", // beginner, intermediate, or senior
  locations: ["Remote", "Canada", "Toronto", "Vancouver"],
  jobTitles: ["Data Analyst", "Junior Data Analyst", "Business Analyst"]
};

return { json: mySkills };
```

**Customize this to match YOUR actual skills and preferences.**

### Step 4: Configure Job Search

Update the **HTTP Request** nodes to search your preferred job boards:

**Example job board APIs:**
- LinkedIn Jobs API
- Indeed API
- RemoteOK API
- We Work Remotely
- AngelList

Or use web scraping if APIs aren't available.

### Step 5: Configure AI Scoring

Open the **Build AI Prompt** node:

```javascript
const prompt = `
You are a job matching expert. Analyze this job posting and score how well it matches the candidate's profile.

CANDIDATE PROFILE:
Skills: ${$json.mySkills.technical.join(", ")}
Experience Level: ${$json.mySkills.experience}
Preferred Locations: ${$json.mySkills.locations.join(", ")}

JOB POSTING:
Title: ${$json.jobTitle}
Company: ${$json.company}
Location: ${$json.location}
Description: ${$json.jobDescription}

SCORING CRITERIA:
- Skills match: 40%
- Experience level fit: 30%
- Location preference: 20%
- Role clarity: 10%

Return ONLY a number from 0-10 representing match quality.
10 = Perfect match, 0 = Terrible match

Score:
`;

return { json: { prompt } };
```

### Step 6: Set Up Automation Schedule

Replace **Manual Trigger** with **Schedule Trigger**:

**Recommended schedules:**
```
0 9 * * *     → Every day at 9 AM
0 9,17 * * *  → Twice daily (9 AM, 5 PM)
0 */6 * * *   → Every 6 hours
```

**Start conservative:** Once daily while testing, then increase frequency.

### Step 7: Configure Email Alerts

Open the **Send Email** node and customize the template:

```
Subject: 🎯 High Match Job Found (Score: {{$json.score}}/10)

Hi,

I found a job that matches your profile really well!

📊 Match Score: {{$json.score}}/10

🏢 Company: {{$json.company}}
💼 Role: {{$json.jobTitle}}
📍 Location: {{$json.location}}

🔗 Apply Here: {{$json.jobUrl}}

Why it's a good match:
{{$json.matchReason}}

---
Automated by your Job Matcher
```

## 📝 How the Workflow Thinks

### Step 1: Skill Matching
```javascript
// AI analyzes job requirements vs. your skills
if (job requires Python && you know Python) → +points
if (job requires 5 years experience && you're beginner) → -points
```

### Step 2: Level Filtering
```javascript
// Filters out jobs beyond your experience
if (jobLevel === "senior" && yourLevel === "beginner") → skip
if (jobLevel === "entry" || "junior") → continue
```

### Step 3: Scoring Logic
```
Score 9-10: Dream job, apply immediately
Score 7-8:  Strong match, definitely apply
Score 5-6:  Moderate match, saved for later
Score 0-4:  Poor match, saved but no alert
```

### Step 4: Duplicate Detection
```javascript
// Checks Google Sheets for existing jobs
if (jobUrl exists in sheet) → skip
if (jobTitle + company combo exists) → skip
else → process
```

### Step 5: Smart Notifications
```javascript
if (score >= 7) {
  saveToSheets();
  sendEmail();  // You get notified
} else {
  saveToSheets();  // Just saved, no email
}
```

## 💡 Real-World Example

**Morning Run (9 AM):**
```
Found 47 "data analyst" jobs
├─ 12 already in your sheet → Skipped
├─ 18 senior level → Filtered out
├─ 17 new beginner/junior jobs analyzed:
    ├─ 3 scored 8-9 → Email sent ✉️
    ├─ 7 scored 5-7 → Saved only 💾
    └─ 7 scored 0-4 → Saved for reference 📋
```

**Result:** You get 3 quality emails, not 47 job spam notifications.

## 🔄 Advanced Modifications

### Add More Job Boards

Add additional **HTTP Request** nodes:
```javascript
// LinkedIn Jobs
GET: https://api.linkedin.com/jobs/search?keywords=data+analyst

// Indeed
GET: https://api.indeed.com/jobs?q=data+analyst&l=remote

// RemoteOK
GET: https://remoteok.io/api/jobs?search=data+analyst
```

Use **Merge** node to combine all sources.

### Add Salary Filtering

```javascript
// In the filtering code node
if (job.salary < 50000) → skip
if (job.salary >= 70000) → bonus points to score
```

### Track Application Status

Add columns to Google Sheets:
- `Applied` (Yes/No)
- `Applied Date`
- `Response Status` (Pending/Interview/Rejected)
- `Interview Date`

### Add Slack Notifications

Replace/add **Slack** node for high scores:
```
Channel: #job-alerts
Message: 🎯 Score 9/10: {{jobTitle}} at {{company}}
```

### Weekly Summary Email

Add a second workflow that runs weekly:
```
Sunday 6 PM:
- Count total jobs found this week
- List top 10 matches
- Send summary email
```

## 🐛 Troubleshooting

### Issue: "No Jobs Found"
**Solution:** 
- Check job board API is working
- Verify search keywords
- Test API endpoints manually
- Check rate limits

### Issue: "All Jobs Scored Low"
**Solution:**
- Your skill profile might be too restrictive
- Broaden the `jobTitles` array
- Lower experience requirements
- Adjust scoring criteria

### Issue: "Duplicate Jobs Still Coming Through"
**Solution:**
- Improve duplicate detection logic
- Check if job URLs are formatted differently
- Add company name + title combo check
- Clear old entries from Google Sheets

### Issue: "Too Many Emails"
**Solution:**
- Raise the score threshold (7 → 8)
- Reduce workflow frequency
- Add more filtering criteria
- Implement daily digest instead

## 📊 What You'll Learn

By building this workflow, you'll master:

- ✅ Web scraping and API integration
- ✅ Complex conditional logic (multiple IF branches)
- ✅ AI-powered decision making (scoring)
- ✅ Duplicate detection algorithms
- ✅ Smart notification systems
- ✅ Data persistence and tracking
- ✅ Production-grade automation

## 🎯 Success Metrics

Track your results:
- **Jobs found per day:** 20-50+
- **Jobs worth applying to (7+):** 2-5 daily
- **Time saved:** 2-3 hours of manual searching
- **Applications submitted:** Track in Google Sheets
- **Interview rate:** Monitor quality of matches

## 💬 Questions?

**Want different job titles?** Update the search keywords  
**Need help with scoring logic?** Check the AI prompt customization  
**Stuck on setup?** Open an issue with screenshots

## 🎓 Mastery Checklist

You've mastered this workflow when you can:

- [ ] Customize skill matching for any role
- [ ] Add new job boards as sources
- [ ] Adjust scoring criteria based on results
- [ ] Handle edge cases (missing data, API failures)
- [ ] Track application outcomes
- [ ] Optimize for your job search strategy

---

**This is automation that actually helps you get hired.**

Not just alerts—intelligent filtering that saves time and increases quality.

**Stop drowning in job posts. Start getting curated opportunities.**

---

[← Back to Main Repo](../../) | [← Previous: AI Research Agent](../02-ai-research-agent/)
