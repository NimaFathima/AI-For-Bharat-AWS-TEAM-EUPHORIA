# Artisan AI - Voice-First Marketplace for Indian Artisans

Built by **Team EUPHORIA** (Sakhil N Maju and Nima Fathima) for the **AI for Bharat hackathon (AWS), 2026**.

The main development repository is [Sakhil-N-Maju/AWS_ARTISAN](https://github.com/Sakhil-N-Maju/AWS_ARTISAN); this repository keeps a copy under my account. An earlier version of the same idea, built on OpenAI, is [Sakhil-N-Maju/indian_artisans](https://github.com/Sakhil-N-Maju/indian_artisans), where I added the Prisma schema and the first AI product-creation pipeline.

---

## The problem
Millions of Indian artisans struggle to sell online: most speak regional languages, are not comfortable with e-commerce tools, and cannot easily write professional product listings. Their craft stories rarely reach buyers.

## The idea
An artisan sends a **photo and a voice note** on WhatsApp in their own language. The system transcribes the voice, looks at the photo, drafts a product listing (title, description, story, price suggestion, tags), and sends a preview back for the artisan to approve, edit or reject. Approved products appear on a web marketplace with a QR code for authenticity.

```
Artisan (WhatsApp: photo + voice)
        |
   Twilio WhatsApp webhook
        |
   Express + TypeScript backend --- PostgreSQL (Prisma)
        |
   AWS Transcribe  -> text (hi, ml, ta, te, bn, en-IN)
   AWS Rekognition -> labels, colours, quality of the photo
   AWS Bedrock (Claude 3 Sonnet) -> listing draft
   AWS S3          -> photos, audio, QR codes
        |
   Next.js marketplace (customers) + admin panel
```

## What actually works (honest status)

| Part | Status |
|---|---|
| WhatsApp webhook via Twilio sandbox (receive photo/voice, reply with messages) | Working in testing |
| Backend API: products, artisans, admin login, QR verification, Prisma schema and seed data | Built |
| AWS S3 upload | Tested, passing (7 March 2026) |
| AWS Rekognition image analysis | Tested, passing (7 March 2026) |
| AWS Transcribe (voice to text) | Code written; **account was not subscribed**, so not verified end to end |
| AWS Bedrock (Claude 3 Sonnet) | Code written; **blocked by an AWS account/payment error** during the hackathon, so not verified end to end |
| AI listing generation in the demo | Ran with a **mock AI** (`USE_MOCK_BEDROCK=true`) that returns template listings |
| Marketplace web app | Built, with a mock-data fallback when the backend is unreachable; Vercel and Render deployment configs included |

So the pipeline and integrations are implemented, but the voice-to-listing path was **not demonstrated with live Transcribe and Bedrock**. I have removed the speed, accuracy and Lighthouse numbers that an earlier version of this README listed, because they were not measured on the live pipeline.

## Tech stack
- **Frontend:** Next.js 14, TypeScript, Tailwind CSS
- **Backend:** Node.js 20, Express, TypeScript, Prisma ORM, PostgreSQL, Winston logging
- **AWS:** Transcribe, Rekognition, Bedrock, S3 (region ap-south-1)
- **Messaging:** Twilio WhatsApp Business API (sandbox for testing)
- **Tooling:** Docker, ngrok (to expose the local webhook), Render and Vercel configs

## Project structure
```
backend/         Express API: config, controllers, services (AWS integrations), routes, middleware, Prisma
frontend-new/    Next.js 14 marketplace and admin UI
frontend/        earlier frontend
infrastructure/  optional Terraform configs
```

## Run it locally
Requirements: Node.js 20+, Docker Desktop, and (for the real AI path) an AWS account with Transcribe and Bedrock access plus a Twilio WhatsApp sandbox.

```bash
git clone https://github.com/NimaFathima/AI-For-Bharat-AWS-TEAM-EUPHORIA.git
cd AI-For-Bharat-AWS-TEAM-EUPHORIA

# backend
cd backend
npm install
# create .env with: AWS_REGION, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_S3_BUCKET,
# TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, TWILIO_WHATSAPP_NUMBER, DATABASE_URL, JWT_SECRET
# set USE_MOCK_BEDROCK=true to run without Bedrock access
docker-compose up -d          # PostgreSQL
npx prisma migrate dev
npm run prisma:seed           # sample artisans and products (development only)
npm run dev                   # http://localhost:3001

# frontend (new terminal)
cd frontend-new
npm install
npm run dev                   # http://localhost:3000
```

### Try the WhatsApp flow
1. `ngrok http 3001`
2. In the Twilio console (WhatsApp sandbox), set the webhook to `https://<your-ngrok-url>/api/whatsapp/webhook`
3. Send a photo and a voice note to the sandbox number from a registered artisan phone.
4. You should get an acknowledgement and then a listing preview to approve, edit or reject.

The seed script creates a development admin account. **Change or remove it before deploying anywhere public.**

## Known limitations
- Voice transcription and Bedrock generation were blocked by AWS account issues during the hackathon, so listings in the demo came from the mock AI.
- WhatsApp works only through the Twilio **sandbox**, not an approved production number.
- No payment flow, order management or shipping. Those are roadmap items.
- No automated test suite for the AI path.

## Roadmap
- Verify the full pipeline with live Transcribe and Bedrock and measure latency and transcription quality on real artisan recordings.
- Payments (Razorpay), orders and shipping.
- Artisan verification workflow and multi-language customer UI.
- Move from the Twilio sandbox to an approved WhatsApp Business number.

## Team
- **Sakhil N Maju** - repository owner, deployment
- **Nima Fathima** - [@NimaFathima](https://github.com/NimaFathima)

B.Tech CSE (AI), Adi Shankara Institute of Engineering and Technology, Kalady.
