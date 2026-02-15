# API Contract - Journal Transcription App

This document defines the API contract expected by the React frontend. Implement these endpoints in your FastAPI backend.

## Base URL

Development: `http://localhost:3001/api`

## Authentication

The API uses **session-based authentication** with cookies. All requests include `credentials: 'include'` to send cookies.

## Global Error Handling

All error responses should follow this format:

```json
{
  "message": "Error description",
  "error": "Optional detailed error"
}
```

HTTP Status Codes:
- `200` - Success
- `201` - Created
- `204` - No Content (successful delete)
- `400` - Bad Request (validation errors)
- `401` - Unauthorized (not logged in)
- `403` - Forbidden (logged in but no access)
- `404` - Not Found
- `500` - Internal Server Error

---

## Authentication Endpoints

### POST `/api/auth/register`

Register a new user account.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

**Response:** `201 Created`
```json
{
  "user": {
    "id": 1,
    "email": "user@example.com",
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

**Errors:**
- `400` - Email already exists
- `400` - Password too short (< 8 characters)
- `400` - Invalid email format

**Notes:**
- Must create a session cookie for the user
- Password should be hashed (bcrypt recommended)

---

### POST `/api/auth/login`

Log in an existing user.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

**Response:** `200 OK`
```json
{
  "user": {
    "id": 1,
    "email": "user@example.com",
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

**Errors:**
- `401` - Invalid email or password

**Notes:**
- Must create a session cookie for the user
- Return generic error message (don't specify if email or password is wrong)

---

### POST `/api/auth/logout`

Log out the current user.

**Request:** No body required

**Response:** `204 No Content`

**Notes:**
- Clear the session cookie
- Should succeed even if not logged in

---

### GET `/api/auth/me`

Get the currently logged-in user's information.

**Request:** No body required

**Response:** `200 OK`
```json
{
  "user": {
    "id": 1,
    "email": "user@example.com",
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

**Errors:**
- `401` - Not logged in

**Notes:**
- Used on app startup to check if user has an active session

---

## Entry Endpoints

All entry endpoints require authentication (401 if not logged in).

### GET `/api/entries`

Get all entries for the current user, optionally filtered.

**Query Parameters:**
- `startDate` (optional) - ISO date string (e.g., `2024-01-01`)
- `endDate` (optional) - ISO date string
- `page` (optional) - Page number for pagination (default: 1)
- `limit` (optional) - Items per page (default: 50)

**Example:** `/api/entries?startDate=2024-01-01&endDate=2024-12-31&page=1&limit=20`

**Response:** `200 OK`
```json
{
  "entries": [
    {
      "id": 1,
      "date": "2024-01-15",
      "transcription": "Today was a wonderful day...",
      "status": "transcribed",
      "page_id": 101,
      "createdAt": "2024-01-15T12:00:00Z",
      "updatedAt": "2024-01-15T14:30:00Z"
    },
    {
      "id": 2,
      "date": "2024-01-14",
      "transcription": null,
      "status": "pending",
      "page_id": 100,
      "createdAt": "2024-01-14T10:00:00Z",
      "updatedAt": "2024-01-14T10:00:00Z"
    }
  ],
  "total": 2
}
```

**Entry Object:**
- `id` - Unique entry ID
- `date` - Journal page date (ISO date string)
- `transcription` - Transcribed text (null if pending)
- `status` - Either `"pending"` or `"transcribed"`
- `page_id` - ID of the associated page image
- `createdAt` - When the entry was created
- `updatedAt` - When the entry was last modified

**Notes:**
- Should only return entries owned by the authenticated user
- Date filtering should be inclusive

---

### GET `/api/entries/:id`

Get a single entry by ID.

**Response:** `200 OK`
```json
{
  "entry": {
    "id": 1,
    "date": "2024-01-15",
    "transcription": "Today was a wonderful day. I spent the morning in the garden...",
    "status": "transcribed",
    "page_id": 101,
    "createdAt": "2024-01-15T12:00:00Z",
    "updatedAt": "2024-01-15T14:30:00Z"
  }
}
```

**Errors:**
- `404` - Entry not found
- `403` - Entry belongs to another user

---

### PUT `/api/entries/:id`

Update an entry (date or transcription).

**Request:**
```json
{
  "date": "2024-01-16",
  "transcription": "Updated transcription text..."
}
```

**Notes:**
- All fields are optional
- Only include fields you want to update

**Response:** `200 OK`
```json
{
  "entry": {
    "id": 1,
    "date": "2024-01-16",
    "transcription": "Updated transcription text...",
    "status": "transcribed",
    "page_id": 101,
    "createdAt": "2024-01-15T12:00:00Z",
    "updatedAt": "2024-01-16T09:15:00Z"
  }
}
```

**Errors:**
- `404` - Entry not found
- `403` - Entry belongs to another user
- `400` - Invalid date format

---

### DELETE `/api/entries/:id`

Delete an entry.

**Response:** `204 No Content`

**Errors:**
- `404` - Entry not found
- `403` - Entry belongs to another user

**Notes:**
- Should NOT delete the associated page image (only the entry)
- If this was the last entry for a page, the page remains

---

## Page Endpoints

All page endpoints require authentication (401 if not logged in).

### GET `/api/pages/:id`

Get a page image by ID.

**Response:** `200 OK`
```json
{
  "page": {
    "id": 101,
    "imageUrl": "http://localhost:3001/uploads/page-101.jpg",
    "date": "2024-01-15",
    "notes": "Morning journal entry",
    "createdAt": "2024-01-15T10:00:00Z"
  }
}
```

**Alternative response format (also accepted):**
```json
{
  "id": 101,
  "imageUrl": "http://localhost:3001/uploads/page-101.jpg",
  "date": "2024-01-15",
  "notes": "Morning journal entry",
  "createdAt": "2024-01-15T10:00:00Z"
}
```

**Page Object:**
- `id` - Unique page ID
- `imageUrl` - Full URL to the image file
- `date` - Journal page date
- `notes` - Optional notes about the page
- `createdAt` - When the page was uploaded

**Errors:**
- `404` - Page not found
- `403` - Page belongs to another user

**Notes:**
- The `imageUrl` should be accessible by the frontend
- Consider using signed URLs if using cloud storage

---

### POST `/api/pages`

Upload a new journal page image.

**Request:** `multipart/form-data`
- `image` (file, required) - Image file (jpg, png, gif, webp)
- `date` (string, required) - Journal page date (ISO format: YYYY-MM-DD)
- `notes` (string, optional) - Additional notes

**Example using FormData:**
```javascript
const formData = new FormData()
formData.append('image', fileObject)
formData.append('date', '2024-01-15')
formData.append('notes', 'Morning pages')
```

**Response:** `201 Created`
```json
{
  "page": {
    "id": 101,
    "imageUrl": "http://localhost:3001/uploads/page-101.jpg",
    "date": "2024-01-15",
    "notes": "Morning pages",
    "createdAt": "2024-01-15T10:00:00Z"
  }
}
```

**Errors:**
- `400` - No image file provided
- `400` - Invalid image format
- `400` - Missing required date field
- `400` - Invalid date format
- `413` - File too large (recommended max: 10MB)

**Notes:**
- Store the image in the filesystem or cloud storage
- Generate unique filename to avoid collisions
- Consider image compression/resizing for storage efficiency
- Should create an initial entry with `status: "pending"` and `transcription: null`

---

### DELETE `/api/pages/:id`

Delete a page and all its associated entries.

**Response:** `204 No Content`

**Errors:**
- `404` - Page not found
- `403` - Page belongs to another user

**Notes:**
- Should delete the image file from storage
- Should delete all entries associated with this page (cascade delete)
- This is a destructive operation - consider soft deletes

---

## Data Model Reference

### User
```typescript
{
  id: number
  email: string
  password_hash: string  // Never expose in API responses
  createdAt: string      // ISO 8601 timestamp
}
```

### Page
```typescript
{
  id: number
  user_id: number        // Foreign key to User
  imageUrl: string       // Full URL to image
  date: string           // ISO date (YYYY-MM-DD)
  notes?: string         // Optional notes
  createdAt: string      // ISO 8601 timestamp
}
```

### Entry
```typescript
{
  id: number
  user_id: number           // Foreign key to User
  page_id: number           // Foreign key to Page
  date: string              // ISO date (YYYY-MM-DD)
  transcription: string | null
  status: 'pending' | 'transcribed'
  createdAt: string         // ISO 8601 timestamp
  updatedAt: string         // ISO 8601 timestamp
}
```

**Relationships:**
- One User has many Pages
- One User has many Entries
- One Page has many Entries
- One Entry belongs to one Page

---

## Implementation Notes for FastAPI

### Session Management
```python
from fastapi import FastAPI, Depends, HTTPException
from starlette.middleware.sessions import SessionMiddleware

app = FastAPI()
app.add_middleware(SessionMiddleware, secret_key="your-secret-key")

# In your auth endpoints:
def create_session(request: Request, user_id: int):
    request.session["user_id"] = user_id

def get_current_user(request: Request):
    user_id = request.session.get("user_id")
    if not user_id:
        raise HTTPException(status_code=401, detail="Not authenticated")
    return get_user_by_id(user_id)
```

### CORS Configuration
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],  # Vite dev server
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### File Upload
```python
from fastapi import File, UploadFile, Form

@app.post("/api/pages")
async def upload_page(
    image: UploadFile = File(...),
    date: str = Form(...),
    notes: str = Form(None)
):
    # Save file and return response
    pass
```

### Date Filtering
```python
from datetime import date

@app.get("/api/entries")
async def get_entries(
    startDate: date = None,
    endDate: date = None,
    page: int = 1,
    limit: int = 50
):
    # Query with filters
    pass
```

---

## Testing the API

You can test the API with curl:

```bash
# Register
curl -X POST http://localhost:3001/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}' \
  -c cookies.txt

# Login
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}' \
  -c cookies.txt

# Get entries (authenticated)
curl http://localhost:3001/api/entries -b cookies.txt

# Upload page
curl -X POST http://localhost:3001/api/pages \
  -F "image=@/path/to/image.jpg" \
  -F "date=2024-01-15" \
  -F "notes=Test upload" \
  -b cookies.txt
```
