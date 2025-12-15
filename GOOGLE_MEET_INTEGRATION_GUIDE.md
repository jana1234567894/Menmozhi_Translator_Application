# Google Meet Integration Guide

## ✅ Completed Steps

### 1. Google Cloud Console Setup
- ✅ Created Google Cloud Project
- ✅ Enabled Google Meet API
- ✅ Created OAuth 2.0 Credentials
  - **Client ID**: `YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com`
  - **Client Secret**: `YOUR_GOOGLE_CLIENT_SECRET`
- ✅ Configured OAuth Consent Screen
- ✅ Added Required Scopes

### 2. Backend Migration
- ✅ Replaced LiveKit with Google Meet API
- ✅ Updated package.json dependencies
- ✅ Implemented OAuth2 flow
- ✅ Created meeting management endpoints

---

## 🔧 Next Steps

### Step 1: Update Supabase Database Schema

You need to add a new table for storing user OAuth tokens and update the meetings table.

**Go to your Supabase Dashboard** → **SQL Editor** and run these queries:

```sql
-- Create user_tokens table for storing Google OAuth tokens
CREATE TABLE IF NOT EXISTS user_tokens (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    access_token TEXT NOT NULL,
    refresh_token TEXT,
    expiry_date BIGINT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(user_id)
);

-- Add RLS policies for user_tokens
ALTER TABLE user_tokens ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view their own tokens"
    ON user_tokens FOR SELECT
    USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own tokens"
    ON user_tokens FOR INSERT
    WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own tokens"
    ON user_tokens FOR UPDATE
    USING (auth.uid() = user_id);

-- Update meetings table to include Google Meet fields
ALTER TABLE meetings 
ADD COLUMN IF NOT EXISTS google_meet_space_name TEXT,
ADD COLUMN IF NOT EXISTS google_meet_uri TEXT,
ADD COLUMN IF NOT EXISTS google_meet_code TEXT;

-- Drop livekit_room column if it exists
ALTER TABLE meetings DROP COLUMN IF EXISTS livekit_room;
```

### Step 2: Update Environment Variables

Create/update `.env` file in `polgotmeet_backend/` directory:

```env
# Google OAuth Credentials
GOOGLE_CLIENT_ID=YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=YOUR_GOOGLE_CLIENT_SECRET
GOOGLE_REDIRECT_URI=http://localhost:3000/auth/google/callback

# Supabase Configuration
SUPABASE_URL=your_supabase_url_here
SUPABASE_SERVICE_KEY=your_supabase_service_key_here

# Server Configuration
PORT=3000
NODE_ENV=development
FRONTEND_URL=http://localhost:5173
```

**Replace** `your_supabase_url_here` and `your_supabase_service_key_here` with your actual Supabase credentials.

### Step 3: Install Dependencies

```bash
cd polgotmeet_backend
npm install
```

### Step 4: Update Google Cloud Console Redirect URIs

Go back to **Google Cloud Console** → **APIs & Services** → **Credentials**:

1. Click on your OAuth 2.0 Client ID
2. Under **Authorized redirect URIs**, add:
   - `http://localhost:3000/auth/google/callback` (for local development)
   - Your production URL when deploying (e.g., `https://your-railway-app.railway.app/auth/google/callback`)
3. Click **SAVE**

### Step 5: Test the Backend Locally

```bash
cd polgotmeet_backend
npm start
```

The server should start on `http://localhost:3000`

---

## 🔄 How the New Flow Works

### 1. **User Authentication with Google**
   - User clicks "Create Meeting" in frontend
   - Frontend calls `GET /auth/google?userId={userId}`
   - Backend returns Google OAuth URL
   - User is redirected to Google consent screen
   - After consent, Google redirects to `/auth/google/callback`
   - Backend stores access/refresh tokens in Supabase
   - User is redirected back to frontend

### 2. **Create Meeting**
   - Frontend calls `POST /create-meeting` with `userId`
   - Backend retrieves user's stored Google tokens
   - Backend creates a Google Meet Space using Google Meet API
   - Backend stores meeting details in Supabase
   - Returns meeting URI and code to frontend

### 3. **Join Meeting**
   - Frontend calls `POST /join-meeting` with `meetingId`, `password`, `userId`
   - Backend validates credentials
   - Returns Google Meet URI
   - Frontend redirects user to Google Meet

### 4. **End Meeting**
   - Frontend calls `POST /end-meeting` with `meetingId`, `userId`
   - Backend verifies host authority
   - Backend ends the Google Meet conference
   - Marks meeting as inactive in database

---

## 📋 API Endpoints

### `GET /auth/google?userId={userId}`
Initiates OAuth flow for a user.

**Response:**
```json
{
  "authUrl": "https://accounts.google.com/o/oauth2/v2/auth?..."
}
```

### `GET /auth/google/callback`
OAuth callback endpoint (handled automatically by Google).

### `POST /create-meeting`
Creates a new Google Meet meeting.

**Request:**
```json
{
  "userId": "user-uuid"
}
```

**Response:**
```json
{
  "meetingId": "ABC-DEF-GHI",
  "password": "123456",
  "meetingUri": "https://meet.google.com/xxx-yyyy-zzz",
  "meetingCode": "xxx-yyyy-zzz",
  "spaceName": "spaces/..."
}
```

**Error (if not authenticated):**
```json
{
  "error": "User not authenticated with Google",
  "needsAuth": true,
  "authUrl": "/auth/google?userId=..."
}
```

### `POST /join-meeting`
Join an existing meeting.

**Request:**
```json
{
  "meetingId": "ABC-DEF-GHI",
  "password": "123456",
  "userId": "user-uuid"
}
```

**Response:**
```json
{
  "meetingUri": "https://meet.google.com/xxx-yyyy-zzz",
  "meetingCode": "xxx-yyyy-zzz",
  "isHost": false
}
```

### `POST /end-meeting`
End a meeting (host only).

**Request:**
```json
{
  "meetingId": "ABC-DEF-GHI",
  "userId": "user-uuid"
}
```

**Response:**
```json
{
  "success": true
}
```

---

## 🚀 Frontend Integration Changes Needed

You'll need to update your frontend to:

1. **Handle OAuth Flow**: When creating a meeting, check if user needs to authenticate first
2. **Redirect to Google Meet**: Instead of using LiveKit components, redirect users to the Google Meet URI
3. **Remove LiveKit Components**: Remove all LiveKit-related UI components

Would you like me to update the frontend code as well?

---

## 🔒 Security Notes

- OAuth tokens are stored securely in Supabase with RLS policies
- Tokens are automatically refreshed when expired
- Only hosts can end meetings
- Password validation for non-host participants
- All API endpoints have proper error handling

---

## 🐛 Troubleshooting

### "User not authenticated with Google"
- User needs to complete OAuth flow first
- Frontend should redirect to the `authUrl` provided in the error

### "Invalid grant" error
- Refresh token may be invalid
- User needs to re-authenticate

### "Insufficient permissions"
- Verify all required scopes are added in Google Cloud Console
- User needs to re-consent with updated scopes

---

## 📝 TODO

- [ ] Run Supabase SQL migrations
- [ ] Update `.env` file with credentials
- [ ] Install npm dependencies
- [ ] Update Google Cloud Console redirect URIs
- [ ] Test backend locally
- [ ] Update frontend to use new API
- [ ] Deploy to Railway/production
- [ ] Update production redirect URIs in Google Cloud Console
