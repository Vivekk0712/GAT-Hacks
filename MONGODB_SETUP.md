# MongoDB Setup Guide

## Overview
This guide will help you migrate from JSON file storage to MongoDB for storing user data, roadmaps, and progress.

## Prerequisites
- MongoDB Atlas account (free tier available) OR local MongoDB installation
- Python with pymongo installed

## Step 1: Get MongoDB Connection String

### Option A: MongoDB Atlas (Cloud - Recommended)

1. Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. Sign up for a free account
3. Create a new cluster (free M0 tier)
4. Click "Connect" on your cluster
5. Choose "Connect your application"
6. Copy the connection string (looks like):
   ```
   mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```
7. Replace `<password>` with your actual password
8. Add `/adapted` before the `?` to specify the database name:
   ```
   mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/adapted?retryWrites=true&w=majority
   ```

### Option B: Local MongoDB

1. Install MongoDB locally
2. Start MongoDB service
3. Use connection string:
   ```
   mongodb://localhost:27017/
   ```

## Step 2: Update Environment Variables

1. Open `backend/.env`
2. Update the MongoDB configuration:
   ```env
   MONGODB_URL=your_mongodb_connection_string_here
   MONGODB_DATABASE=adapted
   ```

Example with MongoDB Atlas:
```env
MONGODB_URL=mongodb+srv://myuser:mypassword@cluster0.xxxxx.mongodb.net/adapted?retryWrites=true&w=majority
MONGODB_DATABASE=adapted
```

Example with local MongoDB:
```env
MONGODB_URL=mongodb://localhost:27017/
MONGODB_DATABASE=adapted
```

## Step 3: Install MongoDB Driver

```bash
cd backend
pip install pymongo
```

Or install all requirements:
```bash
pip install -r requirements.txt
```

## Step 4: Switch to MongoDB

Open `backend/main.py` and find this line (around line 13):

```python
from database import save_user_profile, get_user, check_user_status, update_user_roadmap
# Uncomment the line below and comment the line above to use MongoDB
# from mongodb_database import save_user_profile, get_user, check_user_status, update_user_roadmap
```

Change it to:

```python
# from database import save_user_profile, get_user, check_user_status, update_user_roadmap
# Uncomment the line below and comment the line above to use MongoDB
from mongodb_database import save_user_profile, get_user, check_user_status, update_user_roadmap
```

## Step 5: Migrate Existing Data (Optional)

If you have existing data in `user_state.json`, you can migrate it to MongoDB:

```python
# Run this in Python console or create a migration script
from backend.mongodb_database import migrate_from_json

migrate_from_json()
```

Or create a migration script `backend/migrate.py`:

```python
from mongodb_database import migrate_from_json

if __name__ == "__main__":
    print("Starting migration from JSON to MongoDB...")
    migrate_from_json()
    print("Migration complete!")
```

Run it:
```bash
cd backend
python migrate.py
```

## Step 6: Test the Connection

Start your backend server:

```bash
cd backend
python main.py
```

You should see:
```
✓ Connected to MongoDB: adapted
```

## Step 7: Verify Data

### Check MongoDB Atlas Dashboard
1. Go to your cluster in MongoDB Atlas
2. Click "Browse Collections"
3. You should see the `adapted` database with a `users` collection

### Check Locally
Use MongoDB Compass or mongosh:
```bash
mongosh "your_connection_string"
use adapted
db.users.find()
```

## Database Structure

### Users Collection

```json
{
  "uid": "firebase_user_id",
  "profile": {
    "uid": "firebase_user_id",
    "goal": "Backend Developer",
    "current_skills": ["Python", "Git"],
    "preferred_language": "Go",
    "time_commitment": "10 hours per week",
    "notification_time": "20:00",
    "weekly_hours": 10
  },
  "roadmap": {
    "user_id": "go_backend_learner_1",
    "modules": [
      {
        "title": "Module 1: Git Version Control Essentials",
        "description": "...",
        "week": 1,
        "status": "completed",
        "viva_score": 85,
        "resources": ["..."]
      }
    ]
  },
  "onboarding_completed": true,
  "created_at": "2026-02-10T19:56:47.115537",
  "updated_at": "2026-02-10T23:44:36.696584"
}
```

## Benefits of MongoDB

✅ **Scalable**: Handles millions of users
✅ **Fast**: Optimized for read/write operations
✅ **Reliable**: Built-in replication and backup
✅ **Flexible**: Easy to add new fields
✅ **Cloud-based**: Access from anywhere (with Atlas)
✅ **Free tier**: MongoDB Atlas M0 is free forever

## Troubleshooting

### Connection Error
- Check your connection string is correct
- Ensure your IP is whitelisted in MongoDB Atlas
- Verify your username/password

### Import Error
```bash
pip install pymongo
```

### Migration Issues
- Check `user_state.json` exists
- Verify JSON format is valid
- Check MongoDB connection is working

## Rollback to JSON

If you need to go back to JSON storage:

1. Open `backend/main.py`
2. Change the import back:
   ```python
   from database import save_user_profile, get_user, check_user_status, update_user_roadmap
   ```
3. Restart the server

## Next Steps

After MongoDB is set up:
1. ✅ All user data will be stored in MongoDB
2. ✅ Dashboard will fetch real-time data from MongoDB
3. ✅ Learning Path will show accurate progress
4. ✅ Viva scores will be persisted
5. ✅ Multiple users can use the app simultaneously

## Support

If you encounter issues:
1. Check MongoDB Atlas status
2. Verify connection string
3. Check server logs for errors
4. Ensure pymongo is installed

---

**Ready to switch?** Just update your `.env` file with the MongoDB URL and uncomment the import in `main.py`!
