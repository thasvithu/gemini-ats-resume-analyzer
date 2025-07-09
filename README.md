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
- pdf2image & Pillow for PDF to image conversion  
- python-dotenv for environment variables

## Setup Instructions

1. Clone the repo  
2. Create a virtual environment and activate it  
3. Install dependencies from `requirements.txt`  
4. Add your Google API key in `.env` or Streamlit Secrets  
5. Run `streamlit run app.py`

## Usage

Open the app, paste the job description, upload your resume PDF, and choose the analysis you want!

---

### Developed by [Vithusan.V](https://github.com/thasvithu)
