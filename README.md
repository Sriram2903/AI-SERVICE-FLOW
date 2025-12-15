🤖 AI ServiceFlow – AI-Powered IT Service Desk
AI ServiceFlow is a cloud-native, AI-powered IT service desk that automates ticket creation, accelerates troubleshooting, and supports both text and voice input for a modern self-service experience. It runs on a fully serverless AWS architecture and integrates generative AI with an internal knowledge base for accurate, context-aware support.

✨ Features
🎫 Smart Ticket Creator
Users describe issues in plain language (typed or spoken via the built‑in voice assistant).

AI analyzes each description, detects multiple distinct issues in a single message, and proposes one or more structured tickets.

Automatic classification into categories such as Hardware, Software, Network, Security, Email, Mobile, and General.

Automatic assignment of priority, urgency, problem type, and responsible team.

Estimated resolution time and AI-generated quick-fix suggestions for each ticket.

🧾 Multi‑Ticket Support
A single free‑text (or spoken) prompt can generate multiple tickets.

Each generated ticket includes:

Category, priority, urgency, and problem type.

Assigned team and AI quick fixes.

Batch metadata (sequence number and total in batch) so related tickets are easy to track.

🎙️ Voice Assistant (Voice-to-Text)
Integrated microphone/voice input in the ticket creator and assistant.

Users can speak their issues; speech is converted to text and fed into the same AI analysis pipeline used for typed input.

Improves accessibility and speeds up ticket creation for busy or non-technical users.

💬 AI Troubleshooting Assistant
Chat-style interface for IT support questions.

Uses Retrieval-Augmented Generation (RAG) over an internal IT knowledge base.

Answers are grounded in your documentation and can include source citations for transparency.

Can optionally use the currently selected ticket as context to provide more targeted assistance.

📊 Ticket Dashboard
Retrieves tickets from DynamoDB and renders them in a responsive, interactive UI.

Front-end sorting and filtering by status, priority, and search text.

Clear visual indicators for AI-generated tickets and multi-ticket batch items.

🔐 Authentication & Sessions
User registration and login via AWS Cognito User Pool.

JWT-based authentication from the browser for secure API access.

Session persistence, secure logout, and protection of authenticated routes and actions.

🎨 Modern Web UI (Light/Dark Mode)
Single-page application built with HTML, CSS, and vanilla JavaScript.

Built-in light / dark mode theme toggle with smooth transitions.

User theme preference is stored and restored automatically.

Fully responsive layout for desktop, tablet, and mobile devices.

🏗️ Architecture Overview
AI ServiceFlow follows a serverless microservices pattern on AWS.

🖥️ Frontend
Static assets: index.html, styles.css, script.js.

Deployed via Amazon S3 + CloudFront or any static hosting provider.

Communicates with backend services over HTTPS using REST-style JSON APIs.

Implements:

Authentication UI and session handling.

Smart Ticket Creator (with voice input and multi-ticket preview).

Ticket dashboard.

AI chat assistant.

Knowledge base popups/modals.

Light/dark theme toggling and preference storage.

Integration with all backend Lambda functions.

☁️ Core Backend (AWS Lambda)
1️⃣ Knowledge Base Query Function – AIServiceFlow-KnowledgeBase-Query
Purpose
Answer IT support questions using an internal knowledge base.

Responsibilities

Parse the HTTP request body (message, optional ticketContext).

Build an enhanced query that includes ticket context when available.

Call AWS Bedrock retrieve_and_generate against a configured Knowledge Base (Claude 3 Haiku).

Use vector search to retrieve relevant documents and generate grounded answers.

Extract the final answer text and source citations (e.g., S3 URIs).

Return a structured JSON payload with answer, citations, model information, and flags indicating whether knowledge base context was used.

2️⃣ Ticket Creator Function – AIServiceFlow-TicketCreator
Purpose
Analyze natural language descriptions and create single or multiple tickets.

Flows

Analyze (action: "analyze"):

Detects whether the description contains multiple distinct issues.

For multi-issue inputs:

Splits text using indicators (e.g., “and also”, numbered lists) and smart sentence grouping.

Validates each segment as a ticket candidate.

Runs rule-based “AI analysis” on each segment.

Returns a set of suggested tickets (no database writes yet).

For single issues:

Analyzes the full description.

Returns analysis and suggestions (no database writes).

Create (action: "create"):

Accepts description, analysis, and suggestions from the frontend.

Generates a unique ticket ID (e.g., SF-2025-XXXXXXX, with sequence suffixes for batches when needed).

Builds a complete ticket item with all metadata.

Writes the ticket to the ServiceFlow-Users DynamoDB table.

Returns the created ticket object.

Analysis Logic

categorize_issue(description) – determines category (Hardware, Network, etc.).

determine_priority(description) – infers High/Medium/Low from urgency keywords.

identify_problem_type(description, category) – refines into specific problem types (e.g., Hardware Failure, Connection Issue).

determine_urgency(description) – sets urgency level from time/impact language.

estimate_resolution_time(category, priority) – looks up an estimated resolution window.

generate_suggestions(analysis, description) – produces team assignment and AI quick-fix recommendations.

3️⃣ Ticket Retrieval Function – getTickets
Purpose
Provide a list of tickets for dashboards and reporting.

Responsibilities

Handle CORS preflight (HTTP OPTIONS) requests for browser access.

Scan the ServiceFlow-Users DynamoDB table to retrieve ticket items.

Sort tickets by creation timestamp (newest first).

Return JSON in the format:

json
{
  "tickets": [...],
  "count": <number>,
  "timestamp": "<ISO-8601 time>"
}
🗄️ Data Storage
DynamoDB – ServiceFlow-Users Table
Each ticket item typically contains:

Identity and content

ticketId, subject, description, enhancedDescription

Classification and status

category, priority, urgency, problemType, status

Assignment

assignedTeam, assignee, userId

Timing and SLA

created, createdDate, updated, estimatedResolutionTime

AI metadata

aiGenerated, isMultiTicketBatch, batchSequence, totalInBatch, quickFixes.

🧠 AI & Knowledge Base
AWS Bedrock (Claude 3 Haiku) for:

Knowledge base question answering via RAG.

IT support responses grounded in your own documentation.

Knowledge Base Configuration

knowledgeBaseId (e.g., KQOOGFIEAH).

Vector search configuration (e.g., top 5 results) for semantic retrieval and ranking.

🔑 Authentication
AWS Cognito User Pool

Handles user registration, login, and account management.

Issues JWT tokens that are stored securely on the frontend.

Tokens are attached to outgoing requests and validated by backend services to protect APIs and data.
