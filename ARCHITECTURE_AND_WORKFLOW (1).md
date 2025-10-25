# AMX AI Voice Assistant - Architecture & Workflow Documentation

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [System Architecture Diagram](#system-architecture-diagram)
3. [Component Architecture](#component-architecture)
4. [Data Flow & Workflow](#data-flow--workflow)
5. [Methodology & Steps](#methodology--steps)
6. [Development Roadmap](#development-roadmap)
7. [Technical Stack](#technical-stack)
8. [Deployment Architecture](#deployment-architecture)

---

## Architecture Overview

The AMX AI Voice Assistant is a **real-time voice intelligence application** built with Next.js 14 (App Router) and TypeScript. It enables natural voice conversations with an AI assistant that can perform actions like checking order status, creating support tickets, booking callbacks, and escalating to human agents.

### Key Architectural Principles
- **Client-Side Voice Processing**: Vapi Web SDK handles real-time voice communication
- **Server-Side Actions**: Next.js API routes (or PHP endpoints) handle business logic
- **Modular Component Design**: Reusable UI components with clear separation of concerns
- **Real-time State Management**: React hooks for managing call state, transcripts, and actions
- **Database Persistence**: SQLite/MySQL for storing orders, tickets, callbacks, and audit logs

---

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         USER INTERFACE (Browser)                        │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────┐   │
│  │                    Next.js React Application                    │   │
│  │                                                                  │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │   │
│  │  │  Call UI     │  │  Transcript  │  │  Intent & Actions    │ │   │
│  │  │  Controls    │  │  Display     │  │  Display             │ │   │
│  │  └──────────────┘  └──────────────┘  └──────────────────────┘ │   │
│  │                                                                  │   │
│  │  ┌──────────────────────────────────────────────────────────┐  │   │
│  │  │         State Management (React Hooks)                   │  │   │
│  │  │  - callState    - transcripts    - currentIntent        │  │   │
│  │  │  - actionResult - sessionId      - escalationState      │  │   │
│  │  └──────────────────────────────────────────────────────────┘  │   │
│  │                                                                  │   │
│  └────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ WebRTC / WebSocket
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         VAPI VOICE PLATFORM                              │
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                 │
│  │   Speech     │  │   Natural    │  │   Text to    │                 │
│  │   to Text    │  │   Language   │  │   Speech     │                 │
│  │   (STT)      │  │   Processing │  │   (TTS)      │                 │
│  └──────────────┘  └──────────────┘  └──────────────┘                 │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────┐   │
│  │              Function Calling / Tool Invocation                │   │
│  │  - Detects user intent from conversation                       │   │
│  │  - Calls registered tools with extracted parameters            │   │
│  └────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ HTTPS POST Requests
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     BACKEND API (Next.js / PHP)                         │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────┐   │
│  │                      API Routes / Endpoints                     │   │
│  │                                                                  │   │
│  │  /api/tools/check-order-status      ─┐                         │   │
│  │  /api/tools/create-support-ticket   ─┤                         │   │
│  │  /api/tools/book-callback           ─┤─► Business Logic       │   │
│  │  /api/tools/handoff-to-human        ─┤                         │   │
│  │  /api/audit                          ─┘   Audit Logging        │   │
│  │                                                                  │   │
│  └────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ SQL Queries
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      DATABASE (SQLite / MySQL)                          │
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                 │
│  │   Orders     │  │   Tickets    │  │  Callbacks   │                 │
│  └──────────────┘  └──────────────┘  └──────────────┘                 │
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                 │
│  │   Agents     │  │  Sessions    │  │  Audit Logs  │                 │
│  └──────────────┘  └──────────────┘  └──────────────┘                 │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Component Architecture

### Frontend Components

```
app/
├── layout.tsx                    # Root layout with metadata
├── globals.css                   # Global styles (futuristic design)
├── page.tsx                      # Main call interface
│
├── components/
│   ├── MicMeter.tsx             # Microphone level visualization
│   ├── TranscriptList.tsx       # Real-time transcript display
│   ├── IntentCard.tsx           # Current intent display
│   └── ActionResult.tsx         # Tool call results display
│
├── admin/
│   └── page.tsx                 # Admin dashboard (stats, logs)
│
└── agent/                       # Agent takeover interface (removed in static)
    ├── page.tsx                 # Agent view for escalations
    ├── login/page.tsx           # Agent authentication
    └── dashboard/page.tsx       # Agent dashboard
```

### Backend Structure

```
app/api/                         # API Routes (removed in static export)
├── tools/
│   ├── check-order-status/
│   ├── create-support-ticket/
│   ├── book-callback/
│   └── handoff-to-human/
│
├── agent/
│   ├── login/                   # JWT authentication
│   ├── register/                # Agent registration
│   ├── status/                  # Online/offline status
│   └── escalations/             # Escalation management
│
├── audit/                       # Audit log retrieval
└── admin/
    └── stats/                   # Admin statistics

lib/
├── vapiClient.ts               # Vapi SDK initialization
├── audit.ts                    # Audit logging utilities
├── policy.ts                   # Escalation policy logic
├── auth.ts                     # JWT & bcrypt utilities
└── intentSchema.ts             # Intent detection schema

prisma/
├── schema.prisma               # Database schema
└── seed.js                     # Database seeding
```

---

## Data Flow & Workflow

### 1. Call Initialization Flow

```
User clicks "Start Call"
        ↓
Generate unique sessionId (UUID)
        ↓
Initialize Vapi Web SDK with:
  - API Key
  - Assistant ID
  - Session metadata
        ↓
Vapi establishes WebRTC connection
        ↓
Audio stream starts
        ↓
Update UI state to "active"
```

### 2. Voice Conversation Flow

```
User speaks
        ↓
Vapi STT converts speech to text
        ↓
[TRANSCRIPT EVENT]
  - type: "partial" (while speaking)
  - type: "final" (after pause)
        ↓
Update transcript list in UI
        ↓
Vapi NLP analyzes intent
        ↓
AI generates response
        ↓
Vapi TTS converts text to speech
        ↓
User hears response through browser
```

### 3. Tool Invocation Flow

```
User: "Check my order #12345"
        ↓
Vapi detects intent: check_order_status
        ↓
Extracts parameters: { order_id: "12345" }
        ↓
[FUNCTION_CALL EVENT]
  - tool: "check-order-status"
  - parameters: { order_id: "12345" }
        ↓
Frontend receives event
        ↓
POST request to /api/tools/check-order-status
  Body: { order_id: "12345" }
        ↓
Backend normalizes order ID:
  - Remove "#", spaces, dashes
  - Convert to uppercase
        ↓
Query database (Prisma):
  prisma.order.findUnique({ where: { code: "12345" }})
        ↓
Return result:
  {
    found: true,
    status: "Shipped",
    eta_friendly: "Arriving in 2 days",
    order_id: "12345"
  }
        ↓
Frontend displays in ActionResult component
        ↓
Vapi speaks result:
  "Your order 12345 is shipped and will arrive in 2 days"
```

### 4. Escalation Flow

```
AI detects frustration / user requests human
        ↓
Call handoff_to_human tool
        ↓
Backend logs escalation to audit
        ↓
Create EscalationRequest in database (if using full backend)
        ↓
Frontend shows escalation alert
        ↓
User can open Agent View
        ↓
Agent logs in to dashboard
        ↓
Agent sees pending escalation
        ↓
Agent accepts escalation
        ↓
Agent joins call / takes over conversation
```

### 5. Audit Logging Flow

```
Any significant event occurs:
  - Call started
  - Intent detected
  - Tool called
  - Escalation requested
  - Call ended
        ↓
Call logAudit() function
        ↓
Insert record into Audit table:
  {
    sessionId: "abc-123",
    event: "tool_call",
    payload: JSON.stringify(data),
    timestamp: new Date()
  }
        ↓
Admin can view audit logs in dashboard
```

---

## Methodology & Steps

### Phase 1: Project Setup
1. **Initialize Next.js project** with TypeScript, Tailwind CSS
2. **Configure Prisma** with SQLite/MySQL database
3. **Create database schema** for Orders, Tickets, Callbacks, Agents, Audit
4. **Seed database** with sample data for testing
5. **Set up environment variables** for Vapi API keys

### Phase 2: Vapi Integration
1. **Create Vapi assistant** in Vapi dashboard
2. **Configure system prompt** with instructions and tool definitions
3. **Initialize Vapi Web SDK** in `lib/vapiClient.ts`
4. **Register event handlers** for:
   - `call-start`: Initialize session
   - `speech-start` / `speech-end`: Update UI
   - `message`: Display transcripts (partial/final)
   - `function-call`: Handle tool invocations
   - `call-end`: Clean up session

### Phase 3: Backend API Development
1. **Create API routes** for each Vapi tool:
   - `check-order-status`: Query Orders table
   - `create-support-ticket`: Insert into Tickets table
   - `book-callback`: Insert into Callbacks table
   - `handoff-to-human`: Create escalation record
2. **Implement audit logging** utility
3. **Add admin statistics** endpoint for dashboard
4. **Implement agent authentication** (JWT + bcrypt)
5. **Create escalation management** endpoints

### Phase 4: Frontend UI Development
1. **Design main call interface** (`app/page.tsx`):
   - Call controls (Start/End/Escalate buttons)
   - Status indicators
   - Microphone visualization
2. **Build transcript component** with RTL support
3. **Create intent card** to show AI's understanding
4. **Build action result display** for tool outputs
5. **Implement admin dashboard** with stats and logs
6. **Create agent login and dashboard** interfaces

### Phase 5: Styling & UX
1. **Apply futuristic design** with glassmorphism effects
2. **Add gradient animations** and pulse effects
3. **Implement responsive layout** for mobile/desktop
4. **Add loading states** and transitions
5. **Optimize for sub-500ms perceived latency**

### Phase 6: Testing & Optimization
1. **Test order lookup** with various formats (#12345, AQ12345)
2. **Test support ticket creation** with different subjects
3. **Test callback booking** with date/time parsing
4. **Test escalation flow** end-to-end
5. **Test multilingual support** (English/Arabic)
6. **Optimize function calling** with clear prompts
7. **Test barge-in** (interrupting AI)

### Phase 7: Deployment
1. **Static Export** (for shared hosting):
   - Remove API routes
   - Export as static HTML/CSS/JS
   - Deploy to shared hosting
2. **Full-Stack Deployment** (for VPS/cloud):
   - Deploy Next.js with Node.js
   - Configure MySQL database
   - Set up environment variables
   - Enable HTTPS

---

## Development Roadmap

### MVP (Minimum Viable Product) - Week 1
- ✅ Basic call interface with Start/End buttons
- ✅ Real-time transcript display
- ✅ Order status checking
- ✅ Support ticket creation
- ✅ Callback booking
- ✅ Basic UI with Tailwind CSS

### Enhanced Features - Week 2
- ✅ Futuristic UI design with animations
- ✅ Intent detection display
- ✅ Action result visualization
- ✅ Admin dashboard with statistics
- ✅ Audit logging system
- ✅ Microphone level meter

### Advanced Features - Week 3
- ✅ Agent authentication system
- ✅ Escalation management
- ✅ Agent dashboard for handling escalations
- ✅ Session management with JWT
- ✅ MySQL integration for production

### Optimization & Polish - Week 4
- ✅ Improved function calling prompts
- ✅ Flexible order ID parsing
- ✅ Better error handling
- ✅ Responsive design for all devices
- ✅ Multi-language support (EN/AR)
- ✅ Barge-in optimization

### Deployment Ready - Week 5
- ✅ Static export for shared hosting
- ✅ PHP version for compatibility
- ✅ Database migration scripts
- ✅ Setup wizard for easy installation
- ✅ Comprehensive documentation
- ✅ Deployment guides

### Future Enhancements (Roadmap)
- 🔄 Video call support
- 🔄 Screen sharing for agents
- 🔄 AI sentiment analysis with real-time mood detection
- 🔄 Multi-agent support (team handoffs)
- 🔄 Analytics dashboard with charts
- 🔄 Call recording and playback
- 🔄 Customer profile integration (CRM)
- 🔄 Automated follow-ups
- 🔄 Voice biometrics for authentication
- 🔄 Integration with third-party APIs (Shopify, Salesforce, etc.)

---

## Technical Stack

### Frontend
- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS + Custom CSS
- **State Management**: React Hooks (useState, useEffect, useRef)
- **Voice SDK**: Vapi Web SDK 2.3.0
- **Icons**: SVG inline components

### Backend (Full-Stack Version)
- **Runtime**: Node.js 18+
- **API Framework**: Next.js API Routes
- **ORM**: Prisma
- **Database**: SQLite (dev) / MySQL (production)
- **Authentication**: JWT (jose) + bcrypt

### Backend (PHP Version)
- **Language**: PHP 7.4+
- **Database**: MySQL with PDO
- **Authentication**: JWT (Firebase PHP-JWT)
- **Password Hashing**: password_hash() / password_verify()

### Development Tools
- **Package Manager**: npm
- **Version Control**: Git
- **Linting**: ESLint
- **Formatting**: Prettier (optional)

---

## Deployment Architecture

### Option 1: Static Export (Shared Hosting)

```
┌─────────────────────────────────────────┐
│        Shared Hosting Server            │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │   Static HTML/CSS/JS Files         │ │
│  │   (from Next.js export or          │ │
│  │    static-export/ folder)          │ │
│  │                                     │ │
│  │   - index.html                     │ │
│  │   - styles.css                     │ │
│  │   - app.js (with Vapi SDK)        │ │
│  └────────────────────────────────────┘ │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │   PHP API Endpoints (php-demo/)    │ │
│  │                                     │ │
│  │   - config.php                     │ │
│  │   - api/check-order-status.php    │ │
│  │   - api/create-support-ticket.php │ │
│  │   - api/book-callback.php         │ │
│  │   - api/handoff-to-human.php      │ │
│  └────────────────────────────────────┘ │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │   MySQL Database                   │ │
│  │   (via cPanel / phpMyAdmin)        │ │
│  └────────────────────────────────────┘ │
│                                          │
└─────────────────────────────────────────┘
```

**Advantages**:
- ✅ Works on any shared hosting (no Node.js required)
- ✅ Simple deployment (FTP upload)
- ✅ Low cost
- ✅ Easy to maintain

**Limitations**:
- ❌ No server-side React rendering
- ❌ No advanced API features (middleware, etc.)
- ❌ Limited scalability

### Option 2: Full-Stack Deployment (VPS/Cloud)

```
┌─────────────────────────────────────────┐
│         Cloud Server (VPS)              │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │   Next.js Application              │ │
│  │   (Node.js server on port 3000)    │ │
│  │                                     │ │
│  │   - SSR + Client-side routing      │ │
│  │   - API routes                     │ │
│  │   - Prisma ORM                     │ │
│  └────────────────────────────────────┘ │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │   Nginx Reverse Proxy              │ │
│  │   (port 80/443 → 3000)             │ │
│  └────────────────────────────────────┘ │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │   MySQL Database                   │ │
│  │   (separate service or container)  │ │
│  └────────────────────────────────────┘ │
│                                          │
└─────────────────────────────────────────┘
```

**Advantages**:
- ✅ Full Next.js features (SSR, ISR, etc.)
- ✅ Better performance
- ✅ Scalable architecture
- ✅ Advanced API capabilities

**Requirements**:
- ⚙️ Node.js 18+ installed
- ⚙️ Process manager (PM2)
- ⚙️ Nginx or Apache
- ⚙️ MySQL server

---

## Key Design Decisions

### 1. **Client-Side Voice Processing**
- Voice interaction happens entirely in the browser via Vapi SDK
- No audio streaming to our backend
- Reduces latency and server load

### 2. **Event-Driven Architecture**
- Vapi emits events (speech-start, message, function-call, etc.)
- Frontend listens and updates UI reactively
- Clean separation between voice and business logic

### 3. **Stateless API Routes**
- Each tool endpoint is independent
- No session management in API (sessionId passed in request)
- Easy to scale horizontally

### 4. **Database-First Approach**
- Prisma schema defines data models
- Type-safe queries with TypeScript
- Easy migration between SQLite and MySQL

### 5. **Progressive Enhancement**
- Core functionality works with minimal JavaScript
- Enhanced UX with React and Vapi
- Graceful fallbacks for unsupported features

### 6. **Security Considerations**
- API keys stored in environment variables (never in code)
- Agent passwords hashed with bcrypt
- JWT tokens for agent sessions
- SQL injection prevention with Prisma

---

## Performance Optimization

### Sub-500ms First Audio Feel
1. **Preload Vapi SDK**: Load SDK before user interaction
2. **Warm Start**: Initialize Vapi instance on page load
3. **Optimize Prompts**: Keep system prompts concise
4. **Stream Responses**: Use streaming TTS for faster perceived response
5. **Reduce Round Trips**: Batch tool calls when possible

### UI Performance
1. **Lazy Loading**: Load components only when needed
2. **Memoization**: Use React.memo for expensive components
3. **Virtual Scrolling**: For long transcript lists
4. **CSS Animations**: Hardware-accelerated transforms
5. **Code Splitting**: Separate bundles for admin/agent views

---

## Conclusion

This architecture provides a **scalable, modular, and performant** foundation for a real-time voice AI assistant. The system is designed to be:

- **Flexible**: Works as static export or full-stack app
- **Maintainable**: Clear separation of concerns
- **Extensible**: Easy to add new tools and features
- **Production-Ready**: Comprehensive error handling and logging
- **User-Friendly**: Intuitive UI with real-time feedback

The combination of Vapi's voice platform with Next.js's modern React architecture creates a powerful, low-latency voice experience suitable for customer service, support automation, and conversational AI applications.

---

**Document Version**: 1.0  
**Last Updated**: October 2025  
**Maintained By**: AMX AI Development Team

