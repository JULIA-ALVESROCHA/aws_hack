# Guided Hobby Project Execution, Skill Validation, and Badge Unlocking
## Complete Feature Specification and Claude Code Implementation Guide

> **Document status:** Implementation specification  
> **Primary audience:** Claude Code and the engineering team  
> **Feature ownership:** Guided project execution, step progression, project validation, soft-skill practice evidence, and badge eligibility  
> **Architecture style:** Serverless, event-driven where useful, deterministic domain rules, AI-assisted content generation  
> **MVP principle:** The AI may assist, explain, and evaluate structured evidence, but it must not directly award badges or claim that a user has mastered a soft skill.

---

# 1. Purpose of This Document

This document defines the complete product, business, technical, data, validation, integration, state-management, user-experience, testing, security, and implementation requirements for the **Guided Hobby Project Execution and Badge Validation** feature.

The original concept was to generate one complete PDF containing the entire hobby project and then make a badge available after the user finished it. The product direction has changed.

The new experience is **step-based and interactive**.

The user does not simply receive a document and leave the application. Instead, the user progresses through a structured project inside the product:

```text
Selected hobby
    ↓
Personalized project created
    ↓
Project started
    ↓
Step 1 completed and validated
    ↓
Step 2 unlocked
    ↓
...
    ↓
Final project evidence submitted
    ↓
Soft-skill practice assessment completed
    ↓
Deterministic badge-eligibility rules evaluated
    ↓
Badge unlocked
```

The reference interface shared by the design team uses:

- a project title;
- a difficulty level;
- a vertical sequence of steps;
- a clearly highlighted current step;
- guided instructions;
- conversational prompts;
- progress indicators;
- final validation before reward.

This specification translates that experience into a coherent technical implementation.

---

# 2. Core Product Decision

## 2.1 The Badge Must Not Be Unlocked by a Single Button

The badge must not be released only because the user clicks:

```text
I finished the project
```

It must also not be released only because the user answers a quiz correctly.

A quiz can verify understanding, but it cannot prove that the user actually completed a practical hobby project or practiced a soft skill.

The badge must be unlocked by a combination of:

1. mandatory project steps completed;
2. required evidence submitted;
3. project-specific checkpoints passed;
4. final deliverable confirmed;
5. soft-skill reflection completed;
6. behavioral evidence evaluated;
7. final validation score reaching the configured threshold;
8. no unresolved mandatory validation failure;
9. explicit user confirmation that the project is finished.

---

## 2.2 What the Badge Represents

The badge must represent:

> Verified completion of a guided hobby project containing structured opportunities to practice behaviors associated with a target soft skill.

The badge must **not** represent:

- professional certification;
- psychological evaluation;
- formal proof of mastery;
- guaranteed soft-skill development;
- employment qualification;
- clinical or academic assessment.

Recommended badge wording:

```text
Completed a guided project in [HOBBY] and demonstrated practice of behaviors associated with [SOFT_SKILL].
```

Avoid:

```text
Certified expert in communication.
```

---

# 3. Feature Name and Responsibility

Recommended technical feature name:

```text
GuidedProjectExecution
```

Recommended full domain name:

```text
Guided Hobby Project Execution and Validation
```

This feature owns:

- project execution state;
- step progression;
- step completion;
- step validation;
- evidence submission;
- project progress;
- final project assessment;
- badge eligibility;
- badge issuance metadata;
- optional project export.

This feature does **not** own:

- initial hobby recommendation;
- personality profiling;
- user authentication;
- full profile creation;
- general chat memory;
- public badge sharing;
- LinkedIn publishing;
- payment;
- material purchasing;
- image-generation infrastructure owned by another feature;
- global design-system implementation.

---

# 4. Integration Boundary: Preventing a “Frankenstein” Architecture

Every feature must have an explicit boundary.

No feature should directly modify another feature's internal data model.

## 4.1 Upstream Inputs

This feature receives data from other features.

### User Profile Feature

Expected output:

```json
{
  "userId": "user-123",
  "sessionId": "session-123",
  "name": "Alice",
  "budget": {
    "currency": "BRL",
    "maximum": 80
  },
  "weeklyMinutes": 180,
  "preferredSessionMinutes": 45,
  "preferredEnvironment": ["indoor", "quiet"],
  "socialPreference": "solo",
  "spaceConstraints": ["small-room"],
  "availableMaterials": ["paper", "colored-pencils"],
  "accessibilityPreferences": [],
  "preferredLearningStyle": "hands-on"
}
```

This feature may read this object.

It must not redefine profile semantics or write profile fields.

---

### Hobby Recommendation Feature

Expected output:

```json
{
  "recommendationId": "rec-123",
  "hobbyId": "watercolor-painting",
  "hobbyName": "Watercolor Painting",
  "category": "visual-arts",
  "targetSoftSkill": "communication",
  "recommendationReason": "..."
}
```

This feature begins only after one hobby has been selected.

It must not recommend a new hobby unless the upstream recommendation feature explicitly requests regeneration.

---

### Project Content Generation Feature

If content generation is implemented separately, its output must follow the canonical `ProjectDefinition` contract defined in this document.

The execution feature must not depend on free-form markdown as its source of truth.

---

## 4.2 Downstream Outputs

This feature provides:

- current project state;
- unlocked step;
- progress percentage;
- validation feedback;
- project completion state;
- badge eligibility state;
- badge metadata;
- optional PDF/export data.

---

## 4.3 Shared Integration Rule

Every cross-feature interaction must use one of:

1. a versioned JSON contract;
2. a documented function interface;
3. a documented domain event;
4. a documented API endpoint.

Do not:

- read another module's private variables;
- duplicate the same status logic in UI and Lambda;
- infer progress from chat messages;
- allow the model to invent database identifiers;
- allow the model to directly set `badgeIssued = true`;
- store critical state only inside conversation memory.

---

# 5. Product Experience

## 5.1 Main User Flow

```text
1. User selects a hobby
2. Personalized project is generated
3. User previews project title, goal, difficulty, duration, and materials
4. User starts the project
5. First step becomes active
6. User reads instructions and completes the task
7. User submits the required completion input
8. System validates the submission
9. If valid, next step is unlocked
10. If invalid or incomplete, the current step remains active
11. User receives clear feedback and may retry
12. Final practical step is completed
13. User completes final reflection and assessment
14. System evaluates project-level completion rules
15. Badge eligibility is granted
16. Badge is issued
```

---

## 5.2 Interface Model

The interface should support the following visual structure:

```text
PROJECT HEADER
- Project title
- Hobby
- Difficulty
- Target soft skill
- Progress
- Status

LEFT OR TOP STEP NAVIGATION
- Step number
- Step title
- Short description
- Locked / Available / Active / Submitted / Completed / Needs revision

MAIN STEP CONTENT
- Step objective
- Detailed instructions
- Materials needed
- Estimated time
- Safety notes
- Example image or image prompt
- Mentor-style prompts
- Submission controls
- Validation feedback

BOTTOM ACTION AREA
- Save progress
- Submit step
- Retry
- Continue to next step
```

The exact responsive layout is owned by design/frontend, but status names and behaviors must follow this specification.

---

# 6. Domain Terminology

## Project Definition

The immutable or versioned plan describing what the project contains.

## Project Run

One user's active execution of a project definition.

## Step

One actionable stage within the project.

## Step Attempt

One submission made by the user for a step.

## Evidence

Information submitted to support that the user performed the activity.

## Validation

The process of determining whether a submission satisfies configured criteria.

## Checkpoint

A significant milestone within the project.

## Final Assessment

The project-level assessment performed after mandatory steps are completed.

## Soft-Skill Practice Evidence

Evidence that the user engaged in behaviors associated with the selected soft skill.

## Badge Eligibility

A deterministic state indicating that all badge rules are satisfied.

## Badge Issuance

Creation of a badge record after eligibility has been confirmed.

---

# 7. Project Lifecycle State Machine

The backend is the source of truth.

The frontend must render state returned by the backend.

## 7.1 Project Run Status

Allowed values:

```text
created
ready
in_progress
awaiting_final_validation
validation_failed
completed
badge_eligible
badge_issued
abandoned
expired
```

## 7.2 Valid State Transitions

```text
created
  → ready

ready
  → in_progress
  → abandoned

in_progress
  → awaiting_final_validation
  → abandoned
  → expired

awaiting_final_validation
  → validation_failed
  → completed

validation_failed
  → awaiting_final_validation
  → abandoned

completed
  → badge_eligible

badge_eligible
  → badge_issued
```

No other transitions are valid unless the domain model is versioned.

---

## 7.3 Transition Rules

### `created → ready`

Occurs after:

- project definition is valid;
- mandatory steps exist;
- validation rules exist;
- content is saved.

### `ready → in_progress`

Occurs after the user explicitly starts the project.

### `in_progress → awaiting_final_validation`

Occurs after:

- all mandatory steps are completed;
- all mandatory checkpoints are passed;
- final deliverable evidence exists;
- no blocking step remains.

### `awaiting_final_validation → completed`

Occurs after the final assessment passes.

### `completed → badge_eligible`

Occurs after deterministic badge rules pass.

### `badge_eligible → badge_issued`

Occurs after the badge record is successfully created.

---

# 8. Step Lifecycle State Machine

Allowed values:

```text
locked
available
active
submitted
needs_revision
completed
skipped
```

## 8.1 Valid Transitions

```text
locked → available
available → active
active → submitted
submitted → completed
submitted → needs_revision
needs_revision → active
available → skipped
```

A mandatory step cannot become `skipped`.

---

## 8.2 Unlocking Logic

The next step is unlocked only when:

- all dependency steps are completed;
- the current step passed its required validation;
- no blocking safety acknowledgement is missing;
- the project run is still active.

For a simple linear project:

```text
Step N is available when Step N - 1 is completed.
```

For a branched project:

```text
Step N is available when all step IDs in dependsOn are completed.
```

The AI does not decide whether a step is unlocked.

A deterministic domain service does.

---

# 9. Canonical Project Definition

The project definition is the plan generated before execution.

Example:

```json
{
  "schemaVersion": "1.0",
  "projectDefinitionId": "projdef-123",
  "title": "Street Flora Index",
  "hobby": {
    "id": "cyanotype",
    "name": "Cyanotype Printing",
    "category": "visual-arts"
  },
  "difficulty": {
    "level": 2,
    "label": "apprentice"
  },
  "targetSoftSkill": {
    "id": "critical-thinking",
    "label": "Critical Thinking"
  },
  "summary": "Create a visual index of plants found on your block.",
  "finalDeliverable": "A collection of twelve cyanotype plant prints.",
  "estimatedTotalMinutes": 420,
  "estimatedDurationDays": 14,
  "budget": {
    "currency": "BRL",
    "minimum": 35,
    "recommended": 70
  },
  "steps": [],
  "finalAssessmentDefinition": {},
  "badgeDefinition": {},
  "createdAt": "2026-07-25T12:00:00.000Z",
  "version": 1
}
```

---

# 10. Canonical Step Definition

```json
{
  "stepId": "step-03",
  "order": 3,
  "title": "Expose in direct sun",
  "shortDescription": "Find the correct exposure time yourself",
  "type": "practical",
  "mandatory": true,
  "dependsOn": ["step-02"],
  "estimatedMinutes": 30,
  "objective": "Determine the best exposure time through observation and testing.",
  "instructions": [
    {
      "order": 1,
      "text": "Place the coated test strip in direct sunlight."
    },
    {
      "order": 2,
      "text": "Cover sections of the strip at 5, 10, and 15 minutes."
    },
    {
      "order": 3,
      "text": "Wash the strip and compare the tones after drying."
    }
  ],
  "materials": [
    "coated test strip",
    "opaque card",
    "timer",
    "water"
  ],
  "location": {
    "recommended": "Outdoor area with direct sunlight",
    "alternatives": [
      "Bright balcony",
      "Window area only if direct sunlight is available"
    ]
  },
  "safetyNotes": [
    "Do not stare directly at the sun.",
    "Protect sensitized paper from moisture before exposure."
  ],
  "mentorPrompts": [
    "Guess the exposure time before starting the timer.",
    "What made you choose that number?",
    "What did the grey area teach you?"
  ],
  "submissionDefinition": {
    "requiredEvidenceTypes": [
      "photo",
      "short_answer"
    ],
    "minimumEvidenceCount": 2,
    "questions": [
      {
        "questionId": "q-guess",
        "type": "short_answer",
        "prompt": "What exposure time did you predict and why?",
        "required": true
      },
      {
        "questionId": "q-result",
        "type": "short_answer",
        "prompt": "Which exposure stage produced the most useful result?",
        "required": true
      }
    ]
  },
  "validationDefinition": {
    "mode": "hybrid",
    "rules": [
      {
        "ruleId": "photo-present",
        "type": "evidence_presence",
        "weight": 20,
        "required": true
      },
      {
        "ruleId": "stage-comparison",
        "type": "llm_rubric",
        "weight": 50,
        "required": true
      },
      {
        "ruleId": "reflection-specificity",
        "type": "llm_rubric",
        "weight": 30,
        "required": true
      }
    ],
    "minimumScore": 70
  },
  "completionMessage": "You tested assumptions instead of relying on the first result.",
  "imagePrompt": "Educational split image showing a cyanotype test strip exposed in 5, 10, and 15 minute stages..."
}
```

---

# 11. Step Types

Allowed step types:

```text
preparation
learning
practical
observation
reflection
quiz
feedback
collaboration
presentation
final_deliverable
safety_check
```

A project may combine types.

The `type` must affect submission and validation behavior.

Examples:

### Preparation

Evidence:

- checklist;
- setup photo;
- safety acknowledgement.

### Practical

Evidence:

- photo;
- video;
- file;
- structured answers;
- measurable result.

### Reflection

Evidence:

- text answers;
- comparison;
- lessons learned.

### Quiz

Evidence:

- selected answers;
- score.

### Collaboration

Evidence:

- role description;
- feedback;
- participant confirmation where appropriate.

---

# 12. Evidence Model

## 12.1 Allowed Evidence Types

```text
checklist
short_answer
long_answer
quiz_answer
photo
video
audio
file
link
numeric_measurement
before_after
peer_feedback
self_confirmation
system_event
```

---

## 12.2 Evidence Record

```json
{
  "evidenceId": "evidence-123",
  "projectRunId": "run-123",
  "stepId": "step-03",
  "type": "photo",
  "storageLocation": "s3://...",
  "textValue": null,
  "numericValue": null,
  "metadata": {
    "mimeType": "image/jpeg",
    "sizeBytes": 234567
  },
  "submittedAt": "2026-07-25T13:00:00.000Z"
}
```

---

## 12.3 Evidence Privacy

The user must be informed:

- what evidence is required;
- why it is required;
- whether it is stored;
- whether it is used for AI evaluation;
- whether it can be deleted;
- whether it may appear in the final badge.

Do not require public publication.

Do not expose private evidence in badge metadata by default.

---

# 13. Validation Strategy

Validation must be layered.

## 13.1 Layer 1 — Structural Validation

Deterministic checks:

- required fields present;
- required evidence count reached;
- mandatory questions answered;
- file type allowed;
- file size allowed;
- checklist complete;
- numeric range valid;
- dependency steps complete.

---

## 13.2 Layer 2 — Knowledge Validation

May use:

- quiz;
- multiple-choice questions;
- ordering tasks;
- matching;
- short-answer grading.

This verifies understanding.

It does not prove practical completion alone.

---

## 13.3 Layer 3 — Practical Evidence Validation

May use:

- image presence;
- file metadata;
- before-and-after evidence;
- final artifact;
- structured observations;
- measurable output;
- process evidence.

For the MVP, visual evidence evaluation may be limited to:

- verifying that a file exists;
- asking the user to describe the evidence;
- optionally using a multimodal model if supported and approved.

The system must not claim certainty when evidence cannot be reliably interpreted.

---

## 13.4 Layer 4 — Reflection Quality

The model may evaluate whether the answer includes:

- a specific action;
- an observed result;
- a decision;
- a challenge;
- an adaptation;
- a lesson learned;
- connection to the target behavior.

The model must grade against a rubric.

It must not grade based on writing style, grammar quality, or verbosity.

---

## 13.5 Layer 5 — Soft-Skill Behavioral Evidence

Soft-skill validation must focus on behaviors, not personality labels.

Example for communication:

```text
- organized a message;
- considered an audience;
- explained a choice;
- asked for feedback;
- compared intended and perceived meaning;
- adapted the message after feedback.
```

Example for problem solving:

```text
- identified a problem;
- proposed alternatives;
- tested at least one alternative;
- compared results;
- adjusted the plan;
- explained the decision.
```

---

# 14. Soft-Skill Rubric Model

Every project must define a project-specific rubric.

Example:

```json
{
  "softSkillId": "communication",
  "rubricVersion": "1.0",
  "dimensions": [
    {
      "dimensionId": "message-organization",
      "label": "Message organization",
      "weight": 25,
      "levels": [
        {
          "score": 0,
          "description": "No clear message or sequence was provided."
        },
        {
          "score": 1,
          "description": "A message exists but is difficult to follow."
        },
        {
          "score": 2,
          "description": "The message has a clear beginning, middle, and end."
        },
        {
          "score": 3,
          "description": "The message is clear and intentionally adapted to the audience."
        }
      ]
    },
    {
      "dimensionId": "choice-explanation",
      "label": "Explanation of choices",
      "weight": 25,
      "levels": []
    },
    {
      "dimensionId": "feedback-use",
      "label": "Use of feedback",
      "weight": 25,
      "levels": []
    },
    {
      "dimensionId": "adaptation",
      "label": "Adaptation after feedback",
      "weight": 25,
      "levels": []
    }
  ]
}
```

---

# 15. Final Validation Score

Recommended project-level score:

```text
Project execution: 35%
Evidence completeness: 20%
Reflection quality: 20%
Soft-skill behavioral evidence: 25%
```

Example formula:

```text
finalScore =
  executionScore * 0.35 +
  evidenceScore * 0.20 +
  reflectionScore * 0.20 +
  softSkillScore * 0.25
```

---

## 15.1 Minimum Badge Requirements

Recommended default:

```text
All mandatory steps completed
All blocking checkpoints passed
Final deliverable evidence submitted
FinalScore >= 70
SoftSkillScore >= 60
EvidenceCompletenessScore >= 70
No unresolved safety violation
User explicitly confirms completion
```

These thresholds must be configurable.

---

## 15.2 Why the Soft-Skill Minimum Is Separate

A user could complete the physical artifact but skip the behaviors related to the target soft skill.

Example:

- user creates the artwork;
- user does not explain choices;
- user does not collect feedback;
- user does not reflect on communication.

The project may be practically complete but not yet eligible for the soft-skill practice badge.

The system should explain this clearly and allow the user to complete a corrective activity.

---

# 16. Exact Badge Unlock Trigger

The badge unlock trigger is:

```text
PROJECT_BADGE_ELIGIBILITY_CONFIRMED
```

This domain event may only be emitted when the deterministic function:

```text
evaluateBadgeEligibility(projectRunId)
```

returns:

```json
{
  "eligible": true
}
```

---

## 16.1 Badge Eligibility Pseudocode

```javascript
function evaluateBadgeEligibility({
  projectRun,
  steps,
  checkpoints,
  finalAssessment,
  safetyStatus,
  userCompletionConfirmation
}) {
  const mandatoryStepsCompleted = steps
    .filter((step) => step.mandatory)
    .every((step) => step.status === "completed");

  const mandatoryCheckpointsPassed = checkpoints
    .filter((checkpoint) => checkpoint.mandatory)
    .every((checkpoint) => checkpoint.status === "passed");

  const finalAssessmentPassed =
    finalAssessment.status === "passed" &&
    finalAssessment.finalScore >= finalAssessment.minimumScore &&
    finalAssessment.softSkillScore >= finalAssessment.minimumSoftSkillScore;

  const safetyPassed = safetyStatus !== "blocked";

  const finalEvidencePresent =
    finalAssessment.finalDeliverableEvidenceCount >= 1;

  const confirmed = userCompletionConfirmation === true;

  return {
    eligible:
      mandatoryStepsCompleted &&
      mandatoryCheckpointsPassed &&
      finalAssessmentPassed &&
      safetyPassed &&
      finalEvidencePresent &&
      confirmed
  };
}
```

---

## 16.2 Badge Issuance Rule

The badge service may issue a badge only after receiving:

```text
PROJECT_BADGE_ELIGIBILITY_CONFIRMED
```

It must not independently recalculate project content.

It should still enforce idempotency.

---

# 17. Retry and Remediation Flow

A failed validation must not punish or shame the user.

## 17.1 Step Failure

When a step does not pass:

```text
submitted → needs_revision
```

The system must return:

- what was accepted;
- what is missing;
- why it matters;
- one concrete next action;
- whether previous evidence remains saved;
- how to resubmit.

---

## 17.2 Final Assessment Failure

When final validation fails:

```text
awaiting_final_validation → validation_failed
```

The response must include remediation activities.

Example:

```json
{
  "status": "validation_failed",
  "finalScore": 66,
  "minimumScore": 70,
  "softSkillScore": 52,
  "minimumSoftSkillScore": 60,
  "strengths": [
    "All practical steps were completed.",
    "The final artifact was submitted."
  ],
  "missingEvidence": [
    "No explanation of how feedback changed the final result."
  ],
  "remediationTasks": [
    {
      "taskId": "remediation-01",
      "title": "Collect and apply one piece of feedback",
      "instructions": [
        "Show the final result to one person.",
        "Ask what message they understood.",
        "Write what you changed or why you kept the original version."
      ]
    }
  ]
}
```

After remediation, the user may resubmit.

---

# 18. Quiz Rules

A quiz may be used when it adds value.

It must not be included merely to create the appearance of validation.

## 18.1 Appropriate Uses

- safety;
- basic terminology;
- sequence understanding;
- material handling;
- project-specific decisions;
- recognition of common mistakes.

## 18.2 Inappropriate Uses

- claiming that a person has communication skills;
- replacing practical evidence;
- testing unrelated trivia;
- blocking the user with ambiguous questions;
- using language difficulty as a proxy for skill.

## 18.3 Quiz Configuration

```json
{
  "quizId": "quiz-final-01",
  "passingPercentage": 70,
  "maximumAttempts": 3,
  "shuffleQuestions": true,
  "showCorrectAnswerAfterAttempt": true,
  "questions": []
}
```

For the MVP, do not lock the entire project permanently after failed attempts.

Provide an explanation and allow retry.

---

# 19. Mentor-Style Interaction

The design reference uses conversational coaching during the step.

The AI may:

- ask the user to predict an outcome;
- ask why they chose an approach;
- react to observations;
- compare expected and actual results;
- suggest a correction;
- encourage reflection;
- explain a concept in context.

The AI must not:

- mark a step complete without submission;
- unlock a step directly;
- issue a badge;
- invent evidence;
- pretend to have viewed a file it could not access;
- state that a soft skill was developed with certainty.

---

# 20. Canonical Project Run Model

```json
{
  "schemaVersion": "1.0",
  "projectRunId": "run-123",
  "projectDefinitionId": "projdef-123",
  "userId": "user-123",
  "sessionId": "session-123",
  "status": "in_progress",
  "currentStepId": "step-03",
  "completedMandatorySteps": 2,
  "totalMandatorySteps": 8,
  "progressPercentage": 25,
  "startedAt": "2026-07-25T12:00:00.000Z",
  "completedAt": null,
  "badgeEligibility": {
    "eligible": false,
    "evaluatedAt": null,
    "reasons": []
  },
  "createdAt": "2026-07-25T11:55:00.000Z",
  "updatedAt": "2026-07-25T13:00:00.000Z",
  "version": 4
}
```

---

# 21. Step Progress Model

```json
{
  "projectRunId": "run-123",
  "stepId": "step-03",
  "status": "active",
  "attemptCount": 1,
  "latestAttemptId": "attempt-123",
  "score": null,
  "validationStatus": "not_submitted",
  "startedAt": "2026-07-25T13:00:00.000Z",
  "completedAt": null,
  "updatedAt": "2026-07-25T13:00:00.000Z"
}
```

---

# 22. Step Attempt Model

```json
{
  "attemptId": "attempt-123",
  "projectRunId": "run-123",
  "stepId": "step-03",
  "attemptNumber": 1,
  "answers": [
    {
      "questionId": "q-guess",
      "value": "I predicted 10 minutes because..."
    }
  ],
  "evidenceIds": [
    "evidence-123"
  ],
  "validationResult": {
    "status": "passed",
    "score": 82,
    "ruleResults": [],
    "feedback": "..."
  },
  "submittedAt": "2026-07-25T13:30:00.000Z"
}
```

---

# 23. Final Assessment Model

```json
{
  "assessmentId": "assessment-123",
  "projectRunId": "run-123",
  "status": "passed",
  "executionScore": 90,
  "evidenceScore": 85,
  "reflectionScore": 78,
  "softSkillScore": 72,
  "finalScore": 82.65,
  "minimumScore": 70,
  "minimumSoftSkillScore": 60,
  "rubricResults": [],
  "strengths": [],
  "improvementAreas": [],
  "remediationTasks": [],
  "evaluatedAt": "2026-07-27T18:00:00.000Z",
  "evaluationVersion": "1.0"
}
```

---

# 24. Badge Model

```json
{
  "badgeId": "badge-123",
  "projectRunId": "run-123",
  "userId": "user-123",
  "badgeDefinitionId": "badge-def-123",
  "title": "Critical Thinking in Cyanotype Practice",
  "description": "Completed a guided cyanotype project and demonstrated practice of observation, hypothesis testing, comparison, and adjustment.",
  "hobby": "Cyanotype Printing",
  "targetSoftSkill": "Critical Thinking",
  "difficulty": "Apprentice",
  "evidenceSummary": [
    "Completed all mandatory project steps",
    "Submitted final visual index",
    "Compared exposure-time hypotheses",
    "Documented one process adjustment"
  ],
  "issuedAt": "2026-07-27T18:05:00.000Z",
  "verificationCode": "..."
}
```

Do not include private file URLs.

---

# 25. Recommended DynamoDB Design

For hackathon simplicity and separation of responsibility, use dedicated tables.

## 25.1 `hobby-project-definitions`

Partition key:

```text
projectDefinitionId
```

Stores generated project plans.

---

## 25.2 `hobby-project-runs`

Partition key:

```text
projectRunId
```

Recommended GSI:

```text
userId-createdAt-index
```

Stores project-level execution state.

---

## 25.3 `hobby-step-progress`

Partition key:

```text
projectRunId
```

Sort key:

```text
stepId
```

Stores step status.

---

## 25.4 `hobby-step-attempts`

Partition key:

```text
projectRunId
```

Sort key:

```text
stepAttemptKey
```

Where:

```text
stepAttemptKey = STEP#{stepId}#ATTEMPT#{attemptNumber}
```

---

## 25.5 `hobby-badges`

Partition key:

```text
badgeId
```

Recommended GSI:

```text
projectRunId-index
```

---

## 25.6 Simplified MVP Alternative

If the team must reduce infrastructure:

- `hobby-project-definitions`;
- `hobby-project-runs`;
- store step progress and attempts as nested objects inside the run.

Use the simplified model only when:

- project step counts are small;
- evidence metadata is limited;
- concurrent updates are unlikely;
- DynamoDB item size remains safely below the limit.

Recommended production-oriented design: separate progress and attempts.

Recommended hackathon design: definitions + runs + badges, with compact nested progress.

---

# 26. S3 Storage

Use S3 for:

- user evidence files;
- generated images;
- optional project exports;
- badge images.

Recommended prefixes:

```text
evidence/{userId}/{projectRunId}/{stepId}/{evidenceId}
project-images/{projectDefinitionId}/{imageId}
exports/{userId}/{projectRunId}/{exportId}.pdf
badges/{userId}/{badgeId}.png
```

Use signed URLs for private access.

Do not store base64 image data in DynamoDB.

---

# 27. Required Environment Variables

```text
PROJECT_DEFINITIONS_TABLE
PROJECT_RUNS_TABLE
STEP_PROGRESS_TABLE
STEP_ATTEMPTS_TABLE
BADGES_TABLE
PROJECT_EVIDENCE_BUCKET
PROJECT_ASSETS_BUCKET
BEDROCK_TEXT_MODEL_ID
BEDROCK_VISION_MODEL_ID
BADGE_MINIMUM_FINAL_SCORE
BADGE_MINIMUM_SOFT_SKILL_SCORE
```

Only include tables actually used by the chosen MVP design.

---

# 28. Required Domain Services

## 28.1 `ProjectDefinitionService`

Responsibilities:

- validate project definition;
- save project definition;
- retrieve project definition;
- version project definition.

---

## 28.2 `ProjectRunService`

Responsibilities:

- create run;
- start run;
- get current state;
- calculate progress;
- transition status;
- enforce optimistic concurrency.

---

## 28.3 `StepProgressionService`

Responsibilities:

- determine available steps;
- activate a step;
- submit an attempt;
- mark completion;
- unlock dependent steps.

---

## 28.4 `ValidationService`

Responsibilities:

- structural validation;
- quiz grading;
- rubric preparation;
- AI-assisted evaluation;
- score calculation;
- remediation generation.

---

## 28.5 `BadgeEligibilityService`

Responsibilities:

- evaluate deterministic eligibility;
- emit eligibility event;
- never use free-form AI judgment as the only decision.

---

## 28.6 `BadgeService`

Responsibilities:

- issue badge idempotently;
- create metadata;
- generate verification code;
- optionally generate badge image.

---

# 29. Required Agent Tools

The names below are recommended contracts.

## 29.1 `create_guided_hobby_project`

Creates the project definition.

It must not start execution automatically unless explicitly requested.

---

## 29.2 `start_hobby_project`

Creates or activates a project run.

Input:

```json
{
  "projectDefinitionId": "projdef-123"
}
```

---

## 29.3 `get_project_progress`

Returns:

- project status;
- progress percentage;
- current step;
- completed steps;
- blocked steps;
- badge eligibility state.

---

## 29.4 `get_current_project_step`

Returns only the step currently available or active.

This prevents unnecessarily loading the entire project into the model for every interaction.

---

## 29.5 `submit_project_step`

Accepts:

- answers;
- checklist values;
- evidence references;
- user confirmation.

Creates a step attempt.

---

## 29.6 `validate_project_step`

Runs deterministic and AI-assisted validation.

For the MVP, it may be called internally by `submit_project_step`.

---

## 29.7 `submit_final_project_assessment`

Receives final reflection, quiz answers, and final evidence.

---

## 29.8 `evaluate_badge_eligibility`

Deterministically evaluates rules.

The tool must not accept `eligible` as model-provided input.

---

## 29.9 `issue_project_badge`

Issues a badge only when eligibility is already true.

---

## 29.10 `export_project`

Optional.

Generates a project summary or completed-project report as PDF.

The PDF is now a secondary artifact, not the primary interaction.

---

# 30. Tool Security Rule

The LLM may request an operation.

The domain layer decides whether the operation is valid.

Example:

```text
LLM requests: issue_project_badge
        ↓
BadgeService checks persisted eligibility
        ↓
If false: reject
If true: issue idempotently
```

Never trust model-provided state.

---

# 31. Session and User Identity

Capture `sessionId` and authenticated `userId` outside model-controlled inputs.

Recommended closure pattern:

```javascript
export async function* answerWith(message, context) {
  const { sessionId, userId } = context;

  const submitProjectStep = tool({
    name: "submit_project_step",
    inputSchema: submitProjectStepSchema,
    callback: async (input) => {
      return stepProgressionService.submit({
        ...input,
        sessionId,
        userId
      });
    }
  });
}
```

The model must not generate:

- user ID;
- session ID;
- project ownership;
- badge eligibility;
- current step status.

---

# 32. API Contracts

If the frontend communicates directly with dedicated endpoints, use versioned routes.

Recommended endpoints:

```text
POST   /v1/project-definitions
POST   /v1/project-runs
GET    /v1/project-runs/{projectRunId}
GET    /v1/project-runs/{projectRunId}/current-step
POST   /v1/project-runs/{projectRunId}/steps/{stepId}/attempts
POST   /v1/project-runs/{projectRunId}/final-assessment
POST   /v1/project-runs/{projectRunId}/badge-eligibility
POST   /v1/project-runs/{projectRunId}/badge
GET    /v1/badges/{badgeId}
```

For the hackathon, these operations may remain behind agent tools, but the service contracts should still be separated.

---

# 33. Domain Events

Recommended events:

```text
PROJECT_DEFINITION_CREATED
PROJECT_RUN_STARTED
PROJECT_STEP_ACTIVATED
PROJECT_STEP_SUBMITTED
PROJECT_STEP_VALIDATION_PASSED
PROJECT_STEP_VALIDATION_FAILED
PROJECT_STEP_COMPLETED
PROJECT_FINAL_ASSESSMENT_SUBMITTED
PROJECT_FINAL_ASSESSMENT_PASSED
PROJECT_FINAL_ASSESSMENT_FAILED
PROJECT_COMPLETED
PROJECT_BADGE_ELIGIBILITY_CONFIRMED
PROJECT_BADGE_ISSUED
```

MVP implementation may log these events rather than using EventBridge.

The names must still be consistent across modules.

---

# 34. Recommended AWS Stack

## Core MVP

- AWS Lambda;
- Amazon API Gateway;
- Amazon Bedrock;
- Strands Agents SDK;
- Amazon DynamoDB;
- Amazon S3;
- AWS SAM;
- CloudWatch Logs;
- Node.js;
- Zod;
- AWS SDK v3.

## Optional After MVP

- Amazon EventBridge for domain events;
- Amazon SQS for asynchronous validation;
- AWS Step Functions for long-running image/PDF flows;
- Amazon Bedrock multimodal model for evidence assistance;
- Amazon CloudFront for controlled asset delivery;
- Amazon Cognito for authentication.

Do not introduce optional services during the MVP unless the team has agreed.

---

# 35. Recommended Source Structure

```text
src/
├── agent.mjs
├── index.mjs
├── chat-page.mjs
│
├── domain/
│   ├── project-status.mjs
│   ├── step-status.mjs
│   ├── badge-rules.mjs
│   ├── score-calculator.mjs
│   └── errors.mjs
│
├── schemas/
│   ├── project-definition.schema.mjs
│   ├── project-run.schema.mjs
│   ├── step.schema.mjs
│   ├── evidence.schema.mjs
│   ├── assessment.schema.mjs
│   └── badge.schema.mjs
│
├── services/
│   ├── project-definition.service.mjs
│   ├── project-run.service.mjs
│   ├── step-progression.service.mjs
│   ├── validation.service.mjs
│   ├── badge-eligibility.service.mjs
│   └── badge.service.mjs
│
├── repositories/
│   ├── project-definition.repository.mjs
│   ├── project-run.repository.mjs
│   ├── step-attempt.repository.mjs
│   └── badge.repository.mjs
│
├── tools/
│   ├── create-guided-hobby-project.tool.mjs
│   ├── start-hobby-project.tool.mjs
│   ├── get-project-progress.tool.mjs
│   ├── get-current-project-step.tool.mjs
│   ├── submit-project-step.tool.mjs
│   ├── submit-final-assessment.tool.mjs
│   ├── evaluate-badge-eligibility.tool.mjs
│   └── issue-project-badge.tool.mjs
│
├── prompts/
│   ├── project-generation.prompt.mjs
│   ├── step-validation.prompt.mjs
│   └── final-assessment.prompt.mjs
│
└── utils/
    ├── ids.mjs
    ├── dates.mjs
    ├── logger.mjs
    └── score.mjs
```

If the existing codebase is very small, Claude Code should migrate incrementally.

Do not create all files before verifying the existing structure.

---

# 36. AI Prompt Separation

Do not use one giant prompt for every responsibility.

Recommended prompts:

## Project Generation Prompt

Creates:

- title;
- steps;
- materials;
- instructions;
- validation definitions;
- final assessment;
- badge metadata.

## Step Validation Prompt

Receives only:

- step definition;
- rubric;
- user answers;
- non-sensitive evidence descriptions;
- deterministic validation results.

Returns a structured rubric result.

## Final Assessment Prompt

Receives:

- project summary;
- completed step results;
- final reflection;
- final evidence description;
- soft-skill rubric.

Returns scores and feedback.

The deterministic badge service consumes the assessment result.

---

# 37. AI Validation Output Contract

```json
{
  "schemaVersion": "1.0",
  "status": "passed",
  "score": 82,
  "criteria": [
    {
      "criterionId": "comparison-quality",
      "score": 2,
      "maximumScore": 3,
      "evidence": "The user compared 5, 10, and 15 minute results.",
      "feedback": "The comparison is specific and tied to the observed tones."
    }
  ],
  "strengths": [
    "Specific observation"
  ],
  "missingRequirements": [],
  "feedbackToUser": "Your comparison shows that you tested alternatives rather than accepting the first result.",
  "confidence": "medium"
}
```

Do not allow prose-only validation output.

---

# 38. AI Evaluation Rules

The evaluator must:

- use only provided evidence;
- state when evidence is insufficient;
- grade against the supplied rubric;
- ignore grammar, spelling, and answer length;
- avoid personality diagnosis;
- avoid protected-attribute inference;
- avoid claiming mastery;
- not award the badge;
- not change project state;
- return structured JSON.

---

# 39. Confidence Handling

Allowed values:

```text
low
medium
high
```

Low confidence should trigger:

- manual confirmation;
- additional question;
- request for clearer evidence;
- fallback validation.

It should not silently fail or silently pass.

---

# 40. Progress Calculation

Recommended formula:

```text
progress =
  completed mandatory step weight /
  total mandatory step weight
```

Each step may define a weight.

Example:

```json
{
  "stepId": "step-03",
  "progressWeight": 15
}
```

If no weights exist, use equal weight.

Do not calculate progress only by step index when steps have very different effort.

---

# 41. Badge Design Metadata

The project definition should include:

```json
{
  "badgeDefinition": {
    "badgeDefinitionId": "badge-def-123",
    "title": "Critical Thinking Explorer",
    "subtitle": "Cyanotype Practice",
    "description": "Completed a guided cyanotype project...",
    "iconPrompt": "Minimal badge illustration combining a sun, leaf, and layered test strip...",
    "criteriaSummary": [
      "Completed all mandatory steps",
      "Submitted final deliverable",
      "Passed final project assessment",
      "Demonstrated hypothesis comparison and adaptation"
    ]
  }
}
```

The badge image may be generated later.

---

# 42. Project Export

The complete project may still be exported to PDF.

However, the PDF should reflect the interactive experience.

Possible exports:

## Project Plan Export

Contains:

- roadmap;
- steps;
- materials;
- schedule;
- blank checklists.

## Progress Export

Contains:

- completed steps;
- dates;
- feedback;
- current status.

## Completion Report

Contains:

- project summary;
- final deliverable;
- evidence summary;
- reflection;
- assessment scores;
- badge information.

The PDF must not become the source of truth.

The database is the source of truth.

---

# 43. Concurrency and Idempotency

## 43.1 Optimistic Concurrency

Project runs should include:

```text
version
```

Update with a conditional expression:

```text
version = expectedVersion
```

This prevents double submissions and conflicting progress updates.

## 43.2 Idempotency

Required for:

- step submission;
- final assessment submission;
- badge issuance.

Use:

```text
idempotencyKey
```

A repeated request with the same key should return the original result.

---

# 44. Error Codes

```text
PROJECT_DEFINITION_NOT_FOUND
PROJECT_RUN_NOT_FOUND
PROJECT_NOT_READY
PROJECT_ALREADY_STARTED
INVALID_PROJECT_TRANSITION
STEP_NOT_FOUND
STEP_LOCKED
STEP_ALREADY_COMPLETED
STEP_DEPENDENCY_INCOMPLETE
STEP_SUBMISSION_INVALID
STEP_VALIDATION_FAILED
EVIDENCE_REQUIRED
EVIDENCE_TYPE_NOT_ALLOWED
EVIDENCE_UPLOAD_FAILED
FINAL_ASSESSMENT_NOT_AVAILABLE
FINAL_ASSESSMENT_FAILED
BADGE_NOT_ELIGIBLE
BADGE_ALREADY_ISSUED
UNAUTHORIZED_PROJECT_ACCESS
CONCURRENT_UPDATE_CONFLICT
AI_VALIDATION_UNAVAILABLE
AI_VALIDATION_LOW_CONFIDENCE
PERSISTENCE_ERROR
```

---

# 45. Error Response Contract

```json
{
  "success": false,
  "error": {
    "code": "STEP_LOCKED",
    "message": "This step is not available yet.",
    "details": {
      "requiredCompletedSteps": ["step-02"]
    },
    "recoverable": true,
    "suggestedAction": "Complete the previous step first."
  }
}
```

---

# 46. Security and Privacy

## 46.1 Authorization

Every read or write must verify project ownership.

Do not rely only on a project ID.

## 46.2 File Validation

For uploads:

- allowlist MIME types;
- enforce size limits;
- generate server-side file names;
- avoid executable content;
- use private S3 buckets;
- use signed URLs;
- consider malware scanning after MVP.

## 46.3 Data Minimization

Store only evidence required for project validation.

## 46.4 AI Data Use

Do not send unnecessary user profile fields to the model.

## 46.5 IAM

Use least privilege for:

- DynamoDB table actions;
- S3 prefixes;
- Bedrock model invocation;
- CloudWatch logs.

---

# 47. Observability

Log structured events.

Example:

```json
{
  "event": "PROJECT_STEP_VALIDATION_COMPLETED",
  "projectRunId": "run-123",
  "stepId": "step-03",
  "attemptNumber": 1,
  "status": "passed",
  "score": 82,
  "durationMs": 1420
}
```

Do not log full private evidence.

Recommended metrics:

- project starts;
- project completion rate;
- average step attempts;
- validation failure rate;
- badge eligibility rate;
- badge issuance rate;
- average project duration;
- AI validation error rate;
- evidence upload failure rate.

---

# 48. Accessibility

The frontend and content must support:

- keyboard navigation;
- clear focus state;
- readable contrast;
- status communicated by text and not color alone;
- alt text for images;
- transcript or text alternative for audio/video guidance;
- reduced-motion preference;
- clear error messages;
- simple-language mode where available;
- no time-limited answer unless essential.

---

# 49. Safety Rules

Safety validation may block progression.

Example:

```text
Safety step required before using heat, chemicals, blades, electricity, or power tools.
```

The project definition must identify:

- hazard;
- required precaution;
- required acknowledgement;
- safer alternative;
- whether adult supervision is required.

The badge must not be issued if a mandatory safety step is unresolved.

---

# 50. Business Rules

## BR-GPE-001

A project run must reference a valid project definition.

## BR-GPE-002

A user may start only a project they are authorized to access.

## BR-GPE-003

The backend owns project and step state.

## BR-GPE-004

A mandatory step cannot be skipped.

## BR-GPE-005

A locked step cannot receive a valid submission.

## BR-GPE-006

A step is completed only after its configured validation passes.

## BR-GPE-007

A quiz cannot be the only evidence for a practical project.

## BR-GPE-008

Soft-skill assessment must use behavioral criteria.

## BR-GPE-009

The system must not claim mastery or certification.

## BR-GPE-010

All mandatory steps must be completed before final validation.

## BR-GPE-011

Final deliverable evidence is mandatory.

## BR-GPE-012

Badge eligibility is deterministic.

## BR-GPE-013

The AI cannot directly issue a badge.

## BR-GPE-014

Badge issuance must be idempotent.

## BR-GPE-015

Failed validation must produce actionable remediation.

## BR-GPE-016

Private evidence must not be exposed in public badge metadata.

## BR-GPE-017

PDF generation must not modify project progress.

## BR-GPE-018

Image-generation failure must not block project execution.

## BR-GPE-019

Project progress must survive a new chat session.

## BR-GPE-020

Conversation memory must not be used as the only progress database.

---

# 51. MVP Scope

## Required

- create project definition;
- start project run;
- show current step;
- submit text/checklist evidence;
- optionally upload one image per practical step;
- validate step;
- unlock next step;
- calculate progress;
- submit final reflection;
- calculate final assessment;
- evaluate badge eligibility;
- issue badge metadata;
- persist all state.

## Optional

- multimodal image evaluation;
- video/audio evidence;
- automatic badge image generation;
- PDF completion report;
- EventBridge;
- Step Functions;
- peer validation;
- public badge verification page.

---

# 52. Acceptance Criteria

The feature is accepted when:

1. a selected hobby can become a valid project definition;
2. the project contains ordered steps;
3. each step includes instructions and validation rules;
4. a project run can be started;
5. only the first valid step is initially available;
6. completing a step unlocks its dependency successors;
7. invalid submissions do not unlock the next step;
8. progress persists after refresh or new conversation;
9. all mandatory steps are required;
10. final validation is unavailable before mandatory steps are complete;
11. final assessment returns structured scores;
12. badge eligibility follows deterministic rules;
13. the model cannot directly issue a badge;
14. repeated badge issuance does not create duplicates;
15. the badge wording states practice/completion, not mastery;
16. failure produces a remediation path;
17. unauthorized access is rejected;
18. current memory and streaming behavior remain functional;
19. UI status matches backend status;
20. no duplicate progress logic exists in multiple features.

---

# 53. Testing Strategy

## 53.1 Unit Tests

Test:

- state transitions;
- dependency unlocking;
- progress calculation;
- weighted scores;
- badge eligibility;
- idempotency;
- ownership;
- validation thresholds;
- mandatory-step logic.

---

## 53.2 Integration Tests

Test:

- DynamoDB persistence;
- step submission;
- final assessment;
- badge issuance;
- S3 evidence references;
- tool registration;
- agent-to-service calls.

---

## 53.3 Contract Tests

Validate that upstream feature outputs match:

- profile contract;
- selected hobby contract;
- project definition contract.

---

## 53.4 End-to-End Test

Scenario:

```text
Create project
Start project
Complete Step 1
Fail Step 2
Retry Step 2
Complete remaining steps
Submit final assessment
Fail soft-skill threshold
Complete remediation
Pass final assessment
Unlock badge
Issue badge
Retrieve badge
```

---

# 54. Example End-to-End Project

## Project

```text
Street Flora Index
```

## Hobby

```text
Cyanotype Printing
```

## Soft Skill

```text
Critical Thinking
```

## Step Sequence

### Step 1 — Mix the sensitizer

Validation:

- safety checklist;
- material checklist;
- one setup photo.

### Step 2 — Coat and dry the paper

Validation:

- process confirmation;
- drying-condition answer;
- coated paper evidence.

### Step 3 — Expose in direct sun

Validation:

- predicted exposure time;
- staged test evidence;
- comparison answer.

### Step 4 — Wash and observe

Validation:

- photo after washing;
- observation of color change;
- explanation of unexpected result.

### Step 5 — Collect local plant forms

Validation:

- list of selected plants;
- reason for selection;
- ethical collection acknowledgement.

### Step 6 — Create twelve prints

Validation:

- evidence of final collection;
- quality checklist;
- one process adaptation.

### Step 7 — Organize the index

Validation:

- ordered collection;
- naming system;
- explanation of organization logic.

### Step 8 — Final reflection

Validation:

- hypothesis;
- test;
- result;
- adaptation;
- connection to critical-thinking behaviors.

---

# 55. Claude Code Implementation Rules

Claude Code must:

- inspect before editing;
- preserve existing memory;
- preserve streaming;
- preserve current Bedrock setup;
- preserve existing tools unless migration is approved;
- make small reviewed changes;
- avoid unrelated refactors;
- use Zod schemas;
- separate domain logic from tools;
- use repository modules for persistence;
- centralize status constants;
- centralize badge rules;
- centralize error codes;
- use conditional updates;
- avoid model-controlled IDs;
- avoid duplicating business rules in prompts and UI;
- show diffs;
- not run deployment commands without explicit approval.

---

# 56. Claude Code Prompt — Analyze the Existing Project

```text
Analyze the existing AWS SAM and Strands Agents project without modifying files.

This task concerns only the Guided Hobby Project Execution, Validation, and Badge Unlocking feature.

Inspect:
- template.yaml;
- samconfig.toml;
- src/agent.mjs;
- src/index.mjs;
- src/chat-page.mjs;
- src/package.json;
- all existing tool implementations;
- all existing DynamoDB resources;
- all existing system prompts.

Explain:

1. the current request flow;
2. how userId and sessionId are obtained;
3. how conversation memory is persisted;
4. how tools are registered;
5. where project-definition state should live;
6. where execution state should live;
7. how to preserve current features;
8. which responsibilities already exist in other modules;
9. which proposed modules can be added without duplication;
10. the smallest safe implementation sequence.

Identify any conflict between this specification and the current code.

Do not edit files.
Do not install packages.
Do not run build, deploy, sync, or tests.
```

---

# 57. Claude Code Prompt — Create Shared Domain Constants

```text
Implement shared domain constants and transition validation for guided project execution.

Requirements:

1. Create centralized constants for:
   - project run statuses;
   - step statuses;
   - validation statuses;
   - evidence types;
   - domain event names;
   - error codes.

2. Implement pure functions:
   - canTransitionProjectStatus(current, next);
   - canTransitionStepStatus(current, next);
   - assertValidProjectTransition(current, next);
   - assertValidStepTransition(current, next).

3. No DynamoDB calls.
4. No Bedrock calls.
5. No UI changes.
6. Preserve existing behavior.
7. Add unit tests if a test framework already exists.
8. Do not introduce a new test framework without approval.

Show the diff before applying.
```

---

# 58. Claude Code Prompt — Create Schemas

```text
Implement Zod schemas for the guided project feature.

Create reusable schemas for:

- ProjectDefinition;
- ProjectStep;
- StepInstruction;
- SubmissionDefinition;
- ValidationDefinition;
- ValidationRule;
- ProjectRun;
- StepProgress;
- StepAttempt;
- Evidence;
- FinalAssessment;
- SoftSkillRubric;
- BadgeDefinition;
- BadgeRecord;
- DomainErrorResponse.

Requirements:

1. Include schemaVersion in persisted root objects.
2. Use enums from centralized domain constants.
3. Validate positive time values.
4. Validate non-negative cost values.
5. Validate that mandatory practical steps require practical evidence.
6. Validate that badge rules contain final-score and soft-skill thresholds.
7. Do not add persistence yet.
8. Preserve existing code behavior.
9. Show the diff.
```

---

# 59. Claude Code Prompt — Infrastructure

```text
Update AWS SAM infrastructure for the guided project MVP.

First inspect the current resources and avoid creating duplicate tables or buckets.

Preferred resources:

- ProjectDefinitionsTable;
- ProjectRunsTable;
- BadgesTable;
- ProjectEvidenceBucket.

For the first MVP, step progress and attempts may be stored inside ProjectRunsTable only if:
- item size is safe;
- update patterns remain manageable;
- the architecture decision is documented.

Requirements:

1. Use PAY_PER_REQUEST for DynamoDB.
2. Use encryption defaults.
3. Block public S3 access.
4. Add environment variables.
5. Add least-privilege IAM.
6. Preserve existing conversation-memory resources.
7. Do not modify unrelated API routes.
8. Do not execute SAM commands.
9. Show the complete diff.
```

---

# 60. Claude Code Prompt — Project Definition Service

```text
Implement ProjectDefinitionService.

Responsibilities:

- validate a generated ProjectDefinition;
- assign projectDefinitionId;
- set schemaVersion, timestamps, and version;
- save to the configured DynamoDB table;
- retrieve by ID;
- reject invalid definitions.

Important:

- The service must not start a project run.
- The service must not issue badges.
- The service must not depend on chat memory.
- The service must return structured domain errors.
- Use crypto.randomUUID().
- Do not allow the model to control ownership identifiers.
- Show the diff before applying.
```

---

# 61. Claude Code Prompt — Project Run and Step Progression

```text
Implement ProjectRunService and StepProgressionService.

ProjectRunService must:

- create a run from a valid project definition;
- start a run;
- retrieve the run;
- calculate persisted progress;
- enforce project state transitions;
- use optimistic concurrency.

StepProgressionService must:

- identify available steps;
- activate the current step;
- create step attempts;
- validate dependency completion;
- prevent locked-step submission;
- mark passed steps completed;
- unlock dependent steps;
- update project progress.

Do not call Bedrock directly from these services.
AI validation must be injected through ValidationService.

Do not duplicate state logic in tools.
Show the diff.
```

---

# 62. Claude Code Prompt — Validation Service

```text
Implement ValidationService with layered validation.

Required layers:

1. structural validation;
2. deterministic rule validation;
3. quiz grading when configured;
4. optional AI rubric evaluation;
5. score calculation;
6. structured feedback;
7. remediation suggestions.

Requirements:

- The AI evaluator receives only the required context.
- The evaluator must return structured JSON validated by Zod.
- Low-confidence results must not silently pass.
- Grammar and answer length must not affect the score.
- A quiz cannot be the only validator for a practical step.
- The service must not unlock steps itself.
- The service must not issue badges.
- Show the diff before applying.
```

---

# 63. Claude Code Prompt — Badge Eligibility and Issuance

```text
Implement BadgeEligibilityService and BadgeService.

BadgeEligibilityService must:

- load persisted project-run state;
- verify all mandatory steps;
- verify mandatory checkpoints;
- verify final deliverable evidence;
- verify final-assessment status;
- verify finalScore threshold;
- verify softSkillScore threshold;
- verify safety status;
- verify explicit user completion confirmation;
- return deterministic eligibility reasons.

BadgeService must:

- issue only when persisted eligibility is true;
- be idempotent;
- create badge metadata;
- avoid private evidence URLs;
- set project status to badge_issued only after successful creation.

The model must never provide eligible=true as a trusted value.

Show the diff.
```

---

# 64. Claude Code Prompt — Agent Tools

```text
Implement or register these agent tools using the existing Strands Agents conventions:

- create_guided_hobby_project;
- start_hobby_project;
- get_project_progress;
- get_current_project_step;
- submit_project_step;
- submit_final_project_assessment;
- evaluate_badge_eligibility;
- issue_project_badge.

Requirements:

1. Capture userId and sessionId outside model-controlled input.
2. Tools must call services, not contain duplicated domain logic.
3. Tools must return structured results.
4. Preserve beforeToolCallEvent behavior.
5. Preserve streaming.
6. Preserve conversation memory.
7. Do not expose internal IDs unnecessarily to the user-facing response.
8. Do not run deployment commands.
9. Show the diff.
```

---

# 65. Claude Code Prompt — System Prompt

```text
Update only the system-prompt sections required for guided project execution.

The agent must:

- recognize when the user is working on an active project;
- retrieve current project progress;
- present only the current available step unless the user asks for an overview;
- explain instructions clearly;
- use mentor-style questions;
- request required evidence;
- call submit_project_step;
- wait for backend validation;
- never claim that a step passed before the tool confirms it;
- never unlock a step through conversation alone;
- never issue a badge directly;
- never claim soft-skill mastery;
- explain failed validation constructively;
- offer the configured remediation task;
- respond in the user's language;
- preserve hobbies as recreational experiences.

Do not rewrite unrelated prompt behavior.
Show the diff.
```

---

# 66. Claude Code Prompt — Frontend Integration Contract

```text
Analyze the existing chat frontend and propose the smallest integration for the step-based design.

Required UI states:

- locked;
- available;
- active;
- submitted;
- needs_revision;
- completed.

Required project header data:

- title;
- difficulty;
- target soft skill;
- progress percentage;
- project status.

Required current-step data:

- title;
- objective;
- instructions;
- materials;
- estimated time;
- safety notes;
- mentor prompts;
- submission requirements;
- validation feedback.

Do not redesign the full interface.
Do not duplicate backend state rules.
The frontend must render statuses returned by the backend.
Do not edit files until the proposal is reviewed.
```

---

# 67. Recommended Delivery Sequence

```text
1. Analyze existing code and feature boundaries
2. Create shared domain constants
3. Create schemas
4. Add persistence resources
5. Implement project definition service
6. Implement project run service
7. Implement step progression
8. Implement deterministic validation
9. Add AI-assisted rubric validation
10. Implement final assessment
11. Implement badge eligibility
12. Implement badge issuance
13. Register agent tools
14. Update the system prompt
15. Integrate the step-based UI
16. Add evidence upload
17. Add export/PDF
18. Add image evaluation only if time remains
```

---

# 68. Final Definition of Done

The feature is complete when the user can:

1. open a personalized hobby project;
2. start it;
3. see a structured sequence of steps;
4. complete one step at a time;
5. submit required answers and evidence;
6. receive validation feedback;
7. retry when necessary;
8. unlock the next step only after passing;
9. finish all mandatory stages;
10. submit a final deliverable;
11. complete a soft-skill reflection;
12. receive a structured final assessment;
13. complete remediation if required;
14. unlock a badge only after deterministic rules pass;
15. retrieve the badge later.

The implementation is complete when:

- project state is persisted independently from chat memory;
- feature boundaries are respected;
- backend state is authoritative;
- business rules are centralized;
- AI responses are schema-validated;
- badge issuance is deterministic and idempotent;
- no feature duplicates another feature's behavior;
- the UI follows backend contracts;
- the badge communicates verified practice and project completion, not mastery.
