# Cloudflare Workers & Pages Deployment Guide

This project is fully structured and optimized for deployment to **Cloudflare Workers with Assets** (the modern replacement for Cloudflare Pages). 

By utilizing Wrangler, the React frontend is compiled to static assets and served globally at the edge, while the custom Cloudflare Worker (`worker/index.ts`) handles API requests, file routing (like the dynamic resume PDF download), and runs our high-performance **Cloudflare Workers AI Chatbot** directly on Cloudflare's serverless GPU infrastructure.

---

## 🏗️ Architecture Overview

1. **Frontend**: React 19, Vite, and Tailwind CSS. Built into the `dist/client` directory.
2. **Backend**: A Cloudflare Edge Worker (`worker/index.ts`) that intercepts `/api/chat` and `/resume.pdf`, falling back to static asset serving (`env.ASSETS.fetch`) for all other assets.
3. **AI Engine**: Powered by Cloudflare's **Workers AI** using the state-of-the-art `@cf/meta/llama-3.1-8b-instruct` LLM. It operates with zero API keys and minimal latency directly on Cloudflare.

---

## 🛠️ Step 1: Manual CLI Deployment

For rapid deployment or manual testing, you can deploy directly from your local terminal using Wrangler:

1. **Install Wrangler** (pre-installed in dependencies):
   ```bash
   npm install
   ```

2. **Login to Cloudflare**:
   Authenticate Wrangler with your Cloudflare account:
   ```bash
   npx wrangler login
   ```

3. **Build the Application**:
   Compile the React frontend to `dist/client`:
   ```bash
   npm run build
   ```

4. **Deploy to Cloudflare**:
   Deploy both static assets and the Edge Worker using the bundled npm script:
   ```bash
   npm run deploy
   ```
   *Wrangler will output the live URL of your deployed application (e.g., `https://jelvan-portfolio.username.workers.dev`).*

---

## 🤖 Step 2: Workers AI Activation

Cloudflare Workers AI runs with **zero API keys** but requires the AI binding to be associated with your worker.

The binding has been predefined in `/wrangler.jsonc`:
```json
"ai": {
  "binding": "AI"
}
```

When you run `npm run deploy` or build via GitHub Actions, Cloudflare will automatically bind the AI module. The worker handles this natively:
- **Primary AI Execution**: Attempts to use Cloudflare's edge-based `@cf/meta/llama-3.1-8b-instruct` model.
- **Failover / Local Development**: If running locally without AI bindings or if Cloudflare services are disrupted, the system seamlessly falls back to the server-side **Google Gemini SDK** (using your `GEMINI_API_KEY` environment secret) or a robust offline fallback heuristic.

---

## 🚀 Step 3: Automated GitHub Actions CI/CD

To set up automatic deployments whenever you push changes to your GitHub repository:

### 1. Add Cloudflare Credentials to GitHub Secrets
Navigate to your GitHub repository under **Settings > Secrets and variables > Actions** and click **New repository secret**. Add the following:

- `CLOUDFLARE_API_TOKEN`
  * **How to create**: Go to the Cloudflare Dashboard -> **My Profile > API Tokens > Create Token**. Use the **Edit Cloudflare Workers** template. Ensure the token has edit permissions for Workers, Pages, and Account-level settings.
- `CLOUDFLARE_ACCOUNT_ID`
  * **How to find**: Log into your Cloudflare Dashboard. The Account ID is a 32-character hexadecimal string shown on your dashboard home page URL or Workers overview.

### 2. Push to GitHub
A preconfigured GitHub Actions workflow has been added to `.github/workflows/deploy.yml`. 

Whenever you push to the `main` branch, GitHub Actions will:
1. Check out your code.
2. Install dependencies.
3. Build the React frontend with Vite.
4. Deploy the frontend and worker to Cloudflare globally.

---

## ⚙️ Configuration Files Reference

- **`/wrangler.jsonc`**: Contains routing rules, assets directory pointers, compatibility dates, and the `AI` worker binding.
- **`/worker/index.ts`**: The main Edge entry point that serves static files and processes API requests.
- **`.github/workflows/deploy.yml`**: The continuous delivery workflow configuration.
