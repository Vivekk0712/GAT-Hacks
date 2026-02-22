# YouTube Transcript Fetching Fix

## Problem

The backend was using `youtube_transcript_api` Python library to fetch YouTube transcripts, which was being blocked by YouTube's anti-bot measures when requests came from server IPs. This caused lesson generation to fail frequently.

## Solution

Move YouTube transcript fetching to the **frontend (browser)** where it uses the user's normal IP address, which is less likely to be blocked by YouTube.

## Architecture Changes

### Before (Backend Fetching)
```
Frontend → Backend → YouTube API → Transcript → Backend → AI Processing → Frontend
                ❌ Often blocked by YouTube
```

### After (Frontend Fetching)
```
Frontend → YouTube API → Transcript → Backend → AI Processing → Frontend
           ✅ Uses user's IP (less likely blocked)
```

## Implementation Details

### 1. Frontend Changes

#### New File: `frontend/src/lib/youtubeTranscript.ts`
- Uses `youtube-transcript` npm package
- Fetches transcripts directly from browser
- Tries multiple videos if first one fails
- Handles errors gracefully

```typescript
import { YoutubeTranscript } from 'youtube-transcript';

export async function fetchYoutubeTranscript(videoUrl: string): Promise<TranscriptResult>
export async function fetchYoutubeTranscriptMultiple(videoUrls: string[]): Promise<TranscriptResult>
```

#### Updated: `frontend/src/lib/api.ts`
- Modified `generateLessonContent()` to:
  1. Search for YouTube videos (backend provides URLs)
  2. Fetch transcripts from frontend
  3. Send transcript data to backend for processing

### 2. Backend Changes

#### Updated: `backend/models.py`
- Added `VideoTranscriptData` model
- Added `LessonRequestWithTranscript` model

```python
class VideoTranscriptData(BaseModel):
    text: str
    url: str
    video_id: str

class LessonRequestWithTranscript(BaseModel):
    topic: str
    user_preference: str
    video_transcript: Optional[VideoTranscriptData] = None
```

#### Updated: `backend/main.py`
- Added `/search-youtube` endpoint (returns video URLs only)
- Modified `/generate-lesson-content` to accept pre-fetched transcripts
- Falls back to backend fetching if frontend doesn't provide transcript

```python
@app.get("/search-youtube")
async def search_youtube(query: str):
    # Returns video metadata without transcripts
    
@app.post("/generate-lesson-content")
async def generate_lesson_content(request: LessonRequestWithTranscript):
    # Accepts optional video_transcript from frontend
```

## Benefits

1. **Reliability**: User IPs are less likely to be blocked by YouTube
2. **Performance**: Parallel fetching of video search and transcript
3. **Fallback**: Backend can still try if frontend fails
4. **Transparency**: Users see transcript fetching in browser console
5. **Scalability**: Distributes load across user browsers

## Installation

### Frontend
```bash
cd frontend
npm install youtube-transcript
```

### Backend
No new dependencies required. The `youtube_transcript_api` can remain for fallback purposes.

## Usage Flow

1. User requests a lesson on a topic
2. Frontend calls `/search-youtube` to get video URLs
3. Frontend fetches transcript using `youtube-transcript` library
4. Frontend sends transcript + topic to `/generate-lesson-content`
5. Backend processes transcript with AI
6. Backend returns structured lesson content

## Error Handling

### Frontend Transcript Fetch Fails
- Logs warning to console
- Sends request without transcript
- Backend attempts fallback fetch (may also fail)
- If all fails, AI generates lesson from internal knowledge

### No Videos Found
- Backend returns empty array
- Frontend proceeds without video content
- Lesson generated from documentation only

### Network Issues
- Standard axios error handling
- User-friendly error messages displayed

## Testing

### Test Frontend Transcript Fetching
```typescript
import { fetchYoutubeTranscript } from './lib/youtubeTranscript';

const result = await fetchYoutubeTranscript('https://www.youtube.com/watch?v=VIDEO_ID');
console.log(result.text); // Transcript text
```

### Test Backend Endpoint
```bash
# Search for videos
curl "http://localhost:8001/search-youtube?query=Docker%20tutorial"

# Generate lesson with transcript
curl -X POST "http://localhost:8001/generate-lesson-content" \
  -H "Content-Type: application/json" \
  -d '{
    "topic": "Docker Basics",
    "user_preference": "visual",
    "video_transcript": {
      "text": "...",
      "url": "https://youtube.com/watch?v=...",
      "video_id": "..."
    }
  }'
```

## Monitoring

Check browser console for:
- `Fetching transcript for video: VIDEO_ID`
- `✓ Transcript fetched: N characters`
- `✗ Failed: error message`

Check backend logs for:
- `Using pre-fetched video transcript from frontend`
- `✓ Video transcript added: N chars`

## Future Improvements

1. **Caching**: Cache transcripts in localStorage to avoid re-fetching
2. **Language Support**: Try multiple languages if English fails
3. **Quality Selection**: Prefer auto-generated vs manual captions
4. **Retry Logic**: Exponential backoff for failed fetches
5. **Progress Indicators**: Show transcript fetching progress to user

## Rollback Plan

If issues occur, revert to backend-only fetching:

1. Remove `video_transcript` parameter from frontend API call
2. Backend will use existing `get_video_content()` function
3. No database changes required

## Related Files

- `frontend/src/lib/youtubeTranscript.ts` - New transcript fetcher
- `frontend/src/lib/api.ts` - Updated API client
- `backend/models.py` - New data models
- `backend/main.py` - Updated endpoints
- `backend/tools/content_tools.py` - Existing backend fetcher (fallback)

## Notes

- The `youtube-transcript` npm package uses YouTube's internal API
- No API key required for transcript fetching
- Rate limiting is per-user IP (not per-server)
- Works with public videos that have captions enabled
