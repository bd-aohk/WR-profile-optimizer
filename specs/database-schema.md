# Database Schema - White Rabbit Ashland Member Profiles

This document defines the database schema that mirrors the Neon DB structure used by the White Rabbit Ashland client system.

## Tables

### members

Primary table storing member profile information.

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique member identifier |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Account creation timestamp |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last profile update timestamp |
| first_name | VARCHAR(100) | NOT NULL | Member's first name |
| last_name | VARCHAR(100) | NOT NULL | Member's last name |
| profile_photo_url | TEXT | NULL | URL to member's profile photo |
| what_you_do | TEXT | NULL | Free-form description of member's work/role |
| where_location | VARCHAR(255) | NULL | Member's location (City, State or City, Country) |
| phone_number | VARCHAR(20) | NULL | Phone number (required for front-door check-in) |
| website | TEXT | NULL | Member's personal or professional website URL |
| location_city_state_country | VARCHAR(255) | NULL | Structured location field |

### social_links

Stores multiple social profile URLs per member (one-to-many relationship).

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique social link identifier |
| member_id | UUID | NOT NULL, FOREIGN KEY → members(id) | Reference to member |
| platform_name | VARCHAR(50) | NULL | Platform identifier (linkedin, twitter, github, etc.) |
| url | TEXT | NOT NULL | Full social profile URL |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | When link was added |
| processed | BOOLEAN | DEFAULT FALSE | Whether URL Processing Agent has analyzed this link |

### skills

Stores member skills as individual records (many-to-many via junction table).

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique skill identifier |
| skill_name | VARCHAR(100) | NOT NULL, UNIQUE | Skill name (e.g., "Research", "Systems design") |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | When skill was first added to system |

### member_skills

Junction table linking members to skills.

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique record identifier |
| member_id | UUID | NOT NULL, FOREIGN KEY → members(id) | Reference to member |
| skill_id | UUID | NOT NULL, FOREIGN KEY → skills(id) | Reference to skill |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | When member added this skill |

**Unique Constraint:** (member_id, skill_id)

### interests

Stores member interests as individual records (many-to-many via junction table).

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique interest identifier |
| interest_name | VARCHAR(100) | NOT NULL, UNIQUE | Interest name (e.g., "Semiotics", "Democracy") |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | When interest was first added to system |

### member_interests

Junction table linking members to interests.

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique record identifier |
| member_id | UUID | NOT NULL, FOREIGN KEY → members(id) | Reference to member |
| interest_id | UUID | NOT NULL, FOREIGN KEY → interests(id) | Reference to interest |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | When member added this interest |

**Unique Constraint:** (member_id, interest_id)

### inspirations

Stores member inspirations as individual records (many-to-many via junction table).

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique inspiration identifier |
| inspiration_name | VARCHAR(200) | NOT NULL, UNIQUE | Name of person, book, or idea |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | When inspiration was first added to system |

### member_inspirations

Junction table linking members to inspirations.

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique record identifier |
| member_id | UUID | NOT NULL, FOREIGN KEY → members(id) | Reference to member |
| inspiration_id | UUID | NOT NULL, FOREIGN KEY → inspirations(id) | Reference to inspiration |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | When member added this inspiration |

**Unique Constraint:** (member_id, inspiration_id)

### prompt_responses

Stores long-form text responses to profile prompts.

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique response identifier |
| member_id | UUID | NOT NULL, FOREIGN KEY → members(id) | Reference to member |
| prompt_key | VARCHAR(100) | NOT NULL | Identifier for the prompt (e.g., "current_focus") |
| prompt_text | TEXT | NOT NULL | The actual prompt question |
| response_text | TEXT | NULL | Member's response |
| character_count | INTEGER | NULL | Length of response |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | When response was created |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | When response was last updated |

**Unique Constraint:** (member_id, prompt_key)

**Standard Prompt Keys:**
- `current_focus` - "What is your current focus?"
- `needs_for_vision` - "What do you need to realize your vision?"
- `motivations` - "What motivates you?"

### conversation_history

Stores conversation history between members and the Interactive Agent.

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique conversation message identifier |
| member_id | UUID | NOT NULL, FOREIGN KEY → members(id) | Reference to member |
| session_id | UUID | NOT NULL | Groups messages in same conversation session |
| role | VARCHAR(20) | NOT NULL | "user" or "assistant" |
| message_content | TEXT | NOT NULL | The message text |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | When message was sent |

### profile_completeness

Tracks profile completeness metrics for each member.

| Column Name | Data Type | Constraints | Description |
|-------------|-----------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique record identifier |
| member_id | UUID | NOT NULL, UNIQUE, FOREIGN KEY → members(id) | Reference to member |
| completeness_score | DECIMAL(5,2) | NOT NULL, DEFAULT 0.00 | Percentage (0.00 to 100.00) |
| missing_fields | JSONB | NULL | Array of missing field identifiers |
| last_calculated | TIMESTAMP | NOT NULL, DEFAULT NOW() | When score was last calculated |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | When record was last updated |



## Indexes

```sql
-- Performance indexes for common queries
CREATE INDEX idx_members_created_at ON members(created_at);
CREATE INDEX idx_social_links_member_id ON social_links(member_id);
CREATE INDEX idx_social_links_processed ON social_links(processed);
CREATE INDEX idx_member_skills_member_id ON member_skills(member_id);
CREATE INDEX idx_member_interests_member_id ON member_interests(member_id);
CREATE INDEX idx_member_inspirations_member_id ON member_inspirations(member_id);
CREATE INDEX idx_prompt_responses_member_id ON prompt_responses(member_id);
CREATE INDEX idx_conversation_history_member_id ON conversation_history(member_id);
CREATE INDEX idx_conversation_history_session_id ON conversation_history(session_id);
```

## Sample Data Structure

### Example Member Record

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "first_name": "Benjamin",
  "last_name": "Durham",
  "what_you_do": "Research, Analysis, Systems Design",
  "where_location": "Ploto",
  "phone_number": "(512) 550-4490",
  "website": "https://example.com",
  "skills": ["Research", "Systems design", "analysis", "Data Design"],
  "interests": ["Psi", "Semiotics", "Influence", "Deception", "Security"],
  "inspirations": ["Michel Foucault", "Charles Peirce", "Heather Marsh"],
  "social_links": [
    {
      "platform_name": "twitter",
      "url": "https://twitter.com/you"
    },
    {
      "platform_name": "linkedin",
      "url": "https://www.linkedin.com/in/..."
    }
  ],
  "prompt_responses": {
    "current_focus": "Currently in the midst of producing materials for fundraising...",
    "needs_for_vision": "1. Better narrative construction...",
    "motivations": "My professional goal is to scale my company..."
  }
}
```

## Social Profile Artifacts (Filesystem Storage)

Social Profile Artifacts generated by the URL Processing Agent are stored as markdown files on the filesystem, **independent of the membership database**. This separation maintains ethical boundaries and allows artifacts to be managed separately from member profile data.

**File Naming Convention:** `user_socialprofile_platformname.md`
- Example: `benjamindurham_socialprofile_linkedin.md`

**Storage Location:** Configured filesystem directory (e.g., `/artifacts/social_profiles/`)

**Artifact Structure:**
```markdown
# Social Profile: [Platform Name]

**Member:** [First Name] [Last Name]
**Platform:** [Platform Name]
**Profile URL:** [URL]
**Extracted:** [ISO 8601 Timestamp]

## Skills
- [Skill 1]
- [Skill 2]

## Interests
- [Interest 1]
- [Interest 2]

## Professional Focus
[Description of current work/focus]

## Location
[City, State/Country]

## Goals & Motivations
[Extracted goals or stated motivations]
```

**Access Pattern:**
- URL Processing Agent: Write-only (creates/updates markdown files)
- Interactive Agent: Read-only (loads artifacts as context)
- Backend System: Manages file I/O operations

## Notes

- All tables use UUID for primary keys for better distribution and security
- Timestamps use UTC timezone
- JSONB used for flexible data structures (missing_fields)
- Many-to-many relationships use junction tables for normalization
- Social links support multiple platforms per member
- Profile completeness is calculated and cached for performance
- Conversation history supports session-based retrieval
- Social profile artifacts are stored on filesystem, not in database
- The `social_links.processed` flag indicates if an artifact has been generated for a URL
