# Gmail Campaign Sender

Send personalized HTML email campaigns from a CSV contact list using the Gmail API with OAuth2 authentication.

---

## Features

- **OAuth2 authentication** — secure, no passwords stored
- **CSV-driven contacts** — `email`, `name`, `company` + any custom columns
- **HTML template with variable substitution** — `{{name}}`, `{{company}}`, any column
- **PDF attachment per contact** — matched by email, company name, or a single `default.pdf`
- **Send log CSV** — every send attempt recorded with timestamp, status, and error
- **Idempotent** — skips contacts already logged as sent (re-run safely)
- **Dry-run mode** — preview everything without touching the Gmail API

---

## Quick Start

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Enable Gmail API & download credentials

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a project (or select an existing one)
3. **APIs & Services → Enable APIs** → search for **Gmail API** → Enable
4. **APIs & Services → Credentials → Create Credentials → OAuth 2.0 Client ID**
   - Application type: **Desktop app**
5. Download the JSON file and save it as `credentials.json` in this folder

> **First run only:** a browser window will open asking you to authorise the app.  
> A `token.json` file is saved automatically for future runs.

### 3. Prepare your contacts

Edit `contacts.csv`:

```csv
email,name,company,role
alice@acme.com,Alice Johnson,Acme Corp,VP of Engineering
```

Add as many columns as you like — all become available as `{{column_name}}` in the template.

### 4. Customise the email template

Open `email_template.html` and edit:

- Replace `YourBrand` and `Alex Rivera` with your details
- Update the CTA link (`https://calendly.com/…`)
- Adjust copy, colours, features list

All `{{variable}}` placeholders are filled from the CSV at send time.

### 5. (Optional) Add PDF attachments

Place PDF files in the `attachments/` folder. The script looks for files in this order:

| File name | Matched when |
|---|---|
| `alice@acme.com.pdf` | Exact email match |
| `acme_corp.pdf` | Company name (lowercased, spaces → `_`) |
| `default.pdf` | Fallback for everyone |

If no match is found, the email is sent without an attachment.

### 6. Run the campaign

```bash
# Normal send
python gmail_campaign.py

# Preview only — no emails sent
python gmail_campaign.py --dry-run

# Resend to everyone (ignore send log)
python gmail_campaign.py --resend-all
```

---

## Configuration

Open `gmail_campaign.py` and adjust the constants near the top:

| Constant | Description |
|---|---|
| `SENDER_NAME` | Display name in the `From:` field |
| `SUBJECT` | Subject line template (supports `{name}`, `{company}`, etc.) |
| `CONTACTS_CSV` | Path to the contacts file |
| `TEMPLATE_FILE` | Path to the HTML template |
| `ATTACHMENTS_DIR` | Folder containing PDFs |
| `LOG_FILE` | Path for the send log CSV |

---

## Send Log

Every attempt is appended to `send_log.csv`:

| Column | Description |
|---|---|
| `timestamp` | UTC ISO-8601 timestamp |
| `email` | Recipient address |
| `name` | Recipient name |
| `company` | Company |
| `subject` | Final rendered subject line |
| `attachment` | PDF filename (empty if none) |
| `status` | `sent` / `failed` / `dry_run` |
| `error` | Error message if status is `failed` |

---

## File Structure

```
gmail_campaign/
├── gmail_campaign.py       # Main script
├── email_template.html     # HTML email template
├── contacts.csv            # Contact list
├── requirements.txt        # Python dependencies
├── README.md               # This file
├── credentials.json        # ← YOU provide this (Google Cloud Console)
├── token.json              # Auto-generated after first OAuth login
├── send_log.csv            # Auto-generated on first send
└── attachments/            # Optional PDFs
    ├── alice@acme.com.pdf
    ├── acme_corp.pdf
    └── default.pdf
```

---

## Security Notes

- `credentials.json` and `token.json` contain sensitive OAuth data — **do not commit them to git**.
- Add both to `.gitignore`:
  ```
  credentials.json
  token.json
  send_log.csv
  ```
- The script requests only the `gmail.send` scope — it cannot read your emails.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `credentials.json not found` | Download it from Google Cloud Console (step 2 above) |
| `Access blocked: app not verified` | In Google Cloud Console → OAuth consent screen → add your email as a test user |
| `HttpError 429` | Gmail send quota exceeded (500/day for regular accounts) — wait and retry |
| Template variables not replaced | Check that CSV column names match the `{{placeholder}}` names exactly |
