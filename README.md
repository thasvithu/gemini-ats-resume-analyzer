# ATS Resume Analyzer

An AI-powered Resume Analyzer that evaluates how well your resume matches a job description using Google Gemini's vision and text models, built with Streamlit.

## Features

- Upload your resume PDF  
- Paste any job description  
- Get HR-style resume review  
- Get ATS match percentage & keyword suggestions  
- Download analysis reports

## Tech Stack

- Google Gemini API (gemini-1.5-flash)  
- Streamlit for frontend UI  
- PyMuPDF (fitz) & Pillow for PDF rendering and page previews (no OS-level poppler required)  
- python-dotenv for environment variables

## Setup Instructions

1. Clone the repo  
2. Create a virtual environment and activate it  
3. Install dependencies from `requirements.txt`  
4. Add your Google API key in `.env` or Streamlit Secrets  
5. Run `streamlit run app.py`

## Usage

Open the app, paste the job description, upload your resume PDF, and choose the analysis you want!

## Deployment Guides

Below are proven paths for AWS Elastic Beanstalk, AWS EC2, and Azure App Service. Choose the one that fits your infra.

### Common requirements

- Python 3.10
- Environment variable: `GOOGLE_API_KEY` (Gemini API key)
- Outbound internet access to Google APIs
- For production: enable HTTPS and protect API keys

---

### Deploying to AWS Elastic Beanstalk (Python Platform)

Works well for quick hosting; Streamlit listens on the EB-provided port.

1) Create an EB environment using the Python 3.11 AL2023 platform.
2) Configure Environment properties:
	- `GOOGLE_API_KEY` = your Gemini API key
3) Package and deploy:
	- Zip the repository contents (root-level files, not a parent folder)
	- Upload the ZIP in the EB console and deploy
4) Health checks:
	- If you need a health check path, use `/` or Streamlit’s path `/` and ensure security groups and load balancer health checks are aligned.

Notes:
- This app now uses PyMuPDF, so no `poppler-utils` is required.
- If you previously used a `Procfile`, you can keep: `web: sh -c "streamlit run app.py --server.port $PORT --server.address 0.0.0.0"`.
- Check logs: `/var/log/web.stdout.log`, `/var/log/web.stderr.log`, `/var/log/nginx/error.log`, `/var/log/eb-engine.log`.

---

### Deploying to AWS EC2 (manual, no Docker)

Use EC2 when you want full control. Here’s a minimal setup on Amazon Linux 2023 or Ubuntu.

1) Provision EC2, open inbound security group port (e.g., 80 via Nginx, or 8501/5000 if direct Streamlit).
2) SSH to the instance and install runtime:
	- Python 3.10, pip, git
3) App setup:
	- Clone repo; create venv; `pip install -r requirements.txt`
	- Set `GOOGLE_API_KEY` in shell profile or systemd unit
	- Test run: `streamlit run app.py --server.port 8501 --server.address 0.0.0.0`
4) Optional production hardening:
	- Create a systemd service for Streamlit
	- Install Nginx, configure reverse proxy from 80 → 8501, obtain TLS via Let’s Encrypt

Basic systemd unit example (adjust paths/users):
```
[Unit]
Description=ATS Resume Analyzer (Streamlit)
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/opt/gemini-ats-resume-analyzer
Environment=GOOGLE_API_KEY=YOUR_KEY
ExecStart=/opt/gemini-ats-resume-analyzer/.venv/bin/streamlit run app.py --server.port 8501 --server.address 0.0.0.0
Restart=always

[Install]
WantedBy=multi-user.target
```

---

---

### Developed by [Vithusan.V](https://github.com/thasvithu)

### Deploying to Azure App Service (Code) with CI/CD (No Docker)

This repo is configured to deploy as a standard Python app to Azure App Service (Linux) without using containers. The GitHub Actions workflow builds with Oryx and deploys your code directly.

What you'll provision in Azure:

- Azure Resource Group (e.g., `rg-ats-resume`)
- App Service Plan (Linux)
- Web App (Code) (e.g., `gemini-ats-analyzer-app`)

Repo files used:

- `.github/workflows/azure-webapp-code.yml` — CI/CD pipeline for code-based deploy
- `requirements.txt` — Python dependencies (uses PyMuPDF instead of pdf2image to avoid OS packages)

Secrets to set in GitHub (Repo → Settings → Secrets and variables → Actions):

- `AZURE_CLIENT_ID` — Federated credentials-enabled app registration's client ID
- `AZURE_TENANT_ID` — Your Azure AD tenant ID
- `AZURE_SUBSCRIPTION_ID` — Azure subscription ID
- `GOOGLE_API_KEY` — Gemini API key used at runtime in the Web App

Update workflow env values if needed:

```
AZURE_WEBAPP_NAME: gemini-ats-analyzer-app
AZURE_RESOURCE_GROUP: rg-ats-resume
PYTHON_VERSION: '3.10'
```

First-time Azure setup (one-time):

1) Create an App Service Plan (Linux) and a Web App (Code) targeting Linux.
2) Configure Federated Credentials for GitHub OIDC on the AAD app and grant the app access to the subscription/resource group.
3) In the Web App → Configuration → Application settings, the workflow sets `GOOGLE_API_KEY`, or you can set it manually.

Deploy via CI/CD:

1) Push to `main` (or trigger the workflow manually).
2) GitHub Actions logs into Azure, sets app settings, and deploys code using `azure/webapps-deploy`.

Runtime notes:

- Streamlit is started by the platform using the Oryx-generated startup. If you need a custom startup command, set it in Azure Portal → Configuration → General settings → Startup command, e.g.:
	`streamlit run app.py --server.port $PORT --server.address 0.0.0.0`
- PyMuPDF is used for PDF rendering, so no OS-level Poppler dependency is required.

Troubleshooting:

- Check Log stream for app output/errors.
- If startup fails, set the Startup command as above.
- Ensure `GOOGLE_API_KEY` is configured in App Settings.

