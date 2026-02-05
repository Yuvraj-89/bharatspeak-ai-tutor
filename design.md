# BharatSpeak - AI English Communication Tutor
## System Design Document

## Architecture Overview

### High-Level Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Streamlit     │    │   Python        │    │   External      │
│   Frontend      │◄──►│   Backend       │◄──►│   Services      │
│                 │    │                 │    │                 │
│ • Voice I/O     │    │ • AI Tutor      │    │ • OpenAI API    │
│ • Chat UI       │    │ • Session Mgmt  │    │ • Speech APIs   │
│ • Progress View │    │ • Data Storage  │    │ • TTS Service   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Component Architecture

#### Frontend Layer (Streamlit)
- **Main App**: Core Streamlit application with session management
- **Voice Handler**: Browser-based speech recognition and audio recording
- **Chat Interface**: Real-time conversation display and input
- **Progress Dashboard**: Visual feedback and statistics
- **Scenario Selector**: Role-play scenario selection and setup

#### Backend Layer (Python)
- **AI Tutor Engine**: Core conversation logic and response generation
- **Speech Processor**: Audio processing and transcription handling
- **Feedback Generator**: Grammar, pronunciation, and fluency analysis
- **Session Manager**: User state and conversation history management
- **Data Layer**: Simple SQLite database for progress tracking

#### External Services
- **LLM API**: Gemini  or equivalent for conversation generation
- **Speech-to-Text**: Web Speech API or Google Speech-to-Text
- **Text-to-Speech**: Browser native or Google TTS
- **Grammar Check**: LanguageTool API or built-in rules

## System Components

### 1. AI Tutor Engine

#### Core Behavior Prompting
```python
TUTOR_SYSTEM_PROMPT = """
You are BharatSpeak, an AI English communication tutor for Indian learners.

PERSONALITY:
- Encouraging and patient, like a friendly teacher
- Understanding of Indian cultural context
- Supportive of gradual improvement
- Uses simple, clear explanations

COMMUNICATION STYLE:
- Speak naturally but clearly
- Use Hinglish explanations when concepts are complex
- Provide specific, actionable feedback
- Celebrate small improvements
- Ask follow-up questions to encourage conversation

FEEDBACK APPROACH:
- Point out 1-2 key improvements per response
- Explain grammar rules simply
- Suggest better word choices
- Encourage natural expression over perfect grammar
- Provide Hindi translations for difficult words

SCENARIO ADAPTATION:
- Match formality level to scenario (casual/professional)
- Provide context-appropriate vocabulary
- Guide users through realistic conversations
- Offer alternative responses for different situations
"""
```

#### Response Generation Logic
1. **Input Processing**: Analyze user input for grammar, vocabulary, context
2. **Context Awareness**: Maintain scenario state and conversation flow
3. **Feedback Generation**: Create specific, helpful corrections and suggestions
4. **Response Crafting**: Generate natural, encouraging AI responses
5. **Progress Tracking**: Update user metrics and learning insights

### 2. Scenario Management System

#### Scenario Structure
```python
class Scenario:
    def __init__(self):
        self.name = ""           # "Job Interview"
        self.context = ""        # Background setting
        self.difficulty = ""     # Beginner/Intermediate/Advanced
        self.vocabulary = []     # Key words for this scenario
        self.sample_questions = [] # AI questions to ask
        self.success_criteria = {} # What makes a good conversation
```

#### Core Scenarios

**1. Job Interview Practice**
- Context: Technical/HR interview simulation
- Focus: Professional communication, confidence building
- Key phrases: "Tell me about yourself", "Why should we hire you?"
- Vocabulary: Professional terms, company-related words
- Success metrics: Clarity, confidence, relevant examples

**2. Workplace Conversations**
- Context: Office discussions, meetings, presentations
- Focus: Collaborative communication, asking questions
- Key phrases: "I have a suggestion", "Could you clarify?"
- Vocabulary: Business terms, project management
- Success metrics: Participation, clarity, professionalism

**3. Daily Life Interactions**
- Context: Shopping, restaurant, customer service
- Focus: Practical communication, problem-solving
- Key phrases: "I would like to...", "Could you help me?"
- Vocabulary: Common daily activities, polite expressions
- Success metrics: Task completion, politeness, clarity

### 3. Speech Processing Pipeline

#### Voice Input Flow
```
User Speech → Browser Speech API → Text Transcription → 
Grammar Analysis → AI Processing → Response Generation → 
Text-to-Speech → Audio Output
```

#### Implementation Strategy
```python
class SpeechProcessor:
    def process_audio_input(self, audio_data):
        # 1. Convert speech to text
        transcript = self.speech_to_text(audio_data)
        
        # 2. Analyze for common mistakes
        analysis = self.analyze_speech_patterns(transcript)
        
        # 3. Generate feedback
        feedback = self.generate_pronunciation_feedback(analysis)
        
        return transcript, feedback
    
    def generate_ai_speech(self, text_response):
        # Convert AI response to natural speech
        return self.text_to_speech(text_response)
```

### 4. Feedback System Design

#### Multi-Dimensional Feedback
```python
class FeedbackGenerator:
    def analyze_response(self, user_input, scenario_context):
        return {
            'grammar': self.check_grammar(user_input),
            'vocabulary': self.suggest_vocabulary(user_input),
            'fluency': self.rate_fluency(user_input),
            'context_appropriateness': self.check_context(user_input, scenario_context),
            'pronunciation': self.analyze_pronunciation(user_input),
            'confidence_indicators': self.assess_confidence(user_input)
        }
```

#### Feedback Delivery Strategy
- **Immediate**: Quick corrections during conversation
- **Contextual**: Explanations relevant to current scenario
- **Progressive**: Building on previous feedback
- **Encouraging**: Always highlight improvements
- **Bilingual**: Hindi explanations for complex concepts

### 5. Progress Tracking System

#### Metrics Collection
```python
class ProgressTracker:
    def track_session(self, user_id, session_data):
        metrics = {
            'session_duration': session_data['duration'],
            'words_spoken': len(session_data['transcript'].split()),
            'grammar_score': session_data['grammar_accuracy'],
            'fluency_score': session_data['fluency_rating'],
            'scenario_completion': session_data['scenario_progress'],
            'confidence_indicators': session_data['confidence_metrics']
        }
        self.save_metrics(user_id, metrics)
```

#### Progress Visualization
- **Confidence Trend**: Line chart showing confidence growth over time
- **Grammar Improvement**: Bar chart of common mistake reduction
- **Vocabulary Expansion**: Word count and complexity metrics
- **Scenario Mastery**: Progress bars for different scenario types
- **Speaking Time**: Total practice time and session frequency

## Data Models

### Session Data Structure
```python
{
    "session_id": "uuid",
    "user_id": "temp_user", # For prototype
    "scenario": "job_interview",
    "start_time": "timestamp",
    "conversation_history": [
        {
            "speaker": "user|ai",
            "message": "text",
            "timestamp": "timestamp",
            "feedback": {...}
        }
    ],
    "session_metrics": {
        "total_words": 150,
        "grammar_score": 7.5,
        "fluency_score": 6.8,
        "confidence_score": 7.2
    }
}
```

### User Progress Schema
```sql
CREATE TABLE user_progress (
    id INTEGER PRIMARY KEY,
    user_id TEXT,
    session_date DATE,
    scenario_type TEXT,
    duration_minutes INTEGER,
    words_spoken INTEGER,
    grammar_score REAL,
    fluency_score REAL,
    confidence_score REAL,
    key_improvements TEXT
);
```

## Learning Flow Design

### User Journey
```
1. Welcome & Quick Assessment
   ↓
2. Scenario Selection
   ↓
3. Vocabulary Preview
   ↓
4. Interactive Conversation
   ↓
5. Real-time Feedback
   ↓
6. Session Summary
   ↓
7. Progress Review
```

### Adaptive Learning Logic
```python
def adapt_difficulty(user_performance):
    if user_performance['confidence'] > 8 and user_performance['grammar'] > 7:
        return "increase_difficulty"
    elif user_performance['confidence'] < 5 or user_performance['grammar'] < 4:
        return "decrease_difficulty"
    else:
        return "maintain_level"
```

## Technical Implementation

### Streamlit App Structure
```
bharatspeak/
├── app.py                 # Main Streamlit application
├── components/
│   ├── voice_handler.py   # Speech input/output
│   ├── chat_interface.py  # Conversation UI
│   ├── progress_view.py   # Progress dashboard
│   └── scenario_selector.py
├── core/
│   ├── ai_tutor.py       # AI conversation engine
│   ├── feedback_engine.py # Feedback generation
│   ├── speech_processor.py
│   └── session_manager.py
├── data/
│   ├── scenarios.json    # Scenario definitions
│   ├── vocabulary.json   # Word lists
│   └── database.db       # SQLite database
└── utils/
    ├── prompts.py        # AI prompting templates
    └── helpers.py        # Utility functions
```

### Key Implementation Decisions

#### 1. Streamlit for Rapid Prototyping
- Built-in session state management
- Easy voice integration with JavaScript components
- Quick UI development and iteration
- Good for demo presentations

#### 2. LLM Integration Strategy
```python
class AITutor:
    def __init__(self):
        self.client = OpenAI(api_key=os.getenv('OPENAI_API_KEY'))
        self.conversation_history = []
    
    def generate_response(self, user_input, scenario_context):
        messages = [
            {"role": "system", "content": TUTOR_SYSTEM_PROMPT},
            {"role": "system", "content": f"Current scenario: {scenario_context}"},
            *self.conversation_history,
            {"role": "user", "content": user_input}
        ]
        
        response = self.client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=messages,
            temperature=0.7,
            max_tokens=200
        )
        
        return response.choices[0].message.content
```

#### 3. Voice Processing Approach
- Primary: Browser Web Speech API for simplicity
- Fallback: Text input when voice fails
- Audio recording for pronunciation analysis
- Real-time transcription display

#### 4. Hinglish Support Implementation
```python
def provide_hinglish_explanation(complex_concept):
    hinglish_prompts = {
        "grammar_rule": "Isko Hindi mein samjhate hain: {explanation}",
        "vocabulary": "Ye word ka matlab hai: {meaning}",
        "pronunciation": "Aise bolte hain: {phonetic}"
    }
    return hinglish_prompts[concept_type].format(explanation=explanation)
```

## Demo Flow Design

### 5-Minute Demo Script
1. **Welcome (30s)**: Quick intro, scenario selection
2. **Vocabulary Preview (30s)**: Show key words for chosen scenario
3. **Conversation Practice (3m)**: Interactive dialogue with real-time feedback
4. **Progress Review (1m)**: Show improvements and suggestions

### Demo Scenarios
- **Job Interview**: "Tell me about your strengths"
- **Workplace**: "Presenting an idea in a team meeting"
- **Daily Life**: "Ordering food at a restaurant"

## Deployment Strategy

### Local Development
```bash
pip install streamlit openai speech_recognition
streamlit run app.py
```

### Demo Deployment
- Streamlit Cloud for easy sharing
- Environment variables for API keys
- Simple SQLite database (no external DB needed)
- Responsive design for mobile demos

## Success Metrics for Prototype

### Technical Success
- [ ] End-to-end conversation flow working
- [ ] Voice input/output functional
- [ ] Real-time feedback generation
- [ ] Progress tracking operational
- [ ] 3+ scenarios implemented

### User Experience Success
- [ ] Intuitive interface requiring minimal explanation
- [ ] Natural AI responses with helpful feedback
- [ ] Smooth voice interaction experience
- [ ] Clear progress visualization
- [ ] Effective Hinglish explanations

### Demo Readiness
- [ ] 5-minute demo script prepared
- [ ] Reliable performance under demo conditions
- [ ] Backup plans for technical issues
- [ ] Clear value proposition demonstration
- [ ] Engaging user interaction examples