# Master Questions System

## Overview
A system where superadmins create master questions that can be assigned to multiple organizations. When organization admins modify these questions, it creates organization-specific copies without affecting the original master question.

## Key Features

### 1. Master Questions
- **Creation**: Only superadmins can create master questions (no `organizationId` required)
- **Assignment**: Can be assigned to multiple organizations
- **Protection**: Cannot be directly modified by organization admins
- **Deletion**: Only superadmins can delete master questions (also deletes all customized versions)

### 2. Customized Questions
- **Auto-Creation**: When an org admin updates a master question, a customized copy is created for their organization
- **Visibility**: Org admins only see their customized version (not the master)
- **Isolation**: Changes to customized questions don't affect the master or other organizations
- **Deletion**: Org admins can delete their customized version without affecting others

### 3. Organization-Specific Questions
- **Single Organization**: Regular questions are tied to one organization only
- **Standard Rules**: Follow existing creation, update, and deletion rules

## Database Schema Changes

### Question Model Fields
```javascript
{
  organizationId: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'Organization',
    required: false  // Now optional for master questions
  },
  isMasterQuestion: { 
    type: Boolean, 
    default: false,
    index: true 
  },
  masterQuestionId: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'Question',
    default: null,
    index: true 
  },
  assignedOrganizations: { 
    type: [mongoose.Schema.Types.ObjectId], 
    ref: 'Organization',
    default: [],
    index: true 
  }
}
```

### Model Methods
- `isMaster()` - Check if question is a master question
- `isCustomized()` - Check if question is a customized copy
- `isAssignedTo(organizationId)` - Check if master question is assigned to organization
- `assignToOrganization(organizationId)` - Assign master to organization
- `unassignFromOrganization(organizationId)` - Unassign master from organization

## API Endpoints

### Existing Endpoints (Modified Behavior)
- `GET /questions` - Returns customized versions instead of masters when they exist
- `GET /questions/:id` - Returns customized version if it exists for user's organization
- `POST /questions` - Superadmins can create master questions
- `PATCH /questions/:id` - Org admins trigger customization when updating masters
- `DELETE /questions/:id` - Deletion rules based on question type

### New Endpoints
- `GET /questions/master` - Get all master questions (superadmin only)
- `POST /questions/:id/assign` - Assign master question to organizations (superadmin only)
- `POST /questions/:id/unassign` - Unassign master question from organizations (superadmin only)
- `GET /questions/:id/versions` - Get all customized versions of a master (superadmin only)

## User Flows

### Superadmin Creates Master Question
```javascript
POST /questions
{
  "title": "How was your week?",
  "description": "Share your weekly highlights",
  "isMasterQuestion": true
  // No organizationId
}
```

### Superadmin Assigns to Organizations
```javascript
POST /questions/{questionId}/assign
{
  "organizationIds": ["org1", "org2", "org3"]
}
```

### Org Admin Views Questions
- Sees assigned master questions
- If they've customized a master, they see ONLY their version (not the original)

### Org Admin Modifies Master Question
```javascript
PATCH /questions/{masterQuestionId}
{
  "title": "Custom title for our organization",
  "organizationId": "org1"  // Pass their organization ID
}
```
**Result**: Creates a new customized question:
```javascript
{
  "title": "Custom title for our organization",
  "organizationId": "org1",
  "isMasterQuestion": false,
  "masterQuestionId": "{masterQuestionId}",
  // ... other fields copied from master
}
```

### Org Admin Deletes Customized Question
- Only deletes their organization's customized version
- Master question and other organizations' versions remain intact
- After deletion, they see the master question again

## Response and Reaction Behavior

### Get Question Details (`GET /questions/:id/details`)

**Response Structure**:
```javascript
{
  "question": { /* Question object */ },
  
  "myResponses": [
    // Only current user's responses
    {
      "id": "...",
      "response": "My answer",
      "userId": "...",
      "reactions": {
        "like": [
          { userId: "...", userName: "John Doe", ... },
          { userId: "...", userName: "Jane Smith", ... }
        ],
        "love": [
          { userId: "...", userName: "Bob Wilson", ... }
        ]
      }
    }
  ],
  
  "reactions": {
    // All users' reactions to the question (key-value pairs)
    "like": [
      { userId: "...", userName: "John Doe", userEmail: "...", ... },
      { userId: "...", userName: "Jane Smith", userEmail: "...", ... }
    ],
    "love": [
      { userId: "...", userName: "Bob Wilson", userEmail: "...", ... }
    ]
  },
  
  "stats": {
    "myResponsesCount": 2,
    "totalQuestionReactions": 5,
    "totalResponseReactions": 3
  }
}
```

### Key Points:
1. **Responses**: Only show current user's own responses
2. **Reactions**: Show ALL users' reactions as key-value pairs grouped by reaction type
3. **Response Reactions**: Each response includes reactions from all users

## Access Control

### Superadmin
- ✅ Create master questions
- ✅ Assign/unassign master questions to organizations
- ✅ View all questions (masters and customized)
- ✅ Update master questions directly
- ✅ Delete master questions (cascades to customized versions)
- ✅ View all customized versions of a master

### Organization Admin
- ✅ View assigned master questions (unless they have a customized version)
- ✅ View their customized versions (hides the master)
- ✅ Update master questions (creates customized copy)
- ✅ Update their customized questions
- ✅ Delete their customized questions (only affects their org)
- ❌ Cannot create master questions
- ❌ Cannot directly modify master questions
- ❌ Cannot delete master questions

### Regular Users
- ✅ View questions assigned to their organizations
- ✅ Respond to questions
- ✅ React to questions and responses
- ❌ Cannot create or modify questions

## Query Behavior

### For Organization Admins
```
Query Flow:
1. Get all organization-specific questions (including customized copies)
2. Identify which master questions have been customized
3. Fetch master questions assigned to their organizations
4. EXCLUDE masters that have customized versions
5. Return: Customized questions + Unmodified master questions
```

**Result**: Org admins never see both the master and their customized version simultaneously.

## Examples

### Example 1: Master Question Flow
1. Superadmin creates master question "Weekly Check-in"
2. Assigns to Org A, Org B, Org C
3. All three orgs see "Weekly Check-in"
4. Org A admin changes title to "Weekly Team Sync"
5. **Org A now sees**: "Weekly Team Sync" (customized)
6. **Org B and C see**: "Weekly Check-in" (master)
7. Org A admin deletes their customized question
8. **Org A now sees**: "Weekly Check-in" (master again)

### Example 2: Multiple Customizations
1. Master question assigned to 5 organizations
2. 3 organizations customize it
3. **Result**: 1 master + 3 customized versions (6 total questions in DB)
4. Each org sees only their version (customized or master)
5. Superadmin sees all 6 versions

## Best Practices

1. **Master Question Naming**: Use generic, reusable titles for master questions
2. **Assignment Strategy**: Assign to organizations that need similar questions
3. **Customization Guidance**: Organizations should only customize when necessary
4. **Deletion Awareness**: Deleting a master question affects all organizations
5. **Response Tracking**: Responses are tied to the actual question ID (master or customized)

## Technical Notes

- Uses soft delete for all question deletions
- Customized questions maintain reference to master via `masterQuestionId`
- Reactions and responses are linked to the actual question ID (not the master)
- Query optimization uses indexes on `isMasterQuestion`, `masterQuestionId`, and `assignedOrganizations`

