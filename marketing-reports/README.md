# Rentokil PCI HiCare — Automated Marketing Reports

Automated daily, weekly, and monthly performance reporting system for **PCI HiCare** (https://pci-hicare.com).

Pulls data from Google Ads, Meta Ads, Shopify, and Google Analytics 4 → generates branded HTML email reports → sends WhatsApp summaries to stakeholders.

---

## Project Structure

```
marketing-reports/
├── config/
│   ├── credentials.yaml          # API keys & secrets (NOT committed to git)
│   └── settings.yaml             # Report schedule, thresholds, appearance
├── src/
│   ├── fetchers/
│   │   ├── google_ads.py         # Google Ads API data fetcher
│   │   ├── meta_ads.py           # Meta (Facebook) Ads API fetcher
│   │   ├── shopify.py            # Shopify Admin API fetcher
│   │   └── ga4.py                # Google Analytics 4 fetcher
│   ├── processors/
│   │   ├── aggregator.py         # Combines data across platforms
│   │   └── metrics.py            # KPI calculations (ROAS, AOV, CTR, etc.)
│   ├── reports/
│   │   ├── daily.py              # Daily report generator
│   │   ├── weekly.py             # Weekly report generator
│   │   └── monthly.py            # Monthly report generator
│   ├── templates/
│   │   ├── daily_report.html     # Jinja2 HTML email template (daily)
│   │   ├── weekly_report.html    # Jinja2 HTML email template (weekly)
│   │   └── monthly_report.html   # Jinja2 HTML email template (monthly)
│   ├── senders/
│   │   ├── email_sender.py       # SMTP email delivery
│   │   └── whatsapp_sender.py    # WhatsApp Cloud API / Twilio
│   └── utils/
│       ├── logger.py             # Centralised logging setup
│       └── helpers.py            # Date ranges, formatting utilities
├── schedulers/
│   └── cron_jobs.py              # APScheduler — orchestrates everything
├── data/
│   └── cache/                    # Temporary API response cache
├── logs/                         # Rotating log files
├── tests/                        # Unit & integration tests
├── .env.example                  # Environment variable template
├── .gitignore
├── requirements.txt
└── README.md
```

---

## APIs Used

| Platform | Purpose | Credentials Needed |
|---|---|---|
| Google Ads API | Ad spend, clicks, impressions, conversions | Developer Token, OAuth2 Client ID/Secret, Refresh Token, Customer ID |
| Meta Marketing API | Facebook/Instagram ad spend, reach, ROAS | App ID, App Secret, Long-lived Access Token, Ad Account ID |
| Shopify Admin REST API | Orders, revenue, AOV, top products, funnel | Private App Access Token |
| Google Analytics 4 | Sessions, add-to-cart, checkout, purchase events | GA4 Property ID, Service Account JSON |
| Gmail / SMTP | Send HTML email reports | SMTP credentials / Gmail App Password |
| WhatsApp Cloud API | WhatsApp summary messages | Meta Phone Number ID + Access Token |

---

## Step-by-Step Setup

### Prerequisites

- Python **3.10 or higher**
- A terminal / command prompt
- Access to at least one of the APIs above

---

### Step 1 — Clone & enter the project

```bash
git clone <your-repo-url>
cd marketing-reports
```

---

### Step 2 — Create a Python virtual environment

```bash
python3 -m venv .venv

# Activate it:
source .venv/bin/activate          # macOS / Linux
.venv\Scripts\activate             # Windows
```

---

### Step 3 — Install dependencies

```bash
pip install -r requirements.txt
```

---

### Step 4 — Set up credentials

```bash
# Copy the example files
cp .env.example .env
cp config/credentials.yaml.example config/credentials.yaml   # or edit directly
```

Edit `.env` and/or `config/credentials.yaml` and fill in your real API keys.
See the per-API instructions below.

---

### Step 5 — Obtain API Credentials

#### 5A — Google Ads API

1. Go to https://developers.google.com/google-ads/api/docs/get-started/introduction
2. Apply for a **Developer Token** in your Google Ads Manager account
3. Create an **OAuth 2.0 Client ID** in Google Cloud Console:
   - APIs & Services → Credentials → Create Credentials → OAuth 2.0 Client ID
   - Application type: **Desktop app**
   - Download the JSON — copy `client_id` and `client_secret`
4. Generate a **Refresh Token** using the OAuth2 flow:
   ```bash
   python -c "
   from google_auth_oauthlib.flow import InstalledAppFlow
   flow = InstalledAppFlow.from_client_secrets_file('client_secrets.json',
       scopes=['https://www.googleapis.com/auth/adwords'])
   creds = flow.run_local_server()
   print('Refresh token:', creds.refresh_token)
   "
   ```
5. Find your **Customer ID** in the top-right of Google Ads UI (format: XXX-XXX-XXXX)
6. Fill in `config/credentials.yaml` → `google_ads` section

#### 5B — Meta (Facebook) Ads API

1. Go to https://developers.facebook.com → Create a **new App** (type: Business)
2. Add the **Marketing API** product
3. Get your **App ID** and **App Secret** from App Settings → Basic
4. Generate a **Long-lived Access Token**:
   - Short token: https://developers.facebook.com/tools/explorer/
   - Exchange for long-lived (60 days):
     ```
     GET https://graph.facebook.com/oauth/access_token
       ?grant_type=fb_exchange_token
       &client_id={app-id}
       &client_secret={app-secret}
       &fb_exchange_token={short-token}
     ```
5. Find your **Ad Account ID** in Meta Business Manager → Accounts → Ad Accounts (format: `act_123456`)
6. Fill in `config/credentials.yaml` → `meta_ads` section

#### 5C — Shopify Admin API

1. In Shopify Admin → **Settings → Apps and sales channels → Develop apps**
2. Create a new **custom app** → name it "Marketing Reports"
3. Under **Configuration** → Admin API access scopes, enable:
   - `read_orders`
   - `read_products`
   - `read_analytics`
4. Install the app → copy the **Admin API access token** (shown once)
5. Fill in `config/credentials.yaml` → `shopify` section

#### 5D — Google Analytics 4

1. Go to https://console.cloud.google.com
2. Enable the **Google Analytics Data API**
3. Create a **Service Account**:
   - IAM & Admin → Service Accounts → Create
   - Role: **Viewer**
   - Create a JSON key → download as `config/ga4_service_account.json`
4. In GA4 Admin → **Property Access Management** → Add the service account email with **Viewer** role
5. Copy your **Property ID** from GA4 Admin → Property Settings
6. Fill in `config/credentials.yaml` → `google_analytics` section

#### 5E — Gmail SMTP (Email)

1. Enable **2-Step Verification** on your Google account
2. Go to https://myaccount.google.com/apppasswords
3. Generate an **App Password** for "Mail" → copy the 16-character password
4. Use this as `SMTP_PASSWORD` (not your regular Gmail password)

#### 5F — WhatsApp Cloud API

**Option A — Meta WhatsApp Cloud API (Free)**
1. Go to https://developers.facebook.com → your App → Add **WhatsApp** product
2. Under WhatsApp → Getting Started:
   - Note the **Phone Number ID**
   - Note the **Temporary Access Token** (or create a permanent System User token)
3. Add recipient numbers to the API test list
4. Fill in `config/credentials.yaml` → `whatsapp.meta` section

**Option B — Twilio WhatsApp**
1. Sign up at https://www.twilio.com
2. Activate the **WhatsApp Sandbox** (or apply for a dedicated number)
3. Copy **Account SID** and **Auth Token** from the Twilio Console
4. Fill in `config/credentials.yaml` → `whatsapp.twilio` section

---

### Step 6 — Configure settings

Edit `config/settings.yaml`:

- **`general.timezone`** — set to `Asia/Kolkata` (already set)
- **`schedule`** — adjust cron times for daily/weekly/monthly reports
- **`thresholds`** — set your ROAS, CTR, CPC targets
- **`email_template.recipients`** — add all stakeholder emails per report type

---

### Step 7 — Run a manual test report

```bash
# Test daily report (yesterday's data)
python -m src.reports.daily --date yesterday

# Test weekly report (last 7 days)
python -m src.reports.weekly

# Test monthly report (last 30 days)
python -m src.reports.monthly
```

---

### Step 8 — Start the scheduler

```bash
# Run the scheduler (keeps running — use screen/tmux/systemd in production)
python schedulers/cron_jobs.py
```

---

### Step 9 — Production deployment options

#### Option A — Linux systemd service
```ini
# /etc/systemd/system/marketing-reports.service
[Unit]
Description=PCI HiCare Marketing Reports Scheduler
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/marketing-reports
ExecStart=/opt/marketing-reports/.venv/bin/python schedulers/cron_jobs.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl enable marketing-reports
sudo systemctl start marketing-reports
```

#### Option B — GitHub Actions (cloud, no server needed)
Schedule via `.github/workflows/daily_report.yml` using `on: schedule`.

#### Option C — Google Cloud Run / AWS Lambda
Package as a container and trigger via Cloud Scheduler / EventBridge.

---

## Key Metrics Tracked

| Metric | Source | Report Frequency |
|---|---|---|
| Ad Spend (Google + Meta split) | Google Ads, Meta Ads | Daily, Weekly, Monthly |
| Revenue & ROAS | Meta Ads, Shopify | Daily, Weekly, Monthly |
| Orders & AOV | Shopify | Daily, Weekly, Monthly |
| CTR, CPC, Impressions | Google Ads, Meta Ads | Daily, Weekly, Monthly |
| Sessions → Add to Cart → Checkout → Purchase | GA4 | Daily, Weekly, Monthly |
| Top 10 Campaigns | Google Ads, Meta Ads | Weekly, Monthly |
| Top 10 Products | Shopify | Weekly, Monthly |

---

## Troubleshooting

| Error | Solution |
|---|---|
| `google.ads.googleads.errors.GoogleAdsException` | Check Developer Token approval status and OAuth scopes |
| `facebook_business.exceptions.FacebookRequestError` | Refresh your long-lived token (expires every 60 days) |
| `ShopifyAPI.errors.ClientError: 401` | Regenerate the private app access token in Shopify |
| `SMTPAuthenticationError` | Use a Gmail App Password, not your regular password |
| `ConnectionRefusedError` on WhatsApp | Check phone number ID and that recipient numbers are verified |

---

## Security Notes

- **Never commit** `.env` or `config/credentials.yaml` — both are in `.gitignore`
- Rotate tokens regularly (especially Meta's long-lived token, which expires in 60 days)
- Use a service account with minimum required permissions for GA4
- Consider using **Google Cloud Secret Manager** or **AWS Secrets Manager** for production

---

## License

Internal tool — Rentokil PCI HiCare. Not for public distribution.
