# 🎂 Birthday Email Automation — n8n

Automated birthday reminder that reads from Excel daily and sends personalized Gmail messages.

---

## 📁 Files
| File | Purpose |
|------|---------|
| `birthday_workflow.json` | Import this into n8n |
| `birthdays.xlsx` | Your people data (edit this) |

---

## 🗂️ Excel Format (`birthdays.xlsx`)

| Column | Description | Example |
|--------|-------------|---------|
| **Name** | Full name | Alice Johnson |
| **Email** | Recipient email | alice@company.com |
| **Birthday** | Date (YYYY-MM-DD) | 1990-04-18 |
| **Department** | Optional dept | Engineering |
| **Message** | Custom override message (optional) | Happy B-day! |

> If **Message** is blank, a default warm message is used automatically.

---

## ⚙️ Setup Steps

### 1. Import Workflow into n8n
1. Open n8n → **Workflows** → **Import from File**
2. Upload `birthday_workflow.json`

### 2. Connect Gmail OAuth2
1. Go to **Credentials** → **New** → **Gmail OAuth2**
2. Follow Google OAuth setup (enable Gmail API in Google Cloud Console)
3. In the workflow, click **"Send Birthday Email"** node → select your credential

### 3. Set Excel File Path
Set the environment variable in n8n:
```
BIRTHDAY_EXCEL_PATH=/data/birthdays.xlsx
```
Or update the path directly in the **"Read Excel File"** node settings.

Place `birthdays.xlsx` at that path (mount it into Docker if using n8n Docker).

### 4. Configure Cron Schedule
The cron is set to **9:00 AM daily**:
```
0 9 * * *
```
To change the time, edit the **"Daily Cron"** node:
| Time | Cron Expression |
|------|----------------|
| 8:00 AM | `0 8 * * *` |
| 9:00 AM | `0 9 * * *` (default) |
| 10:30 AM | `30 10 * * *` |

### 5. Activate Workflow
Toggle the workflow to **Active** — it will now run automatically every day!

---

## 🔄 Workflow Flow

```
[Cron: 9AM Daily]
       ↓
[Read Excel File]
       ↓
[Check Today's Birthdays]  ← compares month+day only
       ↓
[Has Birthdays?]
   ↙         ↘
[YES]        [NO]
  ↓            ↓
[Build HTML   [Log: No
 Email]        Birthdays]
  ↓
[Send via Gmail]
  ↓
[Log Success]
```

---

## 📧 Email Preview

The email sent is a **styled HTML card** containing:
- 🎉 Colorful gradient header with person's name
- 🎂 Personalized message (custom or default)
- 🥳 Age display if birth year is in Excel
- ❤️ Branded footer

---

## 🐳 Docker Compose Tip

If running n8n with Docker, mount your Excel file:

```yaml
volumes:
  - ./birthdays.xlsx:/data/birthdays.xlsx
```

---

## 🛠️ Troubleshooting

| Problem | Fix |
|---------|-----|
| No emails sent | Check Excel path is correct and file is accessible |
| Gmail auth error | Re-authorize OAuth2 credential in n8n |
| Wrong birthdays matched | Ensure Birthday column is `YYYY-MM-DD` format |
| Workflow not running | Check workflow is toggled **Active** |
