# 📄 SharePoint Retrieval-Augmented Generation (RAG) 
      System

This project provides a **secure, real-time question-answering system** over SharePoint documents. It uses Retrieval-Augmented Generation with a vector database (ChromaDB), live webhook updates, and a Streamlit interface deployed on AWS EC2.

---

## 🚀 Overview

The SharePoint RAG system enables you to:
✅ Ask natural language questions about your SharePoint files.  
✅ Receive accurate answers based on up-to-date document embeddings.  
✅ Benefit from automatic indexing whenever files are added, edited, or deleted in SharePoint.

---

## ✅ Requirements

- Python 3.10+
- Access to SharePoint site + Microsoft Graph app credentials
- OpenAI or OpenRouter API key for embeddings and LLM responses
- (Optional for production) AWS EC2 or similar Linux server

---

## 🔐 Environment Variables

Create a `.env` file (based on `.env.example`) with:

```env
# LLM provider: openai OR openrouter
LLM_PROVIDER=
OPENAI_API_KEY=
OPENROUTER_API_KEY=

# SharePoint Graph app credentials
TENANT_ID=
CLIENT_ID=
CLIENT_SECRET=
SITE_URL_NEW=

# SharePoint username/password auth
USERNAME=
PASSWORD=

# Public URL of your FastAPI server
SERVER_URL=
```

---

## ⚙️ Setup Instructions

1. **Clone the repository**
   ```bash
   git clone https://github.com/sab110/Sharepoint_RAG.git
   cd Sharepoint_RAG
   ```

2. **Set up a virtual environment & install dependencies**
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

3. **Configure `.env`**  
   Fill in client-specific credentials.

4. **Launch the Streamlit UI locally**
   ```bash
   streamlit run streamlit_app.py --server.port 8501
   ```

5. **(Optional) Run webhook listener locally**
   ```bash
   uvicorn webhook_listener:app --host 0.0.0.0 --port 8000
   ngrok http 8000
   ```
   **Note:** Re-run `register_subscription.py` if the SharePoint webhook subscription expires.

---

## 🔗 SharePoint Integration

- Registers real-time webhooks with Microsoft Graph API.
- Processes file changes via `webhook_listener.py`.
- Automatically reindexes documents with `create_vectordb.py`.
- Tracks processed files in `processed_files.json` to avoid redundant embeddings.

---

## 🌐 Live Deployment

**Domain:** [https://aisharepoint.duckdns.org/](https://aisharepoint.duckdns.org/)  
**Webhook endpoint:** [https://aisharepoint.duckdns.org/webhook](https://aisharepoint.duckdns.org/webhook)

---

## 📁 Directory Structure

```
/home/ubuntu/app
├── streamlit_app.py
├── webhook_listener.py
├── create_vectordb.py
├── register_subscription.py
├── requirements.txt
├── processed_files.json
├── .env
├── chroma_db/          # Chroma vector DB storage
├── venv/               # Python virtual environment
└── logs/               # Streamlit restart logs
```

---

## ⚙️ AWS EC2 Setup

- Provisioned Ubuntu instance with a fixed public IP.
- Security groups open on ports: 22 (SSH), 80 (HTTP), 443 (HTTPS), 8000 (FastAPI), 8501 (Streamlit).
- Domain registered on DuckDNS: `aisharepoint.duckdns.org`.

---

## 🌐 Nginx + SSL

- Proxies:
  - `/` → Streamlit server at `localhost:8501`.
  - `/webhook` → FastAPI server at `localhost:8000`.
- SSL certificates issued with Let’s Encrypt Certbot.
- HTTP → HTTPS redirection enforced.

---

## 🔄 Automation with Systemd

Three services ensure continuous operation:

- **streamlit.service**  
  Runs the Streamlit app in the background.

- **fastapi.service**  
  Runs the FastAPI webhook listener.  
  Runs `create_vectordb.py` before starting to ensure latest embeddings.

- **app_init.service**  
  Runs both `create_vectordb.py` and `register_subscription.py` once on boot for initialization.

**Enable & start services:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable streamlit.service fastapi.service app_init.service
sudo systemctl start streamlit.service fastapi.service app_init.service
```

---

## ⏱️ Cron Jobs

- Restart Streamlit every 5 minutes so new documents are picked up:
  ```
  */5 * * * * /usr/bin/systemctl restart streamlit.service
  ```
- Renew SharePoint webhook subscription every ~29 days:
  ```
  0 0 1,30 * * /home/ubuntu/app/venv/bin/python3 /home/ubuntu/app/register_subscription.py >> /home/ubuntu/app/logs/register_cron.log 2>&1
  ```

---

## 🔄 Real-Time SharePoint Updates

**Workflow:**
1️⃣ A user adds/edits/deletes a document in SharePoint.  
2️⃣ SharePoint sends a webhook to FastAPI `/webhook`.  
3️⃣ FastAPI runs `create_vectordb.py` to reindex documents in ChromaDB.  
4️⃣ Streamlit restarts within 5 minutes, picking up the latest vector DB for immediate Q&A updates.

---

## ✨ Key Highlights

✔ Secure HTTPS with Nginx + Let’s Encrypt  
✔ Real-time document tracking and embedding  
✔ Efficient indexing using `processed_files.json`  
✔ Automated services & cron-based maintenance  
✔ Professional domain setup via DuckDNS  
✔ Seamless Q&A experience over SharePoint documents

---

## ✅ Conclusion

This SharePoint RAG system ensures secure, reliable, and real-time document processing with seamless question-answering capabilities. By integrating Microsoft Graph webhooks, ChromaDB, and Streamlit, it enables your organization to query SharePoint content effortlessly with minimal delay and maximum accuracy.

🔗 Live UI: [https://aisharepoint.duckdns.org/](https://aisharepoint.duckdns.org/)  
🔗 Webhook: [https://aisharepoint.duckdns.org/webhook](https://aisharepoint.duckdns.org/webhook)
