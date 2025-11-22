# Architecture Update: 3-Agent System

## Summary of Changes

The Profile Optimizer System has been updated from a 2-agent to a 3-agent architecture to improve data privacy and security.

## New Architecture

### Three Specialized Agents

1. **Profile Evaluation Agent**
   - **Purpose**: Continuously assess profile completeness and quality
   - **Database Access**: Read-only access to member profiles
   - **Write Access**: Only to profile_completeness table
   - **Responsibilities**:
     - Calculate completeness scores
     - Identify missing/incomplete fields
     - Evaluate field quality (length, detail)
     - Assign priority levels to gaps
     - Trigger reassessment on profile updates

2. **Interactive Agent**
   - **Purpose**: Conduct conversations and generate suggestions
   - **Database Access**: NO direct access to membership database
   - **Write Access**: Only to conversation_history table
   - **Responsibilities**:
     - Receive assessment results as structured data
     - Conduct conversational interviews
     - Generate copy-to-clipboard suggestions
     - Load social profile artifacts for context
     - Adapt questions based on assessment

3. **URL Processing Agent**
   - **Purpose**: Analyze social profiles and generate artifacts
   - **Database Access**: Limited read access (member_id, name only)
   - **Write Access**: Markdown files to filesystem
   - **Responsibilities**:
     - Scrape social profile URLs
     - Extract schema-relevant data
     - Generate markdown artifacts
     - Update processing status

## Key Benefits

1. **Enhanced Security**: Interactive Agent never touches membership database
2. **Clear Separation of Concerns**: Each agent has a single, well-defined responsibility
3. **Better Privacy**: Limits data exposure and attack surface
4. **Real-time Assessment**: Profile Evaluation Agent continuously monitors changes
5. **Cleaner Data Flow**: Assessment → Conversation → Suggestion → Update → Reassessment

## Data Flow

```
User updates profile 
  → Profile Evaluation Agent assesses 
  → Returns structured assessment results
  → Interactive Agent receives assessment (no raw DB data)
  → Generates suggestions based on assessment + artifacts + conversation
  → User copies/pastes suggestions
  → Profile updated
  → Profile Evaluation Agent reassesses
  → Cycle repeats
```

## Updated Requirements

- **Requirement 1**: Updated to show Profile Evaluation Agent triggers first
- **Requirement 2**: Profile Evaluation Agent performs assessment
- **Requirement 2A**: NEW - Continuous assessment on profile changes
- **Requirement 13**: Updated with 3-agent access boundaries
- **Requirement 14-16**: Updated to reference Profile Evaluation Agent for assessment tasks

## Implementation Impact

### New Components to Build

1. **ProfileEvaluationAgent class**
   - assess_profile(member_id) → assessment_results
   - calculate_score(profile, criteria) → float
   - identify_gaps(profile, criteria) → list
   - evaluate_quality(profile, criteria) → dict

2. **Assessment Results Data Structure**
   ```python
   {
     "completeness_score": 65.5,
     "missing_fields": ["skills", "interests"],
     "quality_issues": {
       "what_you_do": "too_short (20 chars, min 50)"
     },
     "priority_recommendations": [
       {"field": "skills", "priority": "high", "reason": "..."},
       {"field": "interests", "priority": "medium", "reason": "..."}
     ]
   }
   ```

3. **Backend API Changes**
   - Profile Evaluation Agent endpoint: POST /api/agent/evaluate-profile
   - Reassessment trigger on profile updates
   - Assessment results passed to Interactive Agent (not raw DB data)

### Modified Components

1. **InteractiveAgent class**
   - Remove direct database access
   - Receive assessment_results as parameter
   - Load artifacts from filesystem
   - Generate suggestions based on assessment context

2. **Agent Orchestrator**
   - Coordinate Profile Evaluation Agent → Interactive Agent flow
   - Handle reassessment triggers
   - Pass structured data between agents

3. **Database Permissions**
   - Profile Evaluation Agent: READ on member tables, WRITE on profile_completeness
   - Interactive Agent: NO access to member tables, WRITE on conversation_history
   - URL Processing Agent: READ on members (id, name only), WRITE to filesystem

## Testing Considerations

- Test Profile Evaluation Agent assessment accuracy
- Test that Interactive Agent cannot access membership database
- Test continuous reassessment on profile updates
- Test assessment results structure and completeness
- Test agent coordination and data flow
