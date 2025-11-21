# Requirements Document

## Introduction

The Profile Optimizer System is a proof-of-concept dual-agent AI architecture designed to enrich member profile data for the White Rabbit Ashland community. The system consists of two specialized agents: the URL Processing Agent (event-driven, triggered by URL uploads) and the Interactive Agent (conversational, engages directly with members). The URL Processing Agent analyzes social profiles and generates structured markdown artifacts that serve as context for the Interactive Agent. The Interactive Agent uses these artifacts along with direct conversation to assess existing profiles and help members build complete profiles. The POC will operate with a mock database that mirrors the existing Neon DB schema used by the client (see #[[file:database-schema.md]] for complete schema definition). The system operates under strict ethical principles: all data collection is opt-in, AI suggestions require human approval before publication, and the system maintains clear boundaries between private conversation context and public profile data. The strategic goal is to ensure sufficient profile completeness across all members to enable pattern identification, illuminate collaborative potential between members, and support future operational capabilities that depend on rich member data.

## Glossary

- **URL Processing Agent**: The event-driven AI agent that analyzes social profile URLs and generates markdown artifacts
- **Interactive Agent**: The conversational AI agent that conducts interviews and suggests profile improvements to members
- **Member**: A White Rabbit Ashland community member who interacts with the Interactive Agent
- **Social Profile Artifact**: A markdown file (format: `user_socialprofile_platformname.md`) containing structured data extracted from a social profile
- **Profile Data**: Information about a member stored in the system, including both public and private data
- **Conversation Context**: Information shared during private chat sessions with the Interactive Agent
- **Public Profile**: Member information visible to other community members
- **Completeness Score**: A metric indicating how much of a member's profile has been filled out
- **Profile Assessment**: The process by which the Interactive Agent evaluates a member's current profile to identify gaps and prioritize data collection
- **POC**: Proof of Concept - the initial implementation demonstrating the dual-agent architecture and core functionality
- **Web Interface**: The browser-based chat application where members interact with the Interactive Agent
- **Backend System**: The FastAPI server that processes requests, manages data, and coordinates both agents
- **Database**: The PostgreSQL database storing member profiles and conversation history
- **Mock Database**: A local database instance that mirrors the Neon DB schema for POC development and testing
- **Neon DB Schema**: The existing database schema used by the White Rabbit Ashland client system (defined in #[[file:database-schema.md]])
- **Platform Name**: The social media or professional networking platform (e.g., linkedin, twitter, github)

## Requirements

### Requirement 1

**User Story:** As a White Rabbit member, I want to activate an AI agent to help optimize my profile through conversation, so that I can build a rich profile at my own pace

#### Acceptance Criteria

1. WHEN a Member clicks the profile optimizer button in the Web Interface, THE Backend System SHALL retrieve the Member's existing profile from the Database
2. THE Interactive Agent SHALL perform a Profile Assessment to calculate a Completeness Score and identify gaps before initiating conversation
3. THE Interactive Agent SHALL present the assessment results to the Member, explaining which profile sections need attention and why they are valuable
4. THE Interactive Agent SHALL prioritize interview questions based on the Profile Assessment, focusing on missing or incomplete fields
5. WHILE conducting an interview, THE Interactive Agent SHALL ask targeted questions in natural language about specific gaps (e.g., missing skills, short bio, absent interests)
6. WHEN a Member responds to interview questions, THE Backend System SHALL store the conversation in the Database
7. THE Interactive Agent SHALL operate in read-only mode and SHALL NOT write directly to the Member's profile in the Database
8. WHEN the Interactive Agent generates profile suggestions, THE Web Interface SHALL provide a copy-to-clipboard button for each suggestion
9. THE Member SHALL manually update their profile fields using the copied suggestions

### Requirement 2

**User Story:** As a White Rabbit member, I want the agent to provide a detailed assessment of my profile quality, so that I understand what needs improvement and why it matters

#### Acceptance Criteria

1. THE Interactive Agent SHALL calculate a Completeness Score as a percentage based on filled versus total profile fields defined in #[[file:database-schema.md]]
2. THE Interactive Agent SHALL evaluate profile field quality, identifying fields that are too short, vague, or lack detail (e.g., bio under 50 characters)
3. THE Interactive Agent SHALL identify specific missing or incomplete profile sections including: first_name, last_name, what_you_do, location, skills, interests, inspirations, and prompt_responses
4. THE Interactive Agent SHALL assign priority levels to missing fields based on strategic importance for collaboration matching and pattern identification
5. WHEN presenting assessment results, THE Interactive Agent SHALL communicate gaps in natural language with specific examples (e.g., "Your bio is only 20 characters - adding more detail about your work would help others understand your expertise")
6. THE Interactive Agent SHALL explain why each missing piece of information is valuable for community collaboration and member discovery

### Requirement 3

**User Story:** As a White Rabbit member, I want to upload social profile URLs that are automatically processed, so that the interactive agent has rich context about my professional background

#### Acceptance Criteria

1. WHEN a Member uploads a social profile URL, THE Backend System SHALL trigger the URL Processing Agent
2. WHEN triggered, THE URL Processing Agent SHALL retrieve and analyze the social profile content
3. THE URL Processing Agent SHALL extract ONLY data elements that correspond to fields defined in the database schema as specified in #[[file:database-schema.md]]
4. THE URL Processing Agent SHALL generate a Social Profile Artifact as a markdown file named `user_socialprofile_platformname.md` containing only schema-relevant data
5. THE Social Profile Artifact SHALL be limited to extracting: skills, interests, professional experience descriptions, location, and publicly stated goals or motivations
6. WHEN the Social Profile Artifact markdown file is created, THE URL Processing Agent SHALL save it to the filesystem independent of the membership Database
7. THE Interactive Agent SHALL read the markdown artifact files from the filesystem when providing context-aware suggestions

### Requirement 4

**User Story:** As a White Rabbit member, I want to manually control what AI-generated suggestions become part of my profile, so that I maintain full control over my published information

#### Acceptance Criteria

1. WHEN the Interactive Agent generates a profile suggestion, THE Web Interface SHALL display the suggestion to the Member with a copy-to-clipboard button
2. THE Interactive Agent SHALL NOT write any content directly to the Member's profile in the Database
3. WHEN a Member clicks the copy-to-clipboard button, THE Web Interface SHALL copy the suggestion text to the system clipboard
4. THE Member SHALL manually paste and edit the copied suggestion into their profile fields
5. THE Backend System SHALL only update the Public Profile when the Member directly edits and saves their profile through the standard profile editing interface

### Requirement 5

**User Story:** As a White Rabbit member, I want the system to distinguish between what I share in private conversation and what the agent suggests for my public profile, so that my privacy is protected

#### Acceptance Criteria

1. THE Backend System SHALL maintain separate storage for Conversation Context (conversation_history table) and Public Profile data (member profile tables)
2. WHEN storing conversation data, THE Backend System SHALL store it in the conversation_history table, which is private and not visible to other members
3. THE Interactive Agent SHALL use conversation context to inform suggestions but SHALL NOT include sensitive or private information in profile suggestions
4. WHERE a Member shares sensitive information in conversation (e.g., personal challenges, financial details, health information), THE Interactive Agent SHALL exclude such information from any profile suggestions
5. THE Interactive Agent SHALL only generate suggestions for profile fields that are appropriate for public visibility within the community

### Requirement 6

**User Story:** As a White Rabbit member, I want to access the profile optimizer through a web-based chat interface, so that I can interact with the agent from any device

#### Acceptance Criteria

1. THE Web Interface SHALL provide a chat-based conversation interface accessible via web browser
2. WHEN a Member sends a message, THE Web Interface SHALL transmit the message to the Backend System within 2 seconds
3. WHEN the Backend System generates a response, THE Web Interface SHALL display the response to the Member within 5 seconds
4. THE Web Interface SHALL display conversation history for the current session
5. THE Web Interface SHALL be responsive and functional on desktop and mobile devices

### Requirement 7

**User Story:** As a system administrator, I want all member data to be stored securely with proper authentication, so that member privacy is protected

#### Acceptance Criteria

1. THE Backend System SHALL require authentication before allowing access to member Profile Data
2. THE Database SHALL encrypt sensitive member information at rest
3. WHEN a Member logs in, THE Backend System SHALL verify credentials before granting access
4. THE Backend System SHALL maintain audit logs of all access to member Profile Data
5. THE Backend System SHALL enforce role-based access control for administrative functions

### Requirement 8

**User Story:** As a White Rabbit member, I want the agent to respect my communication preferences and boundaries, so that I can engage at my own pace

#### Acceptance Criteria

1. THE Interactive Agent SHALL allow Members to pause or end conversations at any time
2. WHERE a Member indicates they want to stop, THE Interactive Agent SHALL save progress and allow resumption later
3. THE Interactive Agent SHALL not send unsolicited messages or notifications to Members
4. WHEN a Member returns to a previous conversation, THE Backend System SHALL restore the conversation context
5. THE Interactive Agent SHALL adapt its communication style based on Member feedback during conversation


### Requirement 9

**User Story:** As the Interactive Agent, I want to access Social Profile Artifacts as supplementary context during conversation, so that I can provide informed recommendations based on the member's social presence

#### Acceptance Criteria

1. WHEN a Member has uploaded social profile URLs and the URL Processing Agent has generated artifacts, THE Backend System SHALL check the filesystem for corresponding Social Profile Artifact markdown files
2. THE Interactive Agent SHALL load available Social Profile Artifacts from the filesystem before engaging with the Member
3. WHILE conducting a conversation, THE Interactive Agent SHALL use information from Social Profile Artifacts to pre-populate suggested profile content (e.g., skills extracted from LinkedIn can be suggested for the skills field)
4. WHEN generating profile suggestions, THE Interactive Agent SHALL synthesize information from both Social Profile Artifacts and direct conversation responses
5. THE Interactive Agent SHALL cite the source when using artifact data in suggestions (e.g., "Based on your LinkedIn profile, I noticed you have experience with...")
6. THE Interactive Agent SHALL distinguish between information from Social Profile Artifacts and information from direct conversation
7. WHERE no Social Profile Artifacts exist for a Member, THE Interactive Agent SHALL conduct interviews using only conversational data and existing profile information

### Requirement 10

**User Story:** As a system administrator, I want the URL Processing Agent to operate independently from user interactions, so that profile analysis doesn't block conversational flow

#### Acceptance Criteria

1. WHEN a URL upload event occurs, THE Backend System SHALL trigger the URL Processing Agent asynchronously
2. THE URL Processing Agent SHALL process social profile URLs without blocking the Interactive Agent
3. WHEN the URL Processing Agent completes processing, THE Backend System SHALL notify the Interactive Agent that new context is available
4. IF the URL Processing Agent encounters an error, THEN THE Backend System SHALL log the error without disrupting the Interactive Agent
5. THE Backend System SHALL queue multiple URL processing requests and handle them sequentially

### Requirement 11

**User Story:** As a White Rabbit member, I want my social profile data to be stored in a standardized format with only necessary information extracted, so that my privacy is respected and data can be easily reviewed

#### Acceptance Criteria

1. THE URL Processing Agent SHALL generate Social Profile Artifacts using a consistent markdown structure that maps directly to database schema fields
2. THE Social Profile Artifact SHALL include metadata sections for platform, extraction date, and profile URL
3. THE Social Profile Artifact SHALL organize extracted data into clearly labeled sections corresponding to schema fields: Skills, Interests, Professional Focus, Location, and Goals
4. THE URL Processing Agent SHALL NOT extract or store personal contact information, private messages, or any data not explicitly defined in the database schema
5. WHEN a Member requests to view their social profile data, THE Web Interface SHALL display the Social Profile Artifact in readable format
6. THE Backend System SHALL version Social Profile Artifacts when profiles are re-analyzed


### Requirement 12

**User Story:** As a developer building the POC, I want the system to use a mock database that mirrors the Neon DB schema, so that I can develop and test the agents without affecting production data

#### Acceptance Criteria

1. THE Backend System SHALL connect to a Mock Database that replicates the Neon DB Schema structure as defined in #[[file:database-schema.md]]
2. THE Mock Database SHALL include sample member profile data for testing agent functionality
3. THE Backend System SHALL support the same database operations on the Mock Database as would be used with the production Neon DB
4. THE Mock Database SHALL implement all tables defined in the schema including members, social_links, skills, interests, inspirations, prompt_responses, conversation_history, profile_completeness, and social_profile_artifacts
5. THE Backend System SHALL provide configuration to switch between Mock Database and production Neon DB connections

### Requirement 13

**User Story:** As a system architect, I want both agents to operate in read-only mode with respect to member profiles, so that we maintain ethical boundaries and member control over their data

#### Acceptance Criteria

1. THE Interactive Agent SHALL have read-only access to member profile data in the Database
2. THE URL Processing Agent SHALL have read-only access to member profile data in the Database
3. THE Backend System SHALL enforce database permissions that prevent both agents from writing to member profile tables
4. THE Interactive Agent SHALL only write to conversation_history table
5. THE URL Processing Agent SHALL only write markdown artifact files to the filesystem, independent of the membership Database
6. THE Backend System SHALL log any attempted unauthorized write operations by agents

### Requirement 14

**User Story:** As a community administrator, I want to ensure all member profiles meet a minimum completeness threshold, so that we can identify collaboration patterns and enable future operational capabilities

#### Acceptance Criteria

1. THE Backend System SHALL define minimum required profile fields necessary for pattern identification and collaboration matching
2. THE Interactive Agent SHALL track progress toward minimum completeness for each Member
3. WHEN a Member's profile falls below the minimum completeness threshold, THE Interactive Agent SHALL prioritize collecting missing critical information
4. THE Backend System SHALL generate reports on overall community profile completeness across all 80+ members
5. THE Interactive Agent SHALL collect profile data in a structured format that enables automated pattern identification and collaboration matching algorithms
