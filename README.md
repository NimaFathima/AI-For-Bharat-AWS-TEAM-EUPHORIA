# Artisan AI - Voice-First Marketplace for Indian Artisans

Built jointly by **Team EUPHORIA** - **Nima Fathima** and **Sakhil N Maju** - for the **AI for Bharat hackathon (AWS), 2026**.

We both committed to this project. The shared repository lives under Sakhil's account at [Sakhil-N-Maju/AWS_ARTISAN](https://github.com/Sakhil-N-Maju/AWS_ARTISAN); this repository is a copy under my account. The marketplace UI is carried over from an earlier version of the same idea built on Next.js, Prisma and OpenAI, which we also built together: [Sakhil-N-Maju/indian_artisans](https://github.com/Sakhil-N-Maju/indian_artisans) (the commit history there shows contributions from both of us).

---

## The problem
Millions of Indian artisans struggle to sell online: most speak regional languages, are not comfortable with e-commerce tools, and cannot easily write professional product listings. Their craft stories rarely reach buyers.

## The idea
An artisan sends a **photo and a voice note** on WhatsApp in their own language. The backend transcribes the voice, analyses the photo, drafts a product listing (title, description, artisan story, cultural context, tags, suggested price) and sends a preview back for the artisan to approve or edit. Approved products appear on a web marketplace, each with a QR code that links to a verification page.

```
Artisan (WhatsApp: photo + voice)
        |
   Twilio WhatsApp webhook
        |
   Express + TypeScript backend --- PostgreSQL (Prisma)
        |
   AWS Transcribe  -> text
   AWS Rekognition -> labels and colours from the photo
   Listing generator -> Bedrock (Claude 3 Sonnet) service, or a built-in mock
   AWS S3          -> photos, audio, QR codes
        |
   Next.js marketplace (customers) + admin pages
```

## What actually works (honest status)

| Part | Status |
|---|---|
| WhatsApp webhook via Twilio sandbox: receive photo/voice, reply in 7 languages | Worked in testing |
| Backend API: products, artisans, auth, QR code generation, Prisma schema | Built |
| AWS S3 upload | Tested, passing (7 March 2026) |
| AWS Rekognition image analysis | Tested, passing (7 March 2026) |
| AWS Transcribe (voice to text) | Code written; **the AWS account was not subscribed to it**, so it was never verified end to end |
| AWS Bedrock (Claude 3 Sonnet) | Service code written; **blocked by an AWS account/payment error** during the hackathon, so never verified |
| Listing generation in the demo | Uses a **mock generator that returns template listings**. In the current code the mock is always on (`USE_MOCK_BEDROCK` is `... \|\| true` in `ai-pipeline.service.ts`), so the Bedrock path never runs until that line is changed |
| Marketplace web app (`frontend-new`) | Built: shop, products, artisans, cart, stories, workshops, community, admin and dashboard pages. Falls back to mock data when the backend is unreachable |
| Payments | Razorpay order, payment-verification and webhook routes exist in `frontend-new/app/api` (carried over from the earlier version); not verified in this project |

So the integrations are implemented, but the voice-to-listing path was **not demonstrated with live Transcribe and Bedrock**. An earlier version of this README listed speed, accuracy and Lighthouse numbers; they were removed because they were never measured on the live pipeline.

## Languages
- WhatsApp replies (acknowledgement, listing preview, errors): English, Hindi, Tamil, Telugu, Malayalam, Bengali, Gujarati.
- Transcription language mapping covers Hindi, Malayalam, Tamil, Telugu, Bengali and English (India). Gujarati is not mapped, so it would fall back to Hindi. Audio is assumed to be OGG.

## Tech stack
- **Marketplace:** Next.js 16, React 19, TypeScript, Tailwind CSS 4, Radix UI / shadcn, Recharts, Vitest
- **Backend:** Node.js, Express 5, TypeScript, Prisma 7, PostgreSQL, Winston, Zod, JWT
- **AWS SDK clients:** Transcribe, Rekognition, Bedrock Runtime, S3, Translate (dependency present)
- **Messaging:** Twilio WhatsApp API (sandbox for testing)
- **Tooling:** Docker for PostgreSQL, ngrok for the webhook, Render and Vercel configs, optional Terraform in `infrastructure/`

## Project structure
```
backend/         Express API: config, controllers, services (AWS, WhatsApp, AI pipeline), routes, middleware, Prisma
frontend-new/    Next.js marketplace and admin UI (the active frontend)
frontend/        earlier frontend
infrastructure/  optional Terraform configs
```

## Run it locally
Requirements: Node.js 20+, Docker Desktop. For the real AI path you also need an AWS account with Transcribe and Bedrock access, and a Twilio WhatsApp sandbox.

```bash
git clone https://github.com/NimaFathima/AI-For-Bharat-AWS-TEAM-EUPHORIA.git
cd AI-For-Bharat-AWS-TEAM-EUPHORIA

# backend
cd backend
npm install
# create .env with: AWS_REGION, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_S3_BUCKET,
# TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, TWILIO_WHATSAPP_NUMBER, DATABASE_URL, JWT_SECRET, FRONTEND_URL
# optional: TEST_ARTISAN_PHONE (phone used by the built-in mock test artisan)
docker-compose up -d          # PostgreSQL
npx prisma migrate dev
npm run prisma:seed           # sample data - read prisma/seed.ts and change any default credentials
npm run dev                   # http://localhost:3001

# frontend (new terminal)
cd frontend-new
npm install
# copy .env.local.example to .env.local and fill it in
npm run dev                   # http://localhost:3000
```

### Try the WhatsApp flow
1. `ngrok http 3001`
2. In the Twilio console (WhatsApp sandbox) set the webhook to `https://<your-ngrok-url>/api/whatsapp/webhook`.
3. Send a photo and a voice note from a registered artisan phone to the sandbox number.
4. You should get an acknowledgement and then a listing preview to approve or edit.

To use real Bedrock, get model access in your AWS account and change the `USE_MOCK_BEDROCK` line in `backend/src/services/ai-pipeline.service.ts`.

## Known limitations
- Transcribe and Bedrock were blocked by AWS account issues during the hackathon; demo listings came from the mock generator.
- The mock switch cannot be turned off from the environment (see above).
- WhatsApp works only through the Twilio **sandbox**, not an approved production number.
- No automated tests for the backend or the AI path (the frontend has a Vitest setup).
- Gujarati speech is not mapped for transcription.

## Roadmap
- Verify the full pipeline with live Transcribe and Bedrock; measure latency and transcription quality on real artisan recordings.
- Make the mock/real switch configurable.
- Test the payment and order flow end to end.
- Artisan verification workflow, approved WhatsApp Business number.

## Team
Both of us worked on this project and its earlier version:
- **Nima Fathima** - [@NimaFathima](https://github.com/NimaFathima)
- **Sakhil N Maju** - [@Sakhil-N-Maju](https://github.com/Sakhil-N-Maju)

B.Tech CSE (AI), Adi Shankara Institute of Engineering and Technology, Kalady.
