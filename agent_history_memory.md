# Agent History and Memory (Tool)

**Stack:** DynamoDB as the canonical, synchronous memory store and Amazon Bedrock through the Converse API for conversation, extraction, project generation, and tool use. Converse supports conversational inference, Guardrails, and model tool calls.

**Behavior:** For this agent, I would treat memory as an evolving, evidence-based user profile, not as a transcript archive and not as a fixed personality classification.

The MVP should answer these questions:

- What has the user explicitly told me?
- What preferences can I cautiously infer?
- Which hobby projects have already been suggested, accepted, rejected, or completed?

## 1. What the agent should remember

### A. Explicit facts

Information directly stated by the user:

```json
{
    "category": "time_availability",
    "value": "2 hours per week",
    "source": "explicit",
    "confidence": 1.0
}
```

Examples:

“I only have time on weekends.”

“I prefer doing things alone.”

“I live in an apartment.”

“I do not want an expensive hobby.”

“I enjoy technology and history.”

Explicit information should have the highest priority.

### B. Inferred preferences

Interpretations based on the conversation:

```json
{
    "category": "learning_style",
    "value": "project_based",
    "source": "inferred",
    "confidence": 0.65,
    "evidence": [
        "User said they prefer learning by building something."
    ]
}
```

The agent should never present these as unquestionable facts.

Instead of:

You are an introverted person.

It should think:

The user has previously preferred solo activities.

This distinction makes the memory more accurate and less intrusive.

### C. Hobby history

The agent must remember:

- Hobbies already suggested
- Projects generated
- Projects accepted
- Projects rejected
- Reasons for rejection
- Progress reported
- Completed projects
- Abandoned projects

This prevents repetitive recommendations.

### D. Constraints

- Constraints may matter more than personality:
- Available time
- Budget range
- Available physical space
- Indoor versus outdoor
- Physical intensity
- Accessibility needs
- Equipment availability
- Desired commitment
- Solo versus group
- Digital versus physical

### E. Motivations

The same hobby can satisfy different needs:

- Relaxation
- Creative expression
- Social connection
- Physical activity
- Intellectual challenge
- Building practical skills
- Collecting
- Creating something tangible
- Career development

## 2. Recommended personality model

Avoid MBTI-style classification in the MVP. Store a small number of continuous preference dimensions.

```json
{
    "social_mode": {
        "value": "mostly_solo",
        "confidence": 0.8
    },
    "activity_level": {
        "value": "low_to_moderate",
        "confidence": 0.6
    },
    "structure_preference": {
        "value": "guided",
        "confidence": 0.7
    },
    "creative_orientation": {
        "value": "high",
        "confidence": 0.75
    },
    "learning_style": {
        "value": "hands_on",
        "confidence": 0.9
    },
    "novelty_preference": {
        "value": "moderate",
        "confidence": 0.5
    }
}
```

A useful MVP set would be:

- social_mode
- activity_level
- indoor_outdoor
- creative_analytical
- guided_exploratory
- digital_physical
- short_long_projects
- learning_style
- novelty_preference
- primary_motivation

## 3. DynamoDB memory model

For the MVP, one DynamoDB table is sufficient.

AWS commonly recommends considering single-table design, and composite sort keys are useful for grouping related data and querying subsets with operations such as begins_with

Table: HobbyAgentMemory

```text
PK                  SK
USER#123            PROFILE#CURRENT
USER#123            MEMORY#2026-07-25T14:30:00Z#abc
USER#123            SESSION#session-001
USER#123            PROJECT#project-001
USER#123            FEEDBACK#project-001#2026-07-25
USER#123            HOBBY#PHOTOGRAPHY
```

### Profile item

```json
{
    "PK": "USER#123",
    "SK": "PROFILE#CURRENT",
    "entity_type": "USER_PROFILE",
    "profile_version": 7,
    "preferences": {
        "social_mode": {
            "value": "mostly_solo",
            "confidence": 0.8,
            "memory_ids": ["mem-001", "mem-009"]
        },
        "learning_style": {
            "value": "hands_on",
            "confidence": 0.9,
            "memory_ids": ["mem-003"]
        }
    },
    "constraints": {
        "weekly_time_minutes": 120,
        "budget_level": "low",
        "available_space": "apartment"
    },
    "updated_at": "2026-07-25T14:30:00Z"
}
```

### Individual memory item

```json
{
    "PK": "USER#123",
    "SK": "MEMORY#2026-07-25T14:30:00Z#mem-009",
    "entity_type": "MEMORY",
    "memory_id": "mem-009",
    "category": "social_mode",
    "value": "mostly_solo",
    "source": "explicit",
    "confidence": 1.0,
    "evidence": "I normally prefer hobbies I can do alone.",
    "session_id": "session-001",
    "status": "active",
    "created_at": "2026-07-25T14:30:00Z"
}
```

### Project item

```json
{
    "PK": "USER#123",
    "SK": "PROJECT#project-001",
    "entity_type": "HOBBY_PROJECT",
    "project_id": "project-001",
    "hobby": "Urban photography",
    "status": "proposed",
    "recommendation_reasons": [
        "Can be done independently",
        "Works with short weekend sessions",
        "Combines technology and creativity"
    ],
    "memory_ids_used": [
        "mem-003",
        "mem-009",
        "mem-012"
    ],
    "created_at": "2026-07-25T14:35:00Z"
}
```

The memory_ids_used field is especially important. It lets you trace why the agent made a recommendation and test whether it invented unsupported characteristics.

## 4. Memory lifecycle

### Step 1 — Load relevant context

At the beginning of an interaction, Lambda retrieves:

```text
PROFILE#CURRENT
Last session summary
Active hobby projects
Rejected hobbies
Recent relevant memories
```

Do not load the complete conversation history into every prompt.

```json
{
    "profile": {},
    "active_project": {},
    "rejected_hobbies": [],
    "recent_relevant_memories": [],
    "current_message": "..."
}
```

### Step 2 — Generate the response

- Claude receives:
- Current message
- Curated profile
- Relevant memories
- Existing projects
- Hobby-project instructions

The model should not receive every stored event.

### Step 3 — Extract memory candidates

After generating the user-facing response, execute a separate structured extraction.

```json
{
    "memory_candidates": [
        {
            "category": "indoor_outdoor",
            "value": "indoor",
            "source": "explicit",
            "confidence": 1.0,
            "evidence": "I would rather do something at home."
        }
    ]
}
```

Separating conversation generation from memory extraction makes the behavior easier to test.

### Step 4 — Validate before saving

Lambda applies deterministic rules:

Is the information useful in future sessions?

Is it about the user?

Is it supported by the message?

Is it sufficiently specific?

Is it sensitive?

Does it contradict an existing memory?

The model proposes memories; the application decides whether they are stored.

### Step 5 — Consolidate the profile

When the new memory agrees with an existing preference:

Increase evidence count

Update last_confirmed_at

Potentially raise confidence

When it contradicts an existing preference:

Do not silently replace it

Mark the old memory as superseded

Create the new memory

Reduce profile certainty if needed

```json
{
    "old_memory": {
        "value": "solo",
        "status": "superseded"
    },
    "new_memory": {
        "value": "small_groups",
        "status": "active",
        "reason": "user_correction"
    }
}
```

## 5. Use Bedrock tool calls for memory operations

Instead of allowing Claude to write directly to DynamoDB, expose controlled tools:

- get_user_profile
- search_user_memories
- save_memory_candidate
- create_hobby_project
- record_project_feedback
- forget_memory

Example tool contract:

```json
{
    "name": "save_memory_candidate",
    "description": "Submit a supported user preference for validation.",
    "inputSchema": {
        "json": {
            "type": "object",
            "properties": {
                "category": {
                    "type": "string"
                },
                "value": {},
                "source": {
                    "enum": ["explicit", "inferred"]
                },
                "confidence": {
                    "type": "number",
                    "minimum": 0,
                    "maximum": 1
                },
                "evidence": {
                    "type": "string"
                }
            },
            "required": [
                "category",
                "value",
                "source",
                "confidence",
                "evidence"
            ]
        }
    }
}
```

Claude requests the tool call, but Lambda still validates and executes it. This creates a clean boundary between probabilistic reasoning and deterministic persistence.

## 6. Hobby-project output

The agent should generate more than a hobby name. It should produce a small, executable project.

```json
{
    "project_name": "Photograph Your Neighborhood",
    "hobby": "Urban photography",
    "why_it_fits": [
        "You prefer independent activities.",
        "You enjoy combining technology with creativity.",
        "The project can be completed in short weekend sessions."
    ],
    "objective": "Create a collection of 12 photographs representing your neighborhood.",
    "duration": "4 weeks",
    "weekly_commitment": "Two 45-minute sessions",
    "starter_materials": [
        "A smartphone or camera",
        "A free photo-editing application",
        "A folder for organizing the photographs"
    ],
    "steps": [
        {
            "week": 1,
            "goal": "Learn framing and choose three locations."
        },
        {
            "week": 2,
            "goal": "Photograph textures, signs, and architecture."
        },
        {
            "week": 3,
            "goal": "Select and edit the best photographs."
        },
        {
            "week": 4,
            "goal": "Create a digital album with 12 photographs."
        }
    ],
    "success_criteria": [
        "Complete the 12-photo collection",
        "Identify which photography style was most enjoyable"
    ],
    "first_action": "Take three photographs within 200 meters of your home.",
    "memory_ids_used": [
        "mem-003",
        "mem-009"
    ]
}
```

The explanation should refer only to supported memories. It should never say something like “because you have an artistic personality” unless the user explicitly said that or sufficient evidence exists.

## 7. Feedback is the most valuable memory

After suggesting a project, collect lightweight feedback:

- Interested
- Not interested
- Maybe later
- Already tried it
- Started
- Completed
- Abandoned

Then ask for one reason:

- Too expensive
- Too much time
- Too social
- Too physically demanding
- Not interesting
- Already familiar
- Missing equipment
- Other

Example feedback memory:

```json
{
    "PK": "USER#123",
    "SK": "FEEDBACK#project-001#2026-08-02T13:00:00Z",
    "entity_type": "PROJECT_FEEDBACK",
    "project_id": "project-001",
    "decision": "rejected",
    "reason": "requires_too_much_travel",
    "derived_memory_candidates": [
        {
            "category": "location_constraint",
            "value": "prefers_home_or_nearby_activities",
            "confidence": 0.8
        }
    ]
}
```

Behavioral feedback should influence future recommendations more strongly than speculative personality inference.

## 8. Memory retention and privacy

The user should be able to say:

- What do you remember about me?
- Correct my preference.
- Forget that information.
- Reset my hobby profile.

For the MVP, expose:

- GET /users/{userId}/profile
- PATCH /users/{userId}/profile
- GET /users/{userId}/memories
- DELETE /users/{userId}/memories/{memoryId}
- DELETE /users/{userId}/memory
- POST /projects/{projectId}/feedback

Apply TTL only to raw conversation events and temporary session summaries. Keep confirmed profile preferences, projects, and feedback until the user removes them.

DynamoDB TTL deletion is asynchronous and expired items can remain visible until the background deletion occurs, so application queries should also filter expired records rather than depending exclusively on physical deletion.

For sensitive information, use Bedrock Guardrails and application-level validation. Guardrails can detect, block, or mask categories of PII, although the detection is probabilistic and should not replace access controls or your own validation.

Do not infer or store:

- Political affiliation
- Religion
- Medical conditions
- Sexual orientation
- Financial details
- Precise address
- Other sensitive characteristics

unless there is a clear feature requirement, explicit user intent, and appropriate controls.

## 9. Recommended system-prompt rules

You are a hobby project recommendation agent.

Use user memories only when they are relevant to the current request.

Distinguish explicit facts from inferred preferences.

Never present an inferred preference as a confirmed personality trait.

Every explanation of why a hobby fits must be supported by one or more memory IDs supplied in the context.

Do not infer or store sensitive personal characteristics.

Do not recommend a previously rejected hobby unless the user's preferences or constraints have materially changed.

When confidence is low, phrase the preference as a hypothesis or ask one short clarifying question.

Generate a concrete project with an objective, duration, weekly commitment, steps, first action, and success criteria.

Submit new memory candidates through the memory tool. Never claim that a memory was saved until the tool confirms it.
