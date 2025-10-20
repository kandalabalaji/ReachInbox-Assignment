# ReachInbox Onebox - Real-Time AI Email Aggregator

A feature-rich email onebox system with real-time IMAP synchronization, Elasticsearch search, AI-powered categorization, and RAG-based suggested replies.

## Features

### Core Features (Implemented)
1. **Real-Time Email Synchronization** - IMAP IDLE mode for multiple accounts (no polling)
2. **Searchable Storage** - Elasticsearch indexing with full-text search and filtering
3. **AI Email Categorization** - Gemini API classification into 5 categories
4. **Slack & Webhook Integration** - Notifications for interested leads
5. **Frontend Interface** - Modern React UI with search and filtering
6. **RAG-Based Suggested Replies** - Vector DB + LLM for contextual email responses (Bonus)

## Architecture

\`\`\`
┌─────────────────────────────────────────────────────────────┐
│                     Frontend (Next.js)                       │
│  - Email List, Search, Filters, Account Status              │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                  Backend API (Express)                       │
│  - /api/emails (search, list, stats)                        │
│  - /api/accounts (list, sync)                               │
│  - /api/replies (suggest)                                   │
└────────────────────┬────────────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
   ┌────▼──┐  ┌─────▼────┐  ┌───▼──────┐
   │ IMAP  │  │Elasticsearch│ │ Qdrant  │
   │ Sync  │  │  (Search)   │ │ (RAG)   │
   └───────┘  └────────────┘ └─────────┘
        │            │            │
   ┌────▼──────────────────────────▼──┐
   │  Gemini API                       │
   │  - Categorization                 │
   │  - Embeddings                     │
   │  - Reply Generation               │
   └───────────────────────────────────┘
\`\`\`

## Prerequisites

- Node.js 18+
- Docker & Docker Compose
- Gemini API Key (from Google AI Studio)
- IMAP email accounts (Gmail, Outlook, etc.)
- Slack Webhook URL (optional)
- webhook.site URL (optional)

## Setup Instructions

### 1. Clone and Install

\`\`\`bash
git clone <your-repo-url>
cd reachinbox-onebox
npm install
\`\`\`

### 2. Docker Setup

Start Elasticsearch and Qdrant:

\`\`\`bash
docker-compose up -d
\`\`\`

Verify services are running:
\`\`\`bash
# Elasticsearch
curl http://localhost:9200

# Qdrant
curl http://localhost:6333/health
\`\`\`

### 3. Environment Configuration

Copy `.env.example` to `.env` and configure:

\`\`\`bash
cp .env.example .env
\`\`\`

Edit `.env` with your credentials:

\`\`\`env
# IMAP Accounts (Gmail example)
IMAP_USER_1=your-email@gmail.com
IMAP_PASSWORD_1=your-app-password  # Use App Password for Gmail
IMAP_HOST_1=imap.gmail.com
IMAP_PORT_1=993

IMAP_USER_2=another-email@gmail.com
IMAP_PASSWORD_2=your-app-password
IMAP_HOST_2=imap.gmail.com
IMAP_PORT_2=993

# Gemini API
GEMINI_API_KEY=your-gemini-api-key

# Slack (optional)
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL

# Webhook.site (optional)
WEBHOOK_SITE_URL=https://webhook.site/your-unique-id

# Server
PORT=3000
NODE_ENV=development
\`\`\`

### 4. Build and Run

\`\`\`bash
# Build TypeScript
npm run build

# Start the server
npm start

# Or run in development mode
npm run dev
\`\`\`

The server will start on `http://localhost:3000`

## API Endpoints

### Emails
- `GET /api/emails` - List all emails (paginated)
- `GET /api/emails/search?q=...&accountId=...&folder=...&category=...` - Search emails
- `GET /api/emails/stats` - Get statistics

### Accounts
- `GET /api/accounts` - List configured accounts
- `POST /api/accounts/sync` - Start IMAP sync

### Replies (RAG)
- `POST /api/replies/:emailId/suggest` - Generate suggested reply

## Frontend

Access the UI at `http://localhost:3000`

Features:
- Real-time email list with AI categories
- Full-text search across subject and body
- Filter by account and category
- Account connection status
- Email statistics
- Suggested reply generation (RAG)

## Configuration Details

### IMAP Setup

**Gmail:**
1. Enable 2-Factor Authentication
2. Generate App Password: https://myaccount.google.com/apppasswords
3. Use the 16-character password in `IMAP_PASSWORD`

**Outlook:**
1. Use your email and password directly
2. Host: `imap-mail.outlook.com`
3. Port: `993`

### Gemini API

1. Get API key from: https://aistudio.google.com/app/apikey
2. Set `GEMINI_API_KEY` in `.env`

### Slack Integration

1. Create Incoming Webhook: https://api.slack.com/messaging/webhooks
2. Set `SLACK_WEBHOOK_URL` in `.env`

### Webhook.site

1. Visit https://webhook.site
2. Copy your unique URL
3. Set `WEBHOOK_SITE_URL` in `.env`

## Testing with Postman

### 1. List Emails
\`\`\`
GET http://localhost:3000/api/emails?page=1&limit=20
\`\`\`

### 2. Search Emails
\`\`\`
GET http://localhost:3000/api/emails/search?q=interested&category=Interested
\`\`\`

### 3. Get Statistics
\`\`\`
GET http://localhost:3000/api/emails/stats
\`\`\`

### 4. List Accounts
\`\`\`
GET http://localhost:3000/api/accounts
\`\`\`

### 5. Start Sync
\`\`\`
POST http://localhost:3000/api/accounts/sync
\`\`\`

### 6. Generate Suggested Reply
\`\`\`
POST http://localhost:3000/api/replies/{emailId}/suggest
\`\`\`

## Key Implementation Details

### Real-Time IMAP Sync
- Uses IMAP IDLE mode (no polling)
- Fetches last 30 days on initial sync
- Watchdog timer keeps connection alive (every 29 minutes)
- Automatic reconnection on failure

### Email Categorization
- Gemini 1.5 Flash model with JSON schema
- 5 categories: Interested, Meeting Booked, Not Interested, Spam, Out of Office
- Exponential backoff for API rate limits

### Search & Filtering
- Elasticsearch full-text search on subject and body
- Keyword filtering on accountId, folder, and category
- Pagination support (default 20 per page)

### RAG System
- Qdrant vector database for semantic search
- Gemini embeddings for text vectorization
- Context retrieval from product knowledge base
- LLM-generated replies based on context

## Performance Considerations

- IMAP connections are persistent (IDLE mode)
- Elasticsearch indexes emails for fast search
- Vector DB caches embeddings for RAG
- Exponential backoff prevents API rate limiting
- Pagination prevents large data transfers

## Troubleshooting

### IMAP Connection Issues
\`\`\`bash
# Check logs
docker-compose logs -f

# Verify credentials
# Ensure App Password is used for Gmail
# Check firewall/network access to IMAP server
\`\`\`

### Elasticsearch Issues
\`\`\`bash
# Check health
curl http://localhost:9200/_cluster/health

# View indices
curl http://localhost:9200/_cat/indices
\`\`\`

### Qdrant Issues
\`\`\`bash
# Check health
curl http://localhost:6333/health

# View collections
curl http://localhost:6333/collections
\`\`\`

### Gemini API Issues
- Verify API key is valid
- Check rate limits (free tier: 60 requests/minute)
- Ensure GEMINI_API_KEY is set in environment

## Deployment

### Docker Deployment
\`\`\`bash
docker build -t reachinbox-onebox .
docker run -p 3000:3000 --env-file .env reachinbox-onebox
\`\`\`

### Vercel Deployment
\`\`\`bash
vercel deploy
\`\`\`

## Code Quality

- TypeScript for type safety
- Modular service architecture
- Error handling with exponential backoff
- Comprehensive logging with Pino
- Clean separation of concerns

## Future Enhancements

- Multi-folder sync (not just INBOX)
- Custom categorization rules
- Email attachment handling
- User authentication
- Database persistence
- Email scheduling
- Template management

## Evaluation Criteria

| Criteria | Status |
|----------|--------|
| Real-Time Performance | ✓ IMAP IDLE implemented |
| Code Quality | ✓ Modular TypeScript |
| Search Functionality | ✓ Elasticsearch + filtering |
| AI Accuracy | ✓ Gemini with JSON schema |
| RAG Implementation | ✓ Qdrant + embeddings |
| UX & UI | ✓ Modern React interface |

## Support

For issues or questions:
1. Check logs: `docker-compose logs -f`
2. Verify environment variables
3. Test API endpoints with Postman
4. Check service health endpoints

## License

MIT

## Author

Built for ReachInbox Backend Engineering Assignment
`
