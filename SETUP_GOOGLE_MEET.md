# 🚀 Google Meet Integration - Complete Setup Guide

## ✅ What We've Done

### 1. Backend Migration (COMPLETED)
- ✅ Replaced LiveKit SDK with Google APIs (`googleapis`)
- ✅ Implemented OAuth2 authentication flow
- ✅ Created endpoints for Google Meet space creation
- ✅ Updated package.json dependencies
- ✅ Installed new dependencies (`npm install` completed)

### 2. Frontend Updates (COMPLETED)
- ✅ Modified Dashboard to handle Google OAuth flow
- ✅ Updated meeting creation to redirect to Google Meet
- ✅ Changed "Join Now" button to open Google Meet in new tab

### 3. Configuration Files Created
- ✅ `server.js` - New backend with Google Meet API
- ✅ `supabase-migration-google-meet.sql` - Database migration script
- ✅ `GOOGLE_MEET_INTEGRATION_GUIDE.md` - Detailed documentation

---

## 📋 Next Steps (What YOU Need to Do)

### Step 1: Update Your `.env` File

Navigate to `polgotmeet_backend/` and create/update the `.env` file:

```bash
cd polgotmeet_backend
```

Create a file named `.env` with this content:

```env
# Google OAuth Credentials
GOOGLE_CLIENT_ID=YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=YOUR_GOOGLE_CLIENT_SECRET
GOOGLE_REDIRECT_URI=http://localhost:3000/auth/google/callback

# Supabase Configuration - REPLACE THESE!
SUPABASE_URL=https://your-project-id.supabase.co
SUPABASE_SERVICE_KEY=your-service-role-key-here

# Server Configuration
PORT=3000
NODE_ENV=development
FRONTEND_URL=http://localhost:5173
```

**IMPORTANT**: Replace `SUPABASE_URL` and `SUPABASE_SERVICE_KEY` with your actual values from Supabase Dashboard.

---

### Step 2: Run Supabase Migration

1. Go to your **Supabase Dashboard**: https://supabase.com/dashboard
2. Select your project
3. Go to **SQL Editor** (left sidebar)
4. Click **"New Query"**
5. Copy and paste the contents of `polgotmeet_backend/supabase-migration-google-meet.sql`
6. Click **"Run"** to execute the migration

This will:
- Create `user_tokens` table for storing Google OAuth tokens
- Add Google Meet columns to `meetings` table
- Set up Row Level Security policies

---

### Step 3: Update Google Cloud Console Redirect URIs

1. Go to **Google Cloud Console**: https://console.cloud.google.com/
2. Navigate to **APIs & Services** → **Credentials**
3. Click on your OAuth 2.0 Client ID
4. Under **Authorized redirect URIs**, make sure you have:
   - `http://localhost:3000/auth/google/callback` (for local testing)
5. Click **SAVE**

**For Production (Railway)**: When you deploy, add your Railway URL:
   - `https://your-app.railway.app/auth/google/callback`

---

### Step 4: Test Locally

1. **Start the backend**:
```bash
cd polgotmeet_backend
npm start
```

You should see:
```
✅ Backend Server running on port 3000
✅ Environment: development
✅ Google Meet API Integration Active
✅ Supabase URL: https://your-project.supabase.co
```

2. **Start the frontend** (in a new terminal):
```bash
cd ..
npm run dev
```

3. **Test the flow**:
   - Open http://localhost:5173
   - Login to your app
   - Click "New Meeting" → "Create Link"
   - You should be redirected to Google OAuth consent screen
   - After granting permissions, you'll be redirected back
   - Try creating a meeting again - it should now work!

---

## 🔄 How It Works Now

### First Time Creating a Meeting:
1. User clicks "Create Meeting"
2. Backend checks if user has Google OAuth tokens
3. If not, returns `needsAuth: true` with auth URL
4. Frontend redirects user to Google OAuth
5. User grants permissions
6. Google redirects back to `/auth/google/callback`
7. Backend stores tokens in Supabase
8. User is redirected to dashboard

### Subsequent Meeting Creation:
1. User clicks "Create Meeting"
2. Backend uses stored tokens to create Google Meet space
3. Returns meeting details with Google Meet URI
4. User can share the meeting or click "Join Now"
5. "Join Now" opens Google Meet in a new tab

### Joining a Meeting:
- Users can still use meeting code + password
- Backend validates and returns Google Meet URI
- Frontend redirects to Google Meet

---

## 🚀 Deploy to Railway (Production)

### 1. Update Environment Variables in Railway:

Go to your Railway project → Variables and add:

```
GOOGLE_CLIENT_ID=YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=YOUR_GOOGLE_CLIENT_SECRET
GOOGLE_REDIRECT_URI=https://your-app.railway.app/auth/google/callback
SUPABASE_URL=your-supabase-url
SUPABASE_SERVICE_KEY=your-service-key
NODE_ENV=production
FRONTEND_URL=https://your-frontend-url.com
```

### 2. Update Google Cloud Console:

Add your Railway redirect URI:
- `https://your-app.railway.app/auth/google/callback`

### 3. Deploy:

```bash
git add .
git commit -m "Migrate to Google Meet API"
git push
```

Railway will automatically deploy your changes.

---

## 🐛 Troubleshooting

### "User not authenticated with Google"
**Solution**: This is expected on first use. User needs to complete OAuth flow.

### "redirect_uri_mismatch" error
**Solution**: Make sure the redirect URI in your `.env` matches exactly what's in Google Cloud Console.

### "Insufficient permissions" error
**Solution**: 
1. Go to Google Cloud Console → APIs & Services → OAuth consent screen
2. Click "EDIT APP"
3. Go to "Scopes" section
4. Make sure these scopes are added:
   - `https://www.googleapis.com/auth/meetings.space.created`
   - `https://www.googleapis.com/auth/meetings.space.readonly`

### Tokens expired
**Solution**: The backend automatically refreshes expired tokens. If issues persist, user needs to re-authenticate.

---

## 📝 Important Notes

1. **Google Meet API Limits**: Google Meet API has usage quotas. Monitor your usage in Google Cloud Console.

2. **OAuth Consent Screen**: If your app is in "Testing" mode, only test users can authenticate. To make it public, you need to verify your app with Google.

3. **Token Storage**: OAuth tokens are stored securely in Supabase with RLS policies. Only the user can access their own tokens.

4. **Meeting Persistence**: Meeting details are still stored in your Supabase `meetings` table for tracking purposes.

---

## ✨ What's Different from LiveKit?

| Feature | LiveKit (Old) | Google Meet (New) |
|---------|--------------|-------------------|
| Video Infrastructure | Self-hosted LiveKit | Google Meet |
| Authentication | LiveKit tokens | Google OAuth 2.0 |
| Meeting UI | Custom React components | Google Meet web interface |
| Scalability | Depends on LiveKit server | Google's infrastructure |
| Cost | LiveKit hosting costs | Free for personal use |
| Features | Custom implementation | Full Google Meet features |

---

## 🎯 Summary

**What's Working:**
- ✅ Backend fully migrated to Google Meet API
- ✅ Frontend updated to handle OAuth and redirect to Google Meet
- ✅ Database migration script ready
- ✅ All dependencies installed

**What You Need to Do:**
1. ⏳ Update `.env` file with Supabase credentials
2. ⏳ Run Supabase migration SQL
3. ⏳ Verify Google Cloud Console redirect URIs
4. ⏳ Test locally
5. ⏳ Deploy to Railway (optional)

---

Need help with any step? Let me know! 🚀
