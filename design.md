# System Design Document

**Project:** AI DevCompanion  
**Version:** 1.0  
**Architecture Type:** Full-Stack Web Application  
**Last Updated:** February 15, 2026

---

## 1. Architecture Overview

AI DevCompanion follows a modern client-server architecture with clear separation of concerns. The system is designed for real-time AI-powered code analysis with a focus on developer experience and visual appeal.

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         User Browser                         │
│  ┌───────────────────────────────────────────────────────┐  │
│  │           React 19 Frontend (Port 5173)               │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐  │  │
│  │  │  Dashboard  │  │  Components  │  │   Services  │  │  │
│  │  │    Page     │  │   (UI/UX)    │  │  (API Calls)│  │  │
│  │  └─────────────┘  └──────────────┘  └─────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP/JSON
                           │ (CORS Enabled)
┌──────────────────────────▼──────────────────────────────────┐
│              FastAPI Backend (Port 8000)                     │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  API Endpoints  │  Models  │  Services  │  Utils     │  │
│  │  (Routes)       │ (Pydantic)│ (LLM)     │ (Prompts)  │  │
│  └───────────────────────────────────────────────────────┘  │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTPS/REST API
                           │ (API Key Auth)
┌──────────────────────────▼──────────────────────────────────┐
│              Google Gemini AI Service                        │
│                  (gemini-2.5-flash)                          │
└──────────────────────────────────────────────────────────────┘
```

### 1.2 Data Flow

```
User Input → Frontend Validation → API Request → Backend Validation
    ↓
Prompt Engineering → LLM Processing → JSON Response → Frontend Rendering
    ↓
Structured Display → User Interaction → Copy/Export
```

---

## 2. Frontend Design

### 2.1 Technology Stack

- **Framework:** React 19.2.0 with functional components and hooks
- **Build Tool:** Vite 7.3.1 for fast development and optimized builds
- **Styling:** TailwindCSS 4.1.18 with custom utility classes
- **Animations:** Framer Motion 12.34.0 for smooth transitions
- **Icons:** Lucide React 0.564.0 for consistent iconography
- **Markdown:** React Markdown 10.1.0 for formatted output

### 2.2 Component Architecture

```
App.jsx (Root)
  └── Dashboard.jsx (Main Page)
      ├── Navbar.jsx (Header)
      ├── InputArea.jsx (Left Panel)
      │   ├── Tab Selector (Error/Code/Log)
      │   ├── Textarea (Input)
      │   └── Analyze Button
      └── OutputCard.jsx (Right Panel)
          ├── Analysis Header
          ├── Summary Section
          ├── Details Grid
          └── Actionable Items List
```

### 2.3 Component Details

#### Navbar Component
- **Purpose:** Branding and navigation
- **Features:**
  - Animated logo with rotation on hover
  - Gradient text effect for branding
  - GitHub link with icon
  - Sticky positioning with glassmorphism
- **Animations:** Fade-in from top on mount

#### InputArea Component
- **Purpose:** User input and mode selection
- **Features:**
  - Three-tab mode selector (Error/Code/Log)
  - Color-coded tabs with icons
  - Monospace textarea for code input
  - Context-aware placeholders
  - Animated submit button with loading state
  - Custom scrollbar styling
- **State Management:**
  - `input`: Current textarea content
  - `mode`: Selected analysis mode
- **Interactions:**
  - Tab switching with visual feedback
  - Real-time input validation
  - Disabled state when loading or empty

#### OutputCard Component
- **Purpose:** Display structured AI analysis results
- **Features:**
  - Conditional rendering based on result availability
  - Confidence badge (High/Medium/Low)
  - Markdown rendering for formatted text
  - Staggered animations for sections
  - Icon-based section headers
  - Numbered list for actionable items
  - Hover effects on interactive elements
- **Sections:**
  - Summary/Explanation (primary insight)
  - Root Cause/Complexity (technical details)
  - Key Concepts (learning tags)
  - Fixes/Tips/Issues (actionable list)

#### Dashboard Component
- **Purpose:** Main application container and state orchestrator
- **State Management:**
  - `result`: AI analysis response
  - `isLoading`: Request status
  - `error`: Error message display
- **Features:**
  - Two-panel grid layout (responsive)
  - Animated background gradients
  - Error notification banner
  - API call orchestration
  - Mode-based routing to correct endpoint

### 2.4 Visual Design System

#### Color Palette
- **Background:** Dark gray (#0a0a0a to #1a1a1a)
- **Accent Primary:** Blue (#3b82f6 to #2563eb)
- **Accent Secondary:** Purple (#8b5cf6 to #7c3aed)
- **Success:** Green (#10b981)
- **Warning:** Amber (#f59e0b)
- **Error:** Red (#ef4444)
- **Text Primary:** White/Light gray (#f3f4f6)
- **Text Secondary:** Medium gray (#9ca3af)

#### Typography
- **Headings:** Bold, gradient text effects
- **Body:** Light weight, relaxed line height
- **Code:** Monospace font (font-mono)
- **Labels:** Uppercase, tracked spacing

#### Effects & Animations
- **Glassmorphism:** `backdrop-blur` with semi-transparent backgrounds
- **Gradients:** Animated background orbs with pulse effect
- **Transitions:** 300ms duration for smooth interactions
- **Hover States:** Scale transforms and color shifts
- **Loading:** Spinning animation with opacity variations
- **Stagger:** Sequential fade-in for list items

#### Responsive Design
- **Breakpoints:**
  - Mobile: < 640px (single column)
  - Tablet: 640px - 1024px (adaptive layout)
  - Desktop: > 1024px (two-column grid)
- **Adaptive Elements:**
  - Tab labels hidden on mobile
  - Grid switches to single column
  - Padding adjustments for smaller screens

### 2.5 Service Layer

#### API Service (`api.js`)
- **Base URL:** `http://localhost:8000`
- **Methods:**
  - `analyzeError(errorMessage, context)`: POST to `/analyze-error`
  - `explainCode(codeSnippet)`: POST to `/explain-code`
  - `summarizeLogs(logContent)`: POST to `/summarize-logs`
- **Error Handling:** Network error detection with throw
- **Headers:** JSON content type for all requests

---

## 3. Backend Design

### 3.1 Technology Stack

- **Framework:** FastAPI (async-capable web framework)
- **Server:** Uvicorn ASGI server
- **Validation:** Pydantic models for type safety
- **AI SDK:** google-genai (official Google SDK)
- **Environment:** python-dotenv for configuration

### 3.2 Application Structure

```
backend/
├── main.py              # FastAPI app, routes, CORS
├── models.py            # Pydantic request/response models
├── services/
│   └── llm.py          # GeminiService class
└── utils/
    └── prompts.py      # Prompt templates
```

### 3.3 Core Components

#### Main Application (`main.py`)
- **Initialization:**
  - Load environment variables
  - Configure CORS for frontend origin
  - Initialize GeminiService with error handling
- **Endpoints:**
  - `GET /`: Health check
  - `POST /analyze-error`: Error analysis
  - `POST /explain-code`: Code explanation
  - `POST /summarize-logs`: Log summarization
- **Error Handling:**
  - Service initialization validation
  - HTTP 500 for uninitialized service
  - Graceful degradation

#### Data Models (`models.py`)
- **Request Models:**
  - `ErrorRequest`: error_message (str), context (Optional[str])
  - `CodeRequest`: code_snippet (str), language (Optional[str])
  - `LogRequest`: log_content (str)
- **Response Models:**
  - `AnalysisResponse`: summary, cause, fixes (List), concepts (List), confidence
  - `ExplanationResponse`: explanation, complexity, tips (List)
  - `SummaryResponse`: summary, key_issues (List)
- **Validation:** Automatic via Pydantic

#### LLM Service (`services/llm.py`)
- **Class:** `GeminiService`
- **Initialization:**
  - Validate API key from environment
  - Create Google GenAI client
  - Set model to `gemini-2.5-flash`
- **Methods:**
  - `analyze_error(error_message, context)`: Error analysis
  - `explain_code(code_snippet)`: Code explanation
  - `summarize_logs(log_content)`: Log summarization
  - `_clean_json(text)`: Remove markdown formatting from responses
- **Error Handling:**
  - Try-catch for all LLM calls
  - Fallback responses with error details
  - JSON parsing with cleanup

#### Prompt Engineering (`utils/prompts.py`)
- **Templates:**
  - `ERROR_ANALYSIS_PROMPT`: Structured error analysis instructions
  - `CODE_EXPLAIN_PROMPT`: Code explanation with complexity analysis
  - `LOG_SUMMARY_PROMPT`: Log summarization with issue extraction
- **Format:** String templates with `.format()` placeholders
- **Instructions:**
  - Role definition (senior developer, teacher, analyst)
  - Output format specification (JSON structure)
  - Required fields and data types
  - Tone and style guidelines

### 3.4 API Design

#### Request/Response Flow
1. **Request Reception:** FastAPI receives POST request
2. **Validation:** Pydantic validates request body
3. **Service Check:** Verify LLM service is initialized
4. **Prompt Construction:** Format template with user input
5. **LLM Call:** Send to Google Gemini API
6. **Response Parsing:** Clean and parse JSON response
7. **Error Handling:** Catch exceptions, return fallback
8. **Response Return:** Send structured JSON to frontend

#### CORS Configuration
- **Allowed Origins:** `http://localhost:5173`
- **Credentials:** Enabled
- **Methods:** All (`*`)
- **Headers:** All (`*`)

---

## 4. AI Integration Design

### 4.1 Google Gemini Integration

- **Model:** gemini-2.5-flash (fast, cost-effective)
- **SDK:** google-genai (official Python client)
- **Authentication:** API key from environment variable
- **Endpoint:** `generativelanguage.googleapis.com`

### 4.2 Prompt Engineering Strategy

#### Role-Based Prompting
- **Error Analysis:** "Senior developer mentor"
- **Code Explanation:** "Coding teacher"
- **Log Summarization:** "Efficient log analyzer"

#### Structured Output
- All prompts request JSON format
- Explicit field definitions
- Type specifications for arrays
- Confidence/complexity metrics

#### Context Injection
- User input embedded in templates
- Optional context fields
- Language hints for code snippets

### 4.3 Response Processing

#### JSON Cleanup
- Remove markdown code fences (```json, ```)
- Strip whitespace
- Handle malformed responses

#### Fallback Mechanism
- Catch all exceptions
- Return structured error responses
- Maintain consistent response format
- Log errors for debugging

---

## 5. User Experience Design

### 5.1 Interaction Flow

```
1. User lands on dashboard
   ↓
2. Sees empty state with instructions
   ↓
3. Selects mode (Error/Code/Log)
   ↓
4. Pastes input text
   ↓
5. Clicks "Analyze" button
   ↓
6. Loading state with spinner
   ↓
7. Results appear with animations
   ↓
8. User reads structured analysis
   ↓
9. Can copy/export results (future)
```

### 5.2 Visual Feedback

- **Loading:** Spinning icon, disabled button, "Processing..." text
- **Success:** Smooth fade-in of results with stagger effect
- **Error:** Red banner at top with error icon and message
- **Hover:** Scale transforms, color changes, shadow effects
- **Focus:** Outline removal, custom focus states

### 5.3 Empty States

- **No Results:** Animated icon, welcome message, usage instructions
- **Error State:** Fallback results with connection troubleshooting

### 5.4 Accessibility Considerations

- **Color Contrast:** High contrast ratios for text
- **Interactive Elements:** Clear hover and focus states
- **Icons:** Paired with text labels
- **Loading States:** Clear visual and text indicators
- **Error Messages:** Descriptive and actionable

---

## 6. Performance Design

### 6.1 Frontend Optimization

- **Code Splitting:** Vite automatic chunking
- **Lazy Loading:** Component-level imports
- **Asset Optimization:** SVG icons, minimal images
- **CSS:** Utility-first approach, minimal custom CSS
- **Animations:** GPU-accelerated transforms

### 6.2 Backend Optimization

- **Async Operations:** FastAPI async endpoints
- **Connection Pooling:** Reusable LLM client
- **Error Caching:** Fallback responses
- **Validation:** Early request validation

### 6.3 Network Optimization

- **Request Size:** Minimal JSON payloads
- **Response Compression:** FastAPI automatic gzip
- **CORS Preflight:** Cached by browser
- **Keep-Alive:** HTTP connection reuse

---

## 7. Security Design

### 7.1 API Key Management

- **Storage:** Environment variables only
- **Access:** Backend-only, never exposed to frontend
- **Validation:** Checked on service initialization
- **Rotation:** Easy to update via .env file

### 7.2 Input Validation

- **Frontend:** Basic empty check
- **Backend:** Pydantic model validation
- **Sanitization:** No code execution, read-only analysis

### 7.3 CORS Policy

- **Strict Origin:** Only localhost:5173 allowed
- **Production:** Update to actual domain
- **Credentials:** Enabled for future auth

### 7.4 Data Privacy

- **No Storage:** Stateless processing
- **No Logging:** User code not persisted
- **Session-Based:** Data cleared after response

---

## 8. Error Handling Design

### 8.1 Frontend Error Handling

- **Network Errors:** Catch fetch failures, show banner
- **Empty Input:** Disable button, prevent submission
- **Loading Timeout:** User can see processing state
- **Fallback UI:** Show partial results even on error

### 8.2 Backend Error Handling

- **Service Initialization:** Graceful failure with HTTP 500
- **LLM Errors:** Catch exceptions, return structured fallback
- **JSON Parsing:** Clean markdown, retry parse
- **Validation Errors:** Automatic Pydantic error responses

### 8.3 User-Facing Messages

- **Connection Error:** "Failed to connect to server. Please ensure backend is running."
- **Processing Error:** "Error analyzing response" with technical details
- **Validation Error:** Automatic from Pydantic (422 status)

---

## 9. Scalability Considerations

### 9.1 Current Limitations

- **Single Server:** No load balancing
- **No Caching:** Every request hits LLM
- **No Rate Limiting:** Unlimited requests
- **No Authentication:** Open access

### 9.2 Future Enhancements

- **Caching Layer:** Redis for common queries
- **Rate Limiting:** Per-IP or per-user limits
- **Load Balancing:** Multiple backend instances
- **Database:** Store user history and preferences
- **CDN:** Static asset delivery
- **Monitoring:** Application performance tracking

---

## 10. Development Workflow

### 10.1 Local Development

```bash
# Terminal 1: Backend
cd backend
pip install -r requirements.txt
uvicorn main:app --reload

# Terminal 2: Frontend
cd frontend
npm install
npm run dev
```

### 10.2 Environment Setup

- **Backend:** Create `.env` with `GEMINI_API_KEY`
- **Frontend:** No environment variables needed
- **Ports:** Backend (8000), Frontend (5173)

### 10.3 Testing Strategy

- **Manual Testing:** Use UI for end-to-end tests
- **API Testing:** Use curl or Postman for endpoints
- **Error Testing:** Test with invalid inputs
- **Performance Testing:** Monitor response times

---

## 11. Design Principles

### 11.1 Core Principles

1. **Simplicity:** Minimal UI, clear purpose
2. **Speed:** Fast responses, smooth animations
3. **Clarity:** Structured output, easy to scan
4. **Beauty:** Modern design, attention to detail
5. **Reliability:** Graceful error handling

### 11.2 Technical Principles

1. **Separation of Concerns:** Clear component boundaries
2. **Type Safety:** Pydantic models, PropTypes
3. **Async First:** Non-blocking operations
4. **Stateless:** No server-side sessions
5. **API-Driven:** Clean REST interface

---

**Note:** This design document reflects the current implementation of AI DevCompanion. The architecture is designed for simplicity and developer experience, with clear paths for future enhancements.
