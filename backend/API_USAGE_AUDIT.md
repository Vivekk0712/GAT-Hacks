# API Usage Audit - AdaptEd Backend

## Summary
The AdaptEd backend (not mcp-ide) uses **only Google/Gemini and Groq APIs**. OpenAI and Hugging Face are **NOT actively used**.

## APIs Actually Used

### ✅ Google Gemini API
**Status**: ACTIVELY USED
**Key**: `GEMINI_API_KEY`
**Purpose**: 
- Main AI model for content generation
- Roadmap generation
- Lesson planning
- Notes generation
- Viva (interview) fallback

**Used In**:
- `agents/content_agent.py`
- `agents/roadmap_agent.py`
- `agents/lesson_agent.py`
- `agents/notes_agent.py`
- `agents/viva_agent_gemini.py`

### ✅ YouTube API
**Status**: ACTIVELY USED
**Key**: `YOUTUBE_API_KEY`
**Purpose**: Video search for learning content
**Fallback**: DuckDuckGo if not available

**Used In**:
- `tools/content_tools.py`

### ✅ Groq API
**Status**: ACTIVELY USED (for Viva only)
**Key**: `GROQ_API_KEY`
**Purpose**: Speech-to-text (Whisper-v3) for Viva interviews
**Required For**: Viva Engine

**Used In**:
- `agents/viva_agent_free.py`
- Required by `/viva/start` and `/viva/respond` endpoints

## APIs NOT Used

### ❌ OpenAI API
**Status**: NOT USED
**Key**: `OPENAI_API_KEY`
**Current Value**: `your_openai_api_key_here` (placeholder)

**Evidence**:
- No imports of `openai` library in backend code
- No actual usage in any agents
- Only mentioned in startup logs as "not found"
- Can be safely removed

### ❌ Hugging Face API
**Status**: NOT USED
**Key**: `HUGGINGFACE_API_KEY`
**Current Value**: `your_huggingface_api_key_here` (placeholder)

**Evidence**:
- No imports of `huggingface_hub` or similar libraries
- No actual usage in any agents
- Only mentioned in startup logs as "not found"
- Can be safely removed

## Recommendations

### 1. Clean Up .env Files

**Remove from `.env` and `.env.example`:**
```bash
# These are NOT used
OPENAI_API_KEY=your_openai_api_key_here
HUGGINGFACE_API_KEY=your_huggingface_api_key_here
```

**Keep only:**
```bash
# Google Gemini API Key (REQUIRED)
GEMINI_API_KEY=AIzaSyArYLkVeCPaGt7GtTbFXdPJYDPrcLmwcpM
GOOGLE_API_KEY=AIzaSyArYLkVeCPaGt7GtTbFXdPJYDPrcLmwcpM

# YouTube API Key (OPTIONAL - for video search)
YOUTUBE_API_KEY=AIzaSyArYLkVeCPaGt7GtTbFXdPJYDPrcLmwcpM

# Groq API Key (REQUIRED for Viva Engine)
GROQ_API_KEY=your_groq_api_key_here
```

### 2. Update Startup Logs

**Remove from `main.py`:**
```python
# These checks are unnecessary
openai_api_key = os.getenv("OPENAI_API_KEY")
huggingface_api_key = os.getenv("HUGGINGFACE_API_KEY")

if openai_api_key:
    print(f"✓ OPENAI_API_KEY loaded...")
else:
    print("⚠ OPENAI_API_KEY not found - Viva Engine will not work")

if huggingface_api_key:
    print(f"✓ HUGGINGFACE_API_KEY loaded...")
else:
    print("⚠ HUGGINGFACE_API_KEY not found - Viva will use Gemini as fallback")
```

### 3. Update Documentation

Update any README or setup docs to reflect that only these APIs are needed:
- Google Gemini (required)
- YouTube (optional)
- Groq (required for Viva)

## API Cost Analysis

### Google Gemini
- **Free Tier**: 60 requests/minute
- **Cost**: Free for development
- **Usage**: High (all content generation)

### YouTube API
- **Free Tier**: 10,000 units/day
- **Cost**: Free for typical usage
- **Usage**: Low (video search only)

### Groq
- **Free Tier**: Available
- **Cost**: Free for Whisper transcription
- **Usage**: Medium (Viva interviews only)

### Total Monthly Cost
**$0** (all using free tiers)

## Viva Engine Requirements

The Viva (interview) feature requires:
1. ✅ `GEMINI_API_KEY` - For AI responses
2. ✅ `GROQ_API_KEY` - For speech-to-text (Whisper)
3. ✅ Edge-TTS - For text-to-speech (built-in, no API key)

**Note**: The error messages in `main.py` incorrectly state that OpenAI is needed for Viva. This is wrong - Viva uses Groq + Gemini.

## Files to Update

### 1. AdaptEd/backend/.env
Remove unused API keys

### 2. AdaptEd/backend/.env.example
Remove unused API keys

### 3. AdaptEd/backend/main.py
Remove OpenAI and Hugging Face checks

### 4. AdaptEd/backend/requirements.txt
Check if `openai` and `huggingface_hub` can be removed

## Verification

To verify no code uses these APIs:
```bash
# Search for OpenAI usage
grep -r "openai" AdaptEd/backend/*.py AdaptEd/backend/agents/*.py AdaptEd/backend/tools/*.py

# Search for Hugging Face usage
grep -r "huggingface\|transformers" AdaptEd/backend/*.py AdaptEd/backend/agents/*.py AdaptEd/backend/tools/*.py
```

Result: No actual usage found (only in venv libraries)

## Conclusion

**Safe to Remove**:
- ❌ OPENAI_API_KEY
- ❌ HUGGINGFACE_API_KEY

**Must Keep**:
- ✅ GEMINI_API_KEY (required)
- ✅ YOUTUBE_API_KEY (optional but recommended)
- ✅ GROQ_API_KEY (required for Viva)

This will simplify setup and reduce confusion for new developers.
