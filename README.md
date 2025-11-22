# White Rabbit Profile Optimizer - Multi-Agent System

## Overview

The White Rabbit Profile Optimizer is a proof-of-concept three-agent AI system designed to help community members build rich, complete profiles through intelligent assessment, conversational guidance, and social profile analysis. The system operates with strict ethical boundaries: all agents are read-only with respect to member profile data, and members maintain full control over what information becomes public.

## System Architecture

### Three Specialized Agents

The system uses a multi-agent architecture where each agent has a specific responsibility and clearly defined data access boundaries:

```
┌─────────────────────────────────────────────────────────────┐
│                    Member Profile Database                   │
│                      (PostgreSQL/Neon)                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ read-only
                              ▼
                  ┌───────────────────────┐
                  │ Profile Evaluation    │
                  │      Agent            │
                  │ • Reads profiles      │
                  │ • Calculates scores   │
                  │ • Identifies gaps     │
                  └───────────────────────┘
                              │
                              │ assessment results
                              ▼
                  ┌───────────────────────┐
                  │   Interactive Agent   │
                  │ • NO DB access        │
                  │ • Conducts interviews │
                  │ • Generates suggestions│
                  └───────────────────────┘
                              │
                              │ copy-to-clipboard
                              ▼
                         ┌─────────┐
                         │  Member │
                         │ manually│
                         │ updates │
                         └─────────┘
```

### 1. Profile Evaluation Agent

**Purpose:** Continuously assess profile completeness and quality

**Data Access:**
- ✅ Read-only access to member profiles
- ✅ Write access to `profile_completeness` table only
- ❌ No access to conversation history

**Responsibilities:**
- Calculate completeness scores based on configurable field weights
- Identify missing or incomplete profile fields
- Evaluate field quality (length, detail, specificity)
- Assign priority levels to gaps based on strategic importance
- Trigger automatic reassessment when profiles are updated
- Return structured assessment data (no raw database records)

**Key Features:**
- **Continuous Monitoring:** Automatically reassesses profiles when members save changes
- **Configurable Criteria:** Admins can adjust field weights and quality thresholds
- **Structured Output:** Returns assessment as JSON for consumption by Interactive Agent

**Example Assessment Output:**
```json
{
  "completeness_score": 65.5,
  "missing_fields": ["skills", "interests", "current_focus"],
  "quality_issues": {
    "what_you_do": "too_short (20 chars, min 50)"
  },
  "priority_recommendations": [
    {
      "field": "skills",
      "priority": "high",
      "reason": "Critical for collaboration matching"
    }
  ]
}
```

### 2. URL Processing Agent

**Purpose:** Analyze social profile URLs and generate markdown artifacts

**Data Access:**
- ✅ Limited read access (member_id and name only for artifact naming)
- ✅ Write access to filesystem for markdown artifacts
- ❌ No access to full member profiles
- ❌ No access to conversation history

**Responsibilities:**
- Scrape social profile URLs (LinkedIn, Twitter, GitHub, etc.)
- Extract ONLY schema-relevant data (skills, interests, location, goals)
- Generate structured markdown artifacts: `user_socialprofile_platformname.md`
- Store artifacts on filesystem (independent of database)
- Respect privacy: NO extraction of contact info, private messages, or non-schema data

**Key Features:**
- **Asynchronous Processing:** Runs in background without blocking user interactions
- **Privacy-First:** Only extracts data that maps to database schema fields
- **Configurable Extraction:** Admins can control what data categories are extracted
- **Filesystem Storage:** Artifacts stored independently for easy access by Interactive Agent

**Example Artifact Structure:**
```markdown
# Social Profile: LinkedIn

**Member:** Benjamin Durham
**Platform:** linkedin
**Profile URL:** https://linkedin.com/in/example
**Extracted:** 2025-11-22T10:30:00Z

## Skills
- Research
- Systems Design
- Data Analysis

## Interests
- AI Ethics
- Systems Thinking

## Professional Focus
Currently working on profile optimization systems...

## Location
Ashland, Oregon

## Goals & Motivations
Building tools to help creators succeed...
```

### 3. Interactive Agent

**Purpose:** Conduct conversational interviews and generate profile suggestions

**Data Access:**
- ❌ NO direct access to member profile database
- ✅ Receives assessment results as structured data from backend
- ✅ Read access to social profile artifacts (filesystem)
- ✅ Write access to `conversation_history` table only

**Responsibilities:**
- Present assessment results in natural language
- Conduct conversational interviews based on identified gaps
- Generate field-specific suggestions using assessment + artifacts + conversation
- Cite sources when using social profile data
- Filter sensitive information from suggestions
- Provide copy-to-clipboard formatted suggestions

**Key Features:**
- **No Database Access:** Enhanced security - cannot read member profiles directly
- **Context Synthesis:** Combines assessment results, artifacts, and conversation
- **Source Citation:** Transparent about where information comes from
- **Copy-to-Clipboard:** Members manually control what gets added to profiles

**Example Interaction Flow:**
1. Member clicks "Optimize Profile" button
2. Backend triggers Profile Evaluation Agent → gets assessment
3. Backend passes assessment to Interactive Agent (not raw DB data)
4. Interactive Agent: "Your profile is 65% complete. Let's focus on adding skills - this helps other members find you for collaboration."
5. Member responds with skills
6. Interactive Agent generates suggestion: "Based on our conversation, here are skills to add: [Copy button]"
7. Member copies and manually pastes into profile

## Data Flow

### Profile Optimization Workflow

```
1. Member clicks "Optimize Profile"
   ↓
2. Backend → Profile Evaluation Agent
   ↓
3. Profile Evaluation Agent reads profile from DB
   ↓
4. Profile Evaluation Agent calculates assessment
   ↓
5. Profile Evaluation Agent returns structured results
   ↓
6. Backend → Interactive Agent (with assessment results)
   ↓
7. Interactive Agent loads social profile artifacts (if exist)
   ↓
8. Interactive Agent presents assessment to member
   ↓
9. Member responds to questions
   ↓
10. Interactive Agent generates suggestions
    ↓
11. Member copies suggestion to clipboard
    ↓
12. Member manually pastes into profile field
    ↓
13. Member saves profile
    ↓
14. Backend triggers Profile Evaluation Agent reassessment
    ↓
15. Cycle repeats with updated assessment
```

### URL Processing Workflow

```
1. Member uploads social profile URL
   ↓
2. Backend stores URL in social_links table (processed=false)
   ↓
3. Backend triggers URL Processing Agent (async)
   ↓
4. URL Processing Agent scrapes URL
   ↓
5. URL Processing Agent extracts schema-relevant data
   ↓
6. URL Processing Agent generates markdown artifact
   ↓
7. URL Processing Agent saves to filesystem
   ↓
8. Backend updates social_links (processed=true)
   ↓
9. Interactive Agent can now load artifact for context
```

## Database Schema

The system uses a PostgreSQL database that mirrors the production Neon DB schema with some POC-specific additions.

### Core Tables

**members** - Main profile data
- Basic info: first_name, last_name, profile_photo_url
- Profile content: what_you_do, where_location, website
- Timestamps: created_at, updated_at

**social_links** - Social profile URLs
- member_id, platform_name, url
- processed flag (for URL Processing Agent tracking)

**skills, interests, inspirations** - Normalized trait data
- Separate tables with many-to-many relationships via junction tables
- Allows reuse and analytics across members

**prompt_responses** - Long-form text answers
- Stores responses to prompts like "What is your current focus?"
- prompt_key, response_text, character_count

**conversation_history** - Agent chat logs
- session_id, role (user/assistant), message_content
- Private - not visible to other members

**profile_completeness** - Cached assessment results
- completeness_score, missing_fields (JSONB), last_calculated
- Updated by Profile Evaluation Agent

### Social Profile Artifacts (Filesystem)

Stored at: `/artifacts/social_profiles/{member_id}/`

Files: `{member_id}_socialprofile_{platform}.md`

**Independent of database** - allows easy access without DB queries

## Mock Membership Form (Testing Tool)

A critical component for POC development and testing.

### Purpose

Quickly create synthetic test profiles with varying completeness levels to test agent behavior.

### Features

**Individual "Populate Field" Buttons:**
- Each form field has a ⚡ button to generate realistic fake data
- Allows selective population to test specific scenarios

**Populate Random Profile:**
- Fills 30-90% of fields randomly
- Simulates realistic incomplete profiles

**Clear All:**
- Resets form to empty state

**Save Profile:**
- Writes directly to mock database
- Triggers Profile Evaluation Agent reassessment

**View Assessment:**
- Activates Interactive Agent to see how it responds to current profile state

### Form Sections

1. **Basic Info:** Name, photo, what you do, location
2. **Links & Contact:** Phone, website, social links
3. **Traits:** Skills, interests, inspirations (multi-select tags)
4. **Prompt Responses:** Current focus, needs for vision, motivations (long-form text)

### Testing Scenarios

- **Empty Profile:** Test agent behavior with no data
- **Minimal Profile:** Only required fields filled
- **Partial Profile:** Some sections complete, others empty
- **Nearly Complete:** One or two fields missing
- **Complete Profile:** All fields populated

## Technology Stack

### Backend
- **Language:** Python 3.11+
- **Framework:** FastAPI
- **Database:** PostgreSQL (local for POC, Neon for production)
- **ORM:** SQLAlchemy
- **Migrations:** Alembic
- **LLM:** Anthropic Claude SDK (direct, no LangChain)
- **Web Scraping:** BeautifulSoup

### Frontend
- **Language:** TypeScript
- **Framework:** React
- **Data Fetching:** TanStack Query (React Query)
- **Build Tool:** Vite
- **Deployment:** Custom web app, eventual subdomain of whiterabbitashland.com

### Development Approach
- **Simple over complex:** Raw Anthropic SDK, no complex frameworks
- **Local development:** Docker Compose for PostgreSQL
- **API-first:** Clean REST API boundaries
- **Iterative:** Start simple, add complexity as needed

## Admin Configuration UI

Admins can dynamically configure agent behavior without code changes.

### Configuration Panels

**1. Assessment Criteria Editor**
- Set field weights (e.g., skills: 20%, interests: 15%)
- Define quality thresholds (min characters for text fields)
- Mark fields as required/recommended/optional
- Preview impact on existing member scores

**2. System Prompt Editors (3 separate panels)**
- Profile Evaluation Agent prompt
- Interactive Agent prompt
- URL Processing Agent prompt
- Version history with rollback
- Test mode with sample inputs

**3. Extraction Criteria Editor**
- Enable/disable data categories for URL Processing Agent
- Custom instructions per category
- Markdown template editor
- Test mode with sample URLs

## Ethical Principles (Non-Negotiable)

1. **Opt-in at member's pace** - No forced data collection
2. **AI suggests, human approves** - Agents never write to profiles directly
3. **Transparent visibility** - Clear about what's public vs private
4. **Respect boundaries** - Honor communication preferences
5. **Privacy distinction** - Conversation context ≠ public profile
6. **Read-only agents** - Strict data access boundaries
7. **Source citation** - Transparent about where information comes from

## Security & Privacy

### Agent Permissions

| Agent | Profile DB | Completeness Table | Conversation | Artifacts |
|-------|-----------|-------------------|--------------|-----------|
| Profile Evaluation | Read-only | Write | No access | No access |
| URL Processing | Limited read (id, name) | No access | No access | Write |
| Interactive | **NO ACCESS** | No access | Write | Read |

### Data Privacy

- Conversation history is private (not visible to other members)
- Social profile artifacts have restricted filesystem permissions
- No PII in logs or error messages
- Audit logging for all agent operations

### Authentication

- JWT-based authentication
- Role-based access control (member, admin)
- Admin-only access to configuration endpoints

## Getting Started

### Current Status: Specification Phase

This project is currently in the **specification phase**. All requirements, design, and implementation tasks have been documented, but no code has been written yet.

### For Developers

To begin implementation:

1. **Review the specifications:**
   - Read `specs/requirements.md` for complete requirements (19 requirements)
   - Read `specs/design.md` for technical architecture
   - Read `specs/database-schema.md` for database structure
   - Read `specs/ARCHITECTURE_UPDATE.md` for 3-agent architecture explanation

2. **Start with Task 1:**
   - Open `specs/tasks.md`
   - Begin with Task 1.1: Initialize backend project
   - Follow tasks sequentially for best results

3. **Prerequisites (for when implementation begins):**
   - Python 3.11+
   - Node.js 18+
   - PostgreSQL 14+
   - Anthropic API key

### For Future Coding Agents

This README and the spec documents in `specs/` are designed to provide complete context for AI coding agents. All architectural decisions, data flows, and implementation details are documented.

## Project Structure

```
WR-profile-optimizer/
├── specs/                          # Complete specification documents
│   ├── requirements.md             # 19 requirements with EARS format
│   ├── design.md                   # Technical architecture
│   ├── database-schema.md          # Complete DB schema
│   ├── tasks.md                    # Implementation plan
│   ├── ARCHITECTURE_UPDATE.md      # 3-agent architecture explanation
│   ├── WR_MemberDBSchema.txt       # Production schema reference
│   └── WR_ProfileOptimizerMASFlowChart.png  # System diagram
├── backend/                        # FastAPI backend (to be implemented)
│   ├── agents/                     # AI agent implementations
│   │   ├── profile_evaluation.py
│   │   ├── url_processing.py
│   │   └── interactive.py
│   ├── services/                   # Business logic
│   ├── models/                     # SQLAlchemy models
│   ├── api/                        # REST API endpoints
│   └── main.py
├── frontend/                       # React frontend (to be implemented)
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── services/
│   └── package.json
└── README.md                       # This file
```

## Implementation Status

**Current Phase:** Specification Complete ✅

**Next Steps:**
1. Set up project structure (Task 1)
2. Implement Profile Evaluation Agent (Task 3)
3. Implement URL Processing Agent (Task 4)
4. Implement Interactive Agent (Task 5)
5. Build frontend interface (Task 6)
6. Create mock membership form (Task 8)

See `specs/tasks.md` for complete implementation plan.

## Key Design Decisions

### Why 3 Agents?

**Original Design:** 2 agents (URL Processing + Interactive)

**Problem:** Interactive Agent had direct database access, creating security and privacy concerns.

**Solution:** Split into 3 agents:
- Profile Evaluation Agent handles all profile database reads
- Interactive Agent receives assessment results as structured data
- Clear separation of concerns and data boundaries

**Benefits:**
- Enhanced security (Interactive Agent can't access member profiles)
- Better privacy (limits data exposure)
- Cleaner architecture (single responsibility per agent)
- Real-time assessment (continuous monitoring of profile changes)

### Why Filesystem for Artifacts?

- **Independence:** Artifacts separate from database
- **Performance:** No DB queries to load context
- **Versioning:** Easy to track changes over time
- **Portability:** Can move to cloud storage (S3) easily

### Why Copy-to-Clipboard?

- **Member Control:** Humans approve all changes
- **Transparency:** Members see exactly what's being added
- **Flexibility:** Members can edit suggestions before saving
- **Ethical:** AI suggests, human decides

## Production Integration Notes

When integrating with the actual White Rabbit production database:

1. **Table Name:** Change `members` to `creator_profiles`
2. **Authentication:** Add Clerk integration
3. **Social Links:** Convert from normalized table to JSONB array
4. **Traits:** Verify `creator_traits` table structure
5. **Prompt Responses:** Verify `prompt_responses` table structure
6. **Missing Schema:** Request complete schema for `creator_traits` and `prompt_responses`

See `specs/database-schema.md` for detailed production alignment notes.

## Contributing

This is a POC for the White Rabbit Ashland community. For questions or contributions, contact the development team.

## License

[To be determined]

## Resources

- **Spec Documents:** See `specs/` directory for complete requirements, design, and tasks
- **Architecture Diagram:** `specs/WR_ProfileOptimizerMASFlowChart.png`
- **Production Schema:** `specs/WR_MemberDBSchema.txt`
- **GitHub Repository:** https://github.com/bd-aohk/WR-profile-optimizer

---

**Built with ❤️ for the White Rabbit Ashland community**

*Setting the narrative for how we move into the future with new technology and innovation - responsible, transparent, and human-centered AI development.*
