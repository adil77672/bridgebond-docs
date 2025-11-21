# Weekly Digest Frontend Integration Guide

This guide provides complete frontend integration instructions for the Weekly Digest feature.

## 📋 Table of Contents

1. [API Endpoints Overview](#api-endpoints-overview)
2. [Authentication](#authentication)
3. [Frontend Integration Examples](#frontend-integration-examples)
4. [UI Components](#ui-components)
5. [Error Handling](#error-handling)
6. [Best Practices](#best-practices)

---

## 🔌 API Endpoints Overview

### Base URL
```
https://api.bridgebond.techsufi.pro/v1
```

### Endpoints

#### 1. Get All Organizations Digest Status (Super Admin Only)
```
GET /weekly-digest/organizations
```

**Query Parameters:**
- `search` (string, optional): Search organizations by name
- `status` (string, optional): Filter by status (`enabled`, `disabled`)
- `page` (number, default: 1): Page number
- `limit` (number, default: 10): Items per page
- `sortBy` (string, optional): Sort field (e.g., `organization.name:asc`)

**Response:**
```json
{
  "results": [
    {
      "id": "org123",
      "organization": {
        "id": "org123",
        "name": "TechCorp Inc",
        "logo": "https://..."
      },
      "status": "Enable",
      "schedule": "Monday, 09:00 AM",
      "openRate": "78%",
      "lastSent": "04-05-2025",
      "enabled": true
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 1,
  "totalResults": 5
}
```

#### 2. Get Digest Configuration
```
GET /weekly-digest/:organizationId
```

**Response:**
```json
{
  "id": "digest123",
  "organizationId": "org123",
  "enabled": true,
  "sendDay": "monday",
  "sendTime": "10:00",
  "sendTimeFormatted": "10:00 AM",
  "audience": {
    "type": "all_organization",
    "departmentIds": [],
    "userIds": []
  },
  "audienceDisplay": "All Organization",
  "contentSettings": {
    "includeBirthdays": true,
    "includeWorkAnniversaries": true,
    "includeNewMembers": true,
    "includeQuestions": false,
    "includeReactions": false,
    "includeGiftSuggestions": false
  },
  "lastSentAt": "2025-12-12T09:00:00Z",
  "nextScheduledAt": "2025-12-19T10:00:00Z"
}
```

#### 3. Update Digest Configuration
```
PATCH /weekly-digest/:organizationId
```

**Request Body:**
```json
{
  "enabled": true,
  "sendDay": "monday",
  "sendTime": "10:00",
  "audience": {
    "type": "all_organization",
    "departmentIds": [],
    "userIds": []
  },
  "contentSettings": {
    "includeBirthdays": true,
    "includeWorkAnniversaries": true,
    "includeNewMembers": true,
    "includeQuestions": false,
    "includeReactions": false,
    "includeGiftSuggestions": false
  }
}
```

**Note:** You can send time in either format:
- 24-hour: `"10:00"` or `"14:30"`
- 12-hour: `"10:00 AM"` or `"02:30 PM"` (will be auto-converted)

#### 4. Get Digest History
```
GET /weekly-digest/:organizationId/history
```

**Query Parameters:**
- `status` (string, optional): Filter by status (`sent`, `failed`, `partial`)
- `page` (number, default: 1)
- `limit` (number, default: 10)

**Response:**
```json
{
  "results": [
    {
      "id": "log123",
      "sentAt": "2025-12-12T09:00:00Z",
      "formattedDate": "Dec 12, 2025",
      "formattedTime": "Monday, 09:00 AM",
      "status": "sent",
      "statusDisplay": "Sent",
      "audienceDisplay": "All Organization",
      "recipients": {
        "total": 50,
        "sent": 50,
        "failed": 0
      },
      "contentIncluded": {
        "birthdays": 3,
        "workAnniversaries": 2,
        "newMembers": 1
      }
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 1,
  "totalResults": 3
}
```

#### 5. Send Digest Manually
```
POST /weekly-digest/:organizationId/send
```

**Response:**
```json
{
  "id": "log123",
  "sentAt": "2025-12-12T09:00:00Z",
  "status": "sent",
  "recipients": {
    "total": 50,
    "sent": 50,
    "failed": 0
  }
}
```

---

## 🔐 Authentication

All endpoints require Bearer token authentication:

```javascript
headers: {
  'Authorization': `Bearer ${accessToken}`,
  'Content-Type': 'application/json'
}
```

**Permissions:**
- **Super Admin**: Can access and manage digest settings for ANY organization
- **Org Admin**: Can only access and manage digest settings for their own organization

---

## 💻 Frontend Integration Examples

### React/TypeScript Example

#### 1. API Service

```typescript
// services/weeklyDigest.service.ts
import axios from 'axios';

const API_BASE_URL = process.env.REACT_APP_API_URL || 'https://api.bridgebond.techsufi.pro/v1';

interface WeeklyDigestConfig {
  id: string;
  organizationId: string;
  enabled: boolean;
  sendDay: 'monday' | 'tuesday' | 'wednesday' | 'thursday' | 'friday' | 'saturday' | 'sunday';
  sendTime: string;
  sendTimeFormatted: string;
  audience: {
    type: 'all_organization' | 'all_departments' | 'specific_departments' | 'specific_users';
    departmentIds: string[];
    userIds: string[];
  };
  audienceDisplay: string;
  contentSettings: {
    includeBirthdays: boolean;
    includeWorkAnniversaries: boolean;
    includeNewMembers: boolean;
    includeQuestions: boolean;
    includeReactions: boolean;
    includeGiftSuggestions: boolean;
  };
  lastSentAt: string | null;
  nextScheduledAt: string | null;
}

interface DigestHistoryItem {
  id: string;
  sentAt: string;
  formattedDate: string;
  formattedTime: string;
  status: 'sent' | 'failed' | 'partial';
  statusDisplay: string;
  audienceDisplay: string;
  recipients: {
    total: number;
    sent: number;
    failed: number;
  };
  contentIncluded: {
    birthdays: number;
    workAnniversaries: number;
    newMembers: number;
    questions: number;
    reactions: number;
  };
}

class WeeklyDigestService {
  private getAuthHeaders() {
    const token = localStorage.getItem('accessToken');
    return {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    };
  }

  // Get all organizations digest status (Super Admin only)
  async getAllOrganizationsDigestStatus(params?: {
    search?: string;
    status?: 'enabled' | 'disabled';
    page?: number;
    limit?: number;
    sortBy?: string;
  }) {
    const response = await axios.get(`${API_BASE_URL}/weekly-digest/organizations`, {
      headers: this.getAuthHeaders(),
      params
    });
    return response.data;
  }

  // Get digest configuration
  async getDigestConfig(organizationId: string): Promise<WeeklyDigestConfig> {
    const response = await axios.get(
      `${API_BASE_URL}/weekly-digest/${organizationId}`,
      { headers: this.getAuthHeaders() }
    );
    return response.data;
  }

  // Update digest configuration
  async updateDigestConfig(
    organizationId: string,
    data: Partial<WeeklyDigestConfig>
  ): Promise<WeeklyDigestConfig> {
    const response = await axios.patch(
      `${API_BASE_URL}/weekly-digest/${organizationId}`,
      data,
      { headers: this.getAuthHeaders() }
    );
    return response.data;
  }

  // Get digest history
  async getDigestHistory(
    organizationId: string,
    params?: {
      status?: 'sent' | 'failed' | 'partial';
      page?: number;
      limit?: number;
    }
  ) {
    const response = await axios.get(
      `${API_BASE_URL}/weekly-digest/${organizationId}/history`,
      {
        headers: this.getAuthHeaders(),
        params
      }
    );
    return response.data;
  }

  // Send digest manually
  async sendDigest(organizationId: string) {
    const response = await axios.post(
      `${API_BASE_URL}/weekly-digest/${organizationId}/send`,
      {},
      { headers: this.getAuthHeaders() }
    );
    return response.data;
  }
}

export const weeklyDigestService = new WeeklyDigestService();
```

#### 2. React Hook Example

```typescript
// hooks/useWeeklyDigest.ts
import { useState, useEffect } from 'react';
import { weeklyDigestService } from '../services/weeklyDigest.service';

export const useWeeklyDigest = (organizationId: string) => {
  const [config, setConfig] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    loadConfig();
  }, [organizationId]);

  const loadConfig = async () => {
    try {
      setLoading(true);
      const data = await weeklyDigestService.getDigestConfig(organizationId);
      setConfig(data);
      setError(null);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  const updateConfig = async (updates: any) => {
    try {
      setLoading(true);
      const updated = await weeklyDigestService.updateDigestConfig(organizationId, updates);
      setConfig(updated);
      setError(null);
      return updated;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };

  return { config, loading, error, updateConfig, refresh: loadConfig };
};
```

#### 3. React Component Example

```tsx
// components/WeeklyDigestSettings.tsx
import React, { useState } from 'react';
import { useWeeklyDigest } from '../hooks/useWeeklyDigest';

interface Props {
  organizationId: string;
}

export const WeeklyDigestSettings: React.FC<Props> = ({ organizationId }) => {
  const { config, loading, error, updateConfig } = useWeeklyDigest(organizationId);
  const [saving, setSaving] = useState(false);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!config) return null;

  const handleSave = async () => {
    setSaving(true);
    try {
      // Convert 12-hour time to 24-hour if needed
      const sendTime = config.sendTime.includes('AM') || config.sendTime.includes('PM')
        ? convertTo24Hour(config.sendTime)
        : config.sendTime;

      await updateConfig({
        enabled: config.enabled,
        sendDay: config.sendDay,
        sendTime,
        audience: config.audience,
        contentSettings: config.contentSettings
      });
      alert('Settings saved successfully!');
    } catch (err) {
      alert('Failed to save settings');
    } finally {
      setSaving(false);
    }
  };

  return (
    <div className="weekly-digest-settings">
      <h2>Weekly Digest Settings</h2>
      
      {/* Enable/Disable Toggle */}
      <div className="form-group">
        <label>
          <input
            type="checkbox"
            checked={config.enabled}
            onChange={(e) => updateConfig({ enabled: e.target.checked })}
          />
          Enable Weekly Digest
        </label>
      </div>

      {/* Send Day */}
      <div className="form-group">
        <label>Send Day</label>
        <select
          value={config.sendDay}
          onChange={(e) => updateConfig({ sendDay: e.target.value })}
        >
          <option value="monday">Monday</option>
          <option value="tuesday">Tuesday</option>
          <option value="wednesday">Wednesday</option>
          <option value="thursday">Thursday</option>
          <option value="friday">Friday</option>
          <option value="saturday">Saturday</option>
          <option value="sunday">Sunday</option>
        </select>
      </div>

      {/* Send Time */}
      <div className="form-group">
        <label>Send Time</label>
        <input
          type="time"
          value={config.sendTime}
          onChange={(e) => updateConfig({ sendTime: e.target.value })}
        />
        <small>Current: {config.sendTimeFormatted}</small>
      </div>

      {/* Audience */}
      <div className="form-group">
        <label>Audience</label>
        <select
          value={config.audience.type}
          onChange={(e) => updateConfig({
            audience: { ...config.audience, type: e.target.value }
          })}
        >
          <option value="all_organization">All Organization</option>
          <option value="all_departments">All Department</option>
          <option value="specific_departments">Specific Department</option>
          <option value="specific_users">Specific Users</option>
        </select>
      </div>

      {/* Content Settings */}
      <div className="form-group">
        <h3>Content Settings</h3>
        <label>
          <input
            type="checkbox"
            checked={config.contentSettings.includeBirthdays}
            onChange={(e) => updateConfig({
              contentSettings: {
                ...config.contentSettings,
                includeBirthdays: e.target.checked
              }
            })}
          />
          Include Birthdays
        </label>
        <label>
          <input
            type="checkbox"
            checked={config.contentSettings.includeWorkAnniversaries}
            onChange={(e) => updateConfig({
              contentSettings: {
                ...config.contentSettings,
                includeWorkAnniversaries: e.target.checked
              }
            })}
          />
          Include Work Anniversaries
        </label>
        <label>
          <input
            type="checkbox"
            checked={config.contentSettings.includeNewMembers}
            onChange={(e) => updateConfig({
              contentSettings: {
                ...config.contentSettings,
                includeNewMembers: e.target.checked
              }
            })}
          />
          Include New Team Members
        </label>
        <label>
          <input
            type="checkbox"
            checked={config.contentSettings.includeGiftSuggestions}
            onChange={(e) => updateConfig({
              contentSettings: {
                ...config.contentSettings,
                includeGiftSuggestions: e.target.checked
              }
            })}
          />
          Include Gift Suggestions
        </label>
      </div>

      <button onClick={handleSave} disabled={saving}>
        {saving ? 'Saving...' : 'Save Changes'}
      </button>
    </div>
  );
};

// Helper function to convert 12-hour to 24-hour format
const convertTo24Hour = (time12: string): string => {
  const [time, period] = time12.split(' ');
  const [hours, minutes] = time.split(':');
  let hour24 = parseInt(hours);
  
  if (period === 'PM' && hour24 !== 12) hour24 += 12;
  if (period === 'AM' && hour24 === 12) hour24 = 0;
  
  return `${String(hour24).padStart(2, '0')}:${minutes}`;
};
```

#### 4. Super Admin List View Component

```tsx
// components/WeeklyDigestControl.tsx
import React, { useState, useEffect } from 'react';
import { weeklyDigestService } from '../services/weeklyDigest.service';

export const WeeklyDigestControl: React.FC = () => {
  const [organizations, setOrganizations] = useState([]);
  const [loading, setLoading] = useState(true);
  const [search, setSearch] = useState('');
  const [page, setPage] = useState(1);

  useEffect(() => {
    loadOrganizations();
  }, [search, page]);

  const loadOrganizations = async () => {
    try {
      setLoading(true);
      const data = await weeklyDigestService.getAllOrganizationsDigestStatus({
        search,
        page,
        limit: 10
      });
      setOrganizations(data.results);
    } catch (err) {
      console.error('Failed to load organizations:', err);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="weekly-digest-control">
      <h1>Weekly Digest Control</h1>
      
      {/* Search */}
      <input
        type="text"
        placeholder="Search Organization..."
        value={search}
        onChange={(e) => setSearch(e.target.value)}
      />

      {/* Table */}
      <table>
        <thead>
          <tr>
            <th>Organization</th>
            <th>Status</th>
            <th>Schedule</th>
            <th>Open Rate</th>
            <th>Last Sent</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {organizations.map((org) => (
            <tr key={org.id}>
              <td>{org.organization.name}</td>
              <td>{org.status}</td>
              <td>{org.schedule}</td>
              <td>{org.openRate || 'N/A'}</td>
              <td>{org.lastSent || 'Never'}</td>
              <td>
                <button onClick={() => handleEdit(org.organization.id)}>
                  Edit
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};
```

---

## 🎨 UI Components

### Recommended UI Elements

1. **Toggle Switch** for Enable/Disable
2. **Dropdown** for Send Day selection
3. **Time Picker** for Send Time (12-hour format with AM/PM)
4. **Dropdown** for Audience selection
5. **Checkboxes** for Content Settings
6. **Table** for History display
7. **Search Bar** for Super Admin organization list
8. **Pagination** for history and organization lists

---

## ⚠️ Error Handling

### Common Error Responses

```typescript
// 401 Unauthorized
{
  "code": 401,
  "message": "Please authenticate"
}

// 403 Forbidden
{
  "code": 403,
  "message": "Only organization admins and superadmins can access digest configuration"
}

// 404 Not Found
{
  "code": 404,
  "message": "Organization not found"
}

// 400 Bad Request
{
  "code": 400,
  "message": "Invalid time format. Use HH:MM (24-hour format)"
}
```

### Error Handling Example

```typescript
try {
  await weeklyDigestService.updateDigestConfig(organizationId, data);
} catch (error) {
  if (error.response) {
    switch (error.response.status) {
      case 401:
        // Redirect to login
        break;
      case 403:
        // Show permission error
        break;
      case 400:
        // Show validation error
        break;
      default:
        // Show generic error
    }
  }
}
```

---

## ✅ Best Practices

1. **Time Format Handling**
   - Accept both 12-hour and 24-hour formats
   - Display in 12-hour format for better UX
   - Store in 24-hour format internally

2. **Real-time Updates**
   - Use polling or WebSocket for status updates
   - Refresh configuration after updates

3. **Optimistic Updates**
   - Update UI immediately, rollback on error
   - Show loading states during API calls

4. **Validation**
   - Validate time format before submission
   - Check required fields (sendDay, sendTime)
   - Validate departmentIds/userIds when using specific audience

5. **Caching**
   - Cache digest configuration
   - Invalidate cache on updates
   - Use React Query or SWR for automatic caching

6. **User Feedback**
   - Show success messages after saves
   - Display error messages clearly
   - Provide loading indicators

---

## 📚 Additional Resources

- [API Documentation](http://localhost:3000/v1/docs) (Swagger UI)
- [Postman Collection](../postman-collection.json)
- [Weekly Digest Feature Docs](../development/WEEKLY_DIGEST_AND_USER_SETTINGS_FEATURE.md)

---

## 🆘 Support

For issues or questions, contact the development team or check the API documentation.

