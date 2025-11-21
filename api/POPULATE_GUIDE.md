# Dynamic Populate Feature

## Quick Start

```javascript
// Single field
GET /api/v1/users?populate=[{"path":"department"}]

// Nested
GET /api/v1/users?populate=[{"path":"department","populate":["organizationId"]}]

// With options
GET /api/v1/users?populate=[{"path":"department","select":"name description"}]
```

## How It Works

The populate middleware allows dynamic population of reference fields via query parameters.

### 1. Add to Route
```javascript
import { validatePopulateOptions } from '../../middlewares/populate.js';
router.get('/', validatePopulateOptions(Model), controller.getAll);
```

### 2. Update Controller
```javascript
const getAll = catchAsync(async (req, res) => {
  const options = pick(req.query, ['sortBy', 'limit', 'page']);
  if (req.populateOptions) {
    options.populate = req.populateOptions;
  }
  const result = await service.queryAll(filter, options);
  res.send(result);
});
```

### 3. Update Service
```javascript
const getById = async (id, populateOptions = []) => {
  let query = Model.findById(id);
  if (populateOptions?.length > 0) {
    populateOptions.forEach(opt => query.populate(opt));
  }
  return query;
};
```

## Examples

### Basic
```json
[{"path":"department"}]
```

### Nested
```json
[{"path":"department","populate":["organizationId"]}]
```

### With Select
```json
[{"path":"department","select":"name description"}]
```

### With Match
```json
[{"path":"celebrations","match":{"isActive":true}}]
```

### Complex
```json
[{
  "path":"department",
  "select":"name",
  "populate":[{"path":"organizationId","select":"name"}]
}]
```

## Validation

- ✅ Schema path validation
- ✅ Reference field validation
- ✅ Nested populate validation
- ✅ Works with pagination

## Available on Models

Currently implemented on:
- User (`/v1/users`)
- Organization (`/v1/organizations`)
- Department (`/v1/departments`)
- Celebration (`/v1/celebrations`)
- Question (`/v1/questions`)

## Errors

| Error | Cause |
|-------|-------|
| Path "xxx" does not exist | Invalid field name |
| Not a reference field | Trying to populate non-ObjectId field |
| Invalid populate format | Malformed JSON |

## Implementation

Middleware: `src/middlewares/populate.js`

