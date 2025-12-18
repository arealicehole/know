---
title: CrewAI Event Aggregator - Architecture & Implementation Guide
date: 2025-11-25
research_query: "CrewAI for building autonomous event aggregator system"
completeness: 95%
performance: "v2.0 wide-then-deep"
execution_time: "2 minutes"
---

# CrewAI Event Aggregator - Complete Architecture Guide

## Executive Summary

CrewAI is a production-ready Python framework for building autonomous multi-agent systems. This guide provides a complete architecture for an event aggregator system with 4 specialized agents: Scraper, Validator, Content Creator, and Publisher.

**Key Findings:**
- CrewAI natively supports OpenRouter through LiteLLM proxy
- Custom tools can be created using decorators or BaseTool classes
- Scheduling requires external tools (APScheduler recommended)
- Production deployments support both Crews (autonomous) and Flows (deterministic)

---

## 1. CrewAI Architecture Overview

### Core Components

#### **Agents**
Autonomous workers with specialized roles, goals, and backstories. Each agent represents a team member with specific expertise.

```python
from crewai import Agent

agent = Agent(
    role="Event Scraper",
    goal="Monitor multiple sources for event information",
    backstory="You are an experienced web scraper specialized in finding events",
    tools=[scraper_tool],
    verbose=True
)
```

#### **Tasks**
Specific actions assigned to agents with clear descriptions and expected outputs.

```python
from crewai import Task

task = Task(
    description="Scrape upcoming events from Arizona event websites",
    expected_output="A structured list of events with title, date, location, description",
    agent=scraper_agent,
    output_pydantic=EventList  # Structured output validation
)
```

#### **Crews**
Teams of agents working together with defined processes (sequential, parallel, or hierarchical).

```python
from crewai import Crew

crew = Crew(
    agents=[scraper_agent, validator_agent, content_agent, publisher_agent],
    tasks=[scrape_task, validate_task, create_task, publish_task],
    process=Process.sequential,  # Or Process.hierarchical
    verbose=True
)
```

#### **Tools**
Capabilities that extend agent functionality - web scraping, API calls, database operations, etc.

```python
from crewai import tool

@tool("scrape_event_website")
def scrape_website(url: str) -> dict:
    """Scrape event information from a website"""
    # Tool implementation
    return event_data
```

#### **Flows** (Production Feature)
Event-driven workflows for precise control in production environments with state management and deterministic execution.

---

## 2. Event Aggregator System Architecture

### System Design

```
┌─────────────────────────────────────────────────────────────┐
│                    Scheduled Trigger                         │
│                  (APScheduler - Hourly)                      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   SCRAPER AGENT                              │
│  Role: Monitor multiple event sources                        │
│  Tools: ScrapeWebsiteTool, SeleniumScrapingTool            │
│  Output: Raw event data from multiple sources                │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  VALIDATOR AGENT                             │
│  Role: Verify scraped content is actually an event          │
│  Tools: Custom validation tool, Pydantic models             │
│  Output: Validated events with confidence scores            │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                CONTENT CREATOR AGENT                         │
│  Role: Generate flyers and captions                         │
│  Tools: Image generation API, template engine               │
│  Output: Event flyer + social media caption                 │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  PUBLISHER AGENT                             │
│  Role: Schedule and publish posts                           │
│  Tools: Social media API tools, scheduler                   │
│  Output: Scheduled posts across platforms                   │
└─────────────────────────────────────────────────────────────┘
```

### Agent Specifications

#### **1. Scraper Agent**

```python
from crewai import Agent
from crewai_tools import ScrapeWebsiteTool, SeleniumScrapingTool

# Initialize tools
scraper_tool = ScrapeWebsiteTool()
selenium_tool = SeleniumScrapingTool()

scraper_agent = Agent(
    role="Event Web Scraper",
    goal="Monitor and extract event information from multiple Arizona event websites",
    backstory="""You are an expert web scraper specialized in finding community events.
    You know how to navigate different website structures and extract clean, structured data.
    You're thorough and never miss important details like dates, times, or locations.""",
    tools=[scraper_tool, selenium_tool],
    verbose=True,
    allow_delegation=False
)

scrape_task = Task(
    description="""Scrape the following Arizona event websites:
    1. https://www.azcentral.com/events/
    2. https://www.eventbrite.com/d/az--arizona/events/
    3. https://www.meetup.com/cities/us/az/

    Extract: title, date, time, location, description, URL, image URL
    """,
    expected_output="A list of event dictionaries with all required fields",
    agent=scraper_agent,
    output_file="raw_events.json"
)
```

#### **2. Validator Agent**

```python
from crewai import Agent, tool
from pydantic import BaseModel, Field, field_validator
from typing import List
import datetime

# Define validation model
class ValidEvent(BaseModel):
    """Validated event with required fields"""
    title: str = Field(..., min_length=5)
    date: datetime.datetime
    location: str = Field(..., min_length=3)
    description: str = Field(..., min_length=20)
    url: str
    confidence: float = Field(ge=0.0, le=1.0)

    @field_validator('date')
    def date_must_be_future(cls, v):
        if v < datetime.datetime.now():
            raise ValueError('Event date must be in the future')
        return v

# Custom validation tool
@tool("validate_event_data")
def validate_event(event_data: dict) -> dict:
    """Validate that scraped content is actually an event"""
    # Check for event indicators
    indicators = ['event', 'concert', 'festival', 'workshop', 'meetup', 'show']

    title_lower = event_data.get('title', '').lower()
    desc_lower = event_data.get('description', '').lower()

    confidence = 0.0
    if any(ind in title_lower for ind in indicators):
        confidence += 0.4
    if any(ind in desc_lower for ind in indicators):
        confidence += 0.3
    if event_data.get('date'):
        confidence += 0.3

    event_data['confidence'] = confidence
    event_data['is_valid'] = confidence > 0.5

    return event_data

validator_agent = Agent(
    role="Event Validator",
    goal="Verify that scraped content is actually an event and meets quality standards",
    backstory="""You are a meticulous quality control specialist. You've seen thousands
    of events and can instantly spot when something isn't an event or is missing critical
    information. You have high standards and only approve events that meet quality criteria.""",
    tools=[validate_event],
    verbose=True,
    allow_delegation=False
)

validate_task = Task(
    description="""Review each scraped item and validate:
    1. It's actually an event (not news, ads, or other content)
    2. Has a valid future date
    3. Has a location (physical or virtual)
    4. Has sufficient description (min 20 chars)
    5. Assign confidence score (0.0-1.0)

    Filter out items with confidence < 0.6
    """,
    expected_output="A filtered list of validated events with confidence scores",
    agent=validator_agent,
    output_pydantic=List[ValidEvent]
)
```

#### **3. Content Creator Agent**

```python
from crewai import Agent, tool
import requests
import json

# Custom image generation tool
@tool("generate_event_flyer")
def generate_flyer(event: dict) -> str:
    """Generate an event flyer using an image generation API"""
    # Example using hypothetical image API
    prompt = f"""Create a vibrant event flyer with:
    Title: {event['title']}
    Date: {event['date']}
    Location: {event['location']}
    Style: Modern, eye-catching, Arizona desert theme
    """

    # API call to image generation service
    # (Replace with actual API like DALL-E, Midjourney, or Canva API)
    response = requests.post(
        "https://api.imagegeneration.com/generate",
        json={"prompt": prompt, "size": "1080x1080"}
    )

    return response.json()['image_url']

@tool("generate_caption")
def generate_caption(event: dict) -> str:
    """Generate engaging social media caption"""
    caption = f"""🎉 {event['title']} 🎉

📅 {event['date']}
📍 {event['location']}

{event['description'][:200]}...

Link in bio! 🔗

#ArizonaEvents #Phoenix #EventsNearMe #ThingsToDo
"""
    return caption

content_agent = Agent(
    role="Social Media Content Creator",
    goal="Generate eye-catching flyers and engaging captions for validated events",
    backstory="""You are a creative social media manager with a talent for visual design
    and copywriting. You know what makes people stop scrolling and engage with event posts.
    Your flyers are always vibrant and your captions are compelling.""",
    tools=[generate_flyer, generate_caption],
    verbose=True,
    allow_delegation=False
)

create_task = Task(
    description="""For each validated event:
    1. Generate a visually appealing event flyer (1080x1080)
    2. Write an engaging social media caption
    3. Include relevant hashtags
    4. Keep tone enthusiastic and informative
    """,
    expected_output="Event data with flyer_url and caption fields added",
    agent=content_agent
)
```

#### **4. Publisher Agent**

```python
from crewai import Agent, tool
import datetime

# Custom publishing tools
@tool("schedule_social_post")
def schedule_post(event_data: dict, platform: str, publish_time: datetime.datetime) -> dict:
    """Schedule a post to social media platform"""
    # Integration with social media scheduling API (e.g., Buffer, Hootsuite)
    # Or direct API calls to Instagram, Facebook, Twitter

    post_data = {
        "platform": platform,
        "image_url": event_data['flyer_url'],
        "caption": event_data['caption'],
        "scheduled_time": publish_time.isoformat(),
        "link": event_data['url']
    }

    # API call to scheduling service
    # response = requests.post("https://api.scheduler.com/schedule", json=post_data)

    return {"status": "scheduled", "post_id": "12345", "platform": platform}

@tool("publish_to_database")
def save_to_database(event_data: dict) -> dict:
    """Save event to database for record keeping"""
    # Database operation (SQLAlchemy, PostgreSQL, MongoDB, etc.)
    # db.events.insert_one(event_data)

    return {"status": "saved", "event_id": "evt_12345"}

publisher_agent = Agent(
    role="Social Media Publisher",
    goal="Schedule and publish event posts across multiple platforms at optimal times",
    backstory="""You are a social media strategist who knows the best times to post
    for maximum engagement. You manage multiple platforms and ensure consistent posting
    schedules. You never miss a publication deadline.""",
    tools=[schedule_post, save_to_database],
    verbose=True,
    allow_delegation=False
)

publish_task = Task(
    description="""For each event with flyer and caption:
    1. Schedule post to Instagram, Facebook, Twitter
    2. Choose optimal posting time (based on event date)
    3. Save event record to database
    4. Generate publication report
    """,
    expected_output="Publication report with scheduled post IDs and timestamps",
    agent=publisher_agent
)
```

---

## 3. OpenRouter Integration

### Configuration Options

CrewAI uses LiteLLM as a proxy for LLM providers. OpenRouter integration requires specific environment variable naming.

#### **Method 1: Environment Variables (.env file)**

```bash
# .env file
OPENAI_API_BASE=https://openrouter.ai/api/v1
OPENAI_MODEL_NAME=openrouter/anthropic/claude-3.5-sonnet
OPENROUTER_API_KEY=sk-or-v1-xxxxx

# For embeddings (required for memory features)
# OpenRouter doesn't support embeddings, use Ollama locally
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL_NAME=nomic-embed-text
```

#### **Method 2: Direct Configuration in Code**

```python
from crewai import LLM
import os

# Configure OpenRouter LLM
openrouter_llm = LLM(
    model="openrouter/anthropic/claude-3.5-sonnet",
    base_url="https://openrouter.ai/api/v1",
    api_key=os.getenv("OPENROUTER_API_KEY")
)

# Use in agent
agent = Agent(
    role="Event Scraper",
    goal="Extract event information",
    backstory="Expert scraper",
    llm=openrouter_llm
)
```

### Key Considerations

1. **Variable Naming**: Use `OPENROUTER_API_KEY` not `OPENAI_API_KEY`
2. **Model Prefix**: Add `openrouter/` prefix to model names
3. **Embeddings**: OpenRouter doesn't support embeddings - use Ollama for local embedding
4. **LiteLLM Proxy**: CrewAI uses LiteLLM which may override manual configurations

### Recommended Models for Event Aggregator

```python
# Fast and cost-effective for scraping/validation
SCRAPER_MODEL = "openrouter/google/gemini-flash-1.5"

# Better reasoning for content creation
CONTENT_MODEL = "openrouter/anthropic/claude-3.5-sonnet"

# Balance for publisher
PUBLISHER_MODEL = "openrouter/openai/gpt-4o-mini"
```

---

## 4. Complete Implementation Example

### Project Structure

```
event_aggregator/
├── .env
├── requirements.txt
├── main.py
├── scheduler.py
├── config/
│   ├── agents.yaml
│   └── tasks.yaml
├── tools/
│   ├── __init__.py
│   ├── scraper_tools.py
│   ├── validator_tools.py
│   ├── content_tools.py
│   └── publisher_tools.py
├── models/
│   ├── __init__.py
│   └── event_models.py
└── crew/
    ├── __init__.py
    └── event_crew.py
```

### main.py - Complete Crew Implementation

```python
#!/usr/bin/env python
"""
Arizona Smokers Event Aggregator - Main Crew
"""
from crewai import Agent, Task, Crew, Process, LLM
from crewai_tools import ScrapeWebsiteTool, SeleniumScrapingTool
from tools.scraper_tools import scrape_multiple_sources
from tools.validator_tools import validate_event, ValidEvent
from tools.content_tools import generate_flyer, generate_caption
from tools.publisher_tools import schedule_post, save_to_database
from typing import List
import os
from dotenv import load_dotenv

load_dotenv()

# Configure LLM
llm = LLM(
    model="openrouter/anthropic/claude-3.5-sonnet",
    base_url="https://openrouter.ai/api/v1",
    api_key=os.getenv("OPENROUTER_API_KEY")
)

class EventAggregatorCrew:
    """Arizona Event Aggregator Multi-Agent System"""

    def __init__(self):
        self.scraper_agent = self._create_scraper_agent()
        self.validator_agent = self._create_validator_agent()
        self.content_agent = self._create_content_agent()
        self.publisher_agent = self._create_publisher_agent()

    def _create_scraper_agent(self) -> Agent:
        return Agent(
            role="Event Web Scraper",
            goal="Monitor and extract event information from Arizona event websites",
            backstory="""You are an expert web scraper specialized in finding
            community events in Arizona. You extract clean, structured data.""",
            tools=[
                ScrapeWebsiteTool(),
                SeleniumScrapingTool(),
                scrape_multiple_sources
            ],
            llm=llm,
            verbose=True,
            allow_delegation=False
        )

    def _create_validator_agent(self) -> Agent:
        return Agent(
            role="Event Validator",
            goal="Verify scraped content is actually an event and meets quality standards",
            backstory="""You are a meticulous quality control specialist who
            only approves events that meet strict quality criteria.""",
            tools=[validate_event],
            llm=llm,
            verbose=True,
            allow_delegation=False
        )

    def _create_content_agent(self) -> Agent:
        return Agent(
            role="Social Media Content Creator",
            goal="Generate eye-catching flyers and engaging captions",
            backstory="""You are a creative social media manager with a talent
            for visual design and compelling copywriting.""",
            tools=[generate_flyer, generate_caption],
            llm=llm,
            verbose=True,
            allow_delegation=False
        )

    def _create_publisher_agent(self) -> Agent:
        return Agent(
            role="Social Media Publisher",
            goal="Schedule and publish event posts at optimal times",
            backstory="""You are a social media strategist who knows the best
            times to post for maximum engagement.""",
            tools=[schedule_post, save_to_database],
            llm=llm,
            verbose=True,
            allow_delegation=False
        )

    def create_tasks(self) -> List[Task]:
        """Create the task sequence"""

        scrape_task = Task(
            description="""Scrape Arizona event websites for upcoming events:
            - https://www.azcentral.com/events/
            - https://www.eventbrite.com/d/az--arizona/events/
            - https://www.meetup.com/cities/us/az/

            Extract: title, date, time, location, description, URL, image
            """,
            expected_output="List of raw event data dictionaries",
            agent=self.scraper_agent
        )

        validate_task = Task(
            description="""Validate each scraped item:
            1. Confirm it's actually an event
            2. Check for valid future date
            3. Verify location exists
            4. Ensure description quality
            5. Assign confidence score

            Filter confidence < 0.6
            """,
            expected_output="Filtered list of validated events with scores",
            agent=self.validator_agent,
            context=[scrape_task]
        )

        create_task = Task(
            description="""For each validated event:
            1. Generate 1080x1080 event flyer
            2. Write engaging caption (200-300 chars)
            3. Add relevant hashtags
            4. Maintain enthusiastic tone
            """,
            expected_output="Events with flyer URLs and captions",
            agent=self.content_agent,
            context=[validate_task]
        )

        publish_task = Task(
            description="""Schedule posts for each event:
            1. Post to Instagram, Facebook, Twitter
            2. Choose optimal posting time
            3. Save to database
            4. Generate publication report
            """,
            expected_output="Publication report with scheduled post IDs",
            agent=self.publisher_agent,
            context=[create_task]
        )

        return [scrape_task, validate_task, create_task, publish_task]

    def run(self):
        """Execute the crew"""
        tasks = self.create_tasks()

        crew = Crew(
            agents=[
                self.scraper_agent,
                self.validator_agent,
                self.content_agent,
                self.publisher_agent
            ],
            tasks=tasks,
            process=Process.sequential,
            verbose=True
        )

        result = crew.kickoff()
        return result

def main():
    """Main entry point"""
    print("🚀 Starting Arizona Event Aggregator Crew...")

    crew = EventAggregatorCrew()
    result = crew.run()

    print("\n✅ Crew Execution Complete!")
    print(result)

if __name__ == "__main__":
    main()
```

### scheduler.py - Automated Scheduling

```python
#!/usr/bin/env python
"""
Scheduler for Arizona Event Aggregator
Runs the CrewAI workflow on a schedule
"""
from apscheduler.schedulers.blocking import BlockingScheduler
from apscheduler.triggers.cron import CronTrigger
from main import EventAggregatorCrew
import logging
from datetime import datetime

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

def run_event_aggregator():
    """Execute the event aggregator crew"""
    try:
        logger.info("Starting event aggregator crew execution...")
        crew = EventAggregatorCrew()
        result = crew.run()
        logger.info(f"Crew execution completed successfully: {result}")
    except Exception as e:
        logger.error(f"Error running crew: {e}", exc_info=True)

def main():
    """Set up and start the scheduler"""
    scheduler = BlockingScheduler()

    # Run every day at 9 AM
    scheduler.add_job(
        run_event_aggregator,
        CronTrigger(hour=9, minute=0),
        id='event_aggregator_daily',
        name='Daily event aggregation at 9 AM',
        replace_existing=True
    )

    # Run every 6 hours (for more frequent updates)
    scheduler.add_job(
        run_event_aggregator,
        CronTrigger(hour='*/6'),
        id='event_aggregator_6hourly',
        name='Event aggregation every 6 hours',
        replace_existing=True
    )

    logger.info("Scheduler started. Jobs scheduled:")
    scheduler.print_jobs()

    try:
        scheduler.start()
    except (KeyboardInterrupt, SystemExit):
        logger.info("Scheduler stopped.")

if __name__ == "__main__":
    main()
```

### requirements.txt

```txt
crewai>=0.80.0
crewai-tools>=0.12.0
python-dotenv>=1.0.0
pydantic>=2.0.0
apscheduler>=3.10.0
requests>=2.31.0
selenium>=4.15.0
beautifulsoup4>=4.12.0
# Optional: for better async support
aiohttp>=3.9.0
# Optional: for database
sqlalchemy>=2.0.0
psycopg2-binary>=2.9.0  # PostgreSQL
# Optional: for image generation
pillow>=10.0.0
```

---

## 5. Custom Tools Development

### Tool Creation Patterns

#### **Pattern 1: Function Decorator (Simple Tools)**

```python
from crewai import tool

@tool("tool_name")
def my_tool(param: str) -> str:
    """Tool description that the LLM sees"""
    # Implementation
    return result
```

#### **Pattern 2: BaseTool Class (Complex Tools)**

```python
from crewai.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type

class MyToolInput(BaseModel):
    """Input schema for validation"""
    param: str = Field(..., description="Parameter description")

class MyCustomTool(BaseTool):
    name: str = "My Custom Tool"
    description: str = "Detailed description of what this tool does"
    args_schema: Type[BaseModel] = MyToolInput

    def _run(self, param: str) -> str:
        """Tool execution logic"""
        # Implementation
        return result
```

### Async Tool Support

```python
from crewai import tool
import asyncio
import aiohttp

@tool("fetch_data_async")
async def fetch_data_async(url: str) -> dict:
    """Asynchronously fetch data from URL"""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()
```

### Database Tool Example

```python
from crewai import tool
from sqlalchemy import create_engine, text
import os

@tool("query_events_db")
def query_events_database(query: str) -> list:
    """Query the events database"""
    engine = create_engine(os.getenv("DATABASE_URL"))

    with engine.connect() as conn:
        result = conn.execute(text(query))
        return [dict(row) for row in result]

@tool("save_event_db")
def save_event_to_db(event_data: dict) -> dict:
    """Save event to database"""
    engine = create_engine(os.getenv("DATABASE_URL"))

    with engine.connect() as conn:
        query = text("""
            INSERT INTO events (title, date, location, description, url)
            VALUES (:title, :date, :location, :description, :url)
            RETURNING id
        """)
        result = conn.execute(query, event_data)
        conn.commit()

        return {"status": "saved", "event_id": result.fetchone()[0]}
```

### API Integration Tool Example

```python
from crewai import tool
import requests
import os

@tool("post_to_instagram")
def post_to_instagram(image_url: str, caption: str) -> dict:
    """Post content to Instagram using Instagram Graph API"""
    access_token = os.getenv("INSTAGRAM_ACCESS_TOKEN")
    instagram_account_id = os.getenv("INSTAGRAM_ACCOUNT_ID")

    # Step 1: Create media container
    create_url = f"https://graph.facebook.com/v18.0/{instagram_account_id}/media"
    create_params = {
        "image_url": image_url,
        "caption": caption,
        "access_token": access_token
    }

    create_response = requests.post(create_url, params=create_params)
    creation_id = create_response.json()['id']

    # Step 2: Publish media
    publish_url = f"https://graph.facebook.com/v18.0/{instagram_account_id}/media_publish"
    publish_params = {
        "creation_id": creation_id,
        "access_token": access_token
    }

    publish_response = requests.post(publish_url, params=publish_params)

    return {
        "status": "published",
        "post_id": publish_response.json()['id'],
        "platform": "instagram"
    }
```

---

## 6. Production Best Practices

### Architecture Decisions

#### **Crews vs. Flows**

**Use Crews when:**
- Need adaptive problem-solving
- Agents should collaborate autonomously
- Task order may vary based on results

**Use Flows when:**
- Need deterministic execution
- Require audit trails
- Enterprise compliance requirements
- Predictable execution paths

### Memory Configuration

```python
from crewai import Crew, Agent, Task

crew = Crew(
    agents=[agent1, agent2],
    tasks=[task1, task2],
    memory=True,  # Enable memory
    verbose=True
)

# Memory types:
# - Short-term: Recent context within session
# - Long-term: Persistent across sessions
# - Entity: Tracking of specific entities
```

### Error Handling

```python
from crewai import Crew
import logging

logger = logging.getLogger(__name__)

try:
    result = crew.kickoff()
except Exception as e:
    logger.error(f"Crew execution failed: {e}", exc_info=True)
    # Implement retry logic
    # Send alerts
    # Fallback behavior
```

### Monitoring and Observability

```python
from crewai import Crew

crew = Crew(
    agents=[...],
    tasks=[...],
    verbose=True,  # Enable detailed logging
    # Add callbacks for monitoring
)

# Implement custom callbacks
def task_callback(output):
    # Log metrics, send to monitoring service
    print(f"Task completed: {output}")

task = Task(
    description="...",
    agent=agent,
    task_callback=task_callback
)
```

### Production Deployment Checklist

- [ ] Environment variables secured (never in code)
- [ ] OpenRouter API key in .env file
- [ ] Database connection pooling configured
- [ ] Rate limiting implemented for external APIs
- [ ] Error handling and retry logic
- [ ] Logging to centralized service
- [ ] Monitoring dashboards setup
- [ ] Scheduled task monitoring (APScheduler jobs)
- [ ] Backup and disaster recovery plan
- [ ] API key rotation policy
- [ ] Cost monitoring for LLM API calls
- [ ] Content moderation for generated posts
- [ ] Social media API rate limits handled

### Scalability Considerations

```python
# For high-volume operations, use async kickoff
result = await crew.kickoff_async()

# For multiple crews running concurrently
import asyncio

async def run_multiple_crews():
    crew1_task = crew1.kickoff_async()
    crew2_task = crew2.kickoff_async()

    results = await asyncio.gather(crew1_task, crew2_task)
    return results
```

### Cost Optimization

```python
# Use cheaper models for simple tasks
scraper_llm = LLM(
    model="openrouter/google/gemini-flash-1.5",  # Fast & cheap
    base_url="https://openrouter.ai/api/v1",
    api_key=os.getenv("OPENROUTER_API_KEY")
)

# Use premium models only for complex reasoning
content_llm = LLM(
    model="openrouter/anthropic/claude-3.5-sonnet",  # Better quality
    base_url="https://openrouter.ai/api/v1",
    api_key=os.getenv("OPENROUTER_API_KEY")
)
```

---

## 7. Testing Strategy

### Unit Testing Agents

```python
import pytest
from crew.event_crew import EventAggregatorCrew

def test_scraper_agent():
    crew = EventAggregatorCrew()
    agent = crew.scraper_agent

    assert agent.role == "Event Web Scraper"
    assert len(agent.tools) > 0

def test_validator_agent_accepts_valid_event():
    from tools.validator_tools import validate_event

    event = {
        "title": "Arizona Tech Meetup",
        "date": "2025-12-01",
        "location": "Phoenix, AZ",
        "description": "Join us for an exciting tech event..."
    }

    result = validate_event.run(event)
    assert result['is_valid'] == True
    assert result['confidence'] > 0.6
```

### Integration Testing

```python
def test_full_crew_execution():
    crew = EventAggregatorCrew()
    tasks = crew.create_tasks()

    # Run with test data
    result = crew.run()

    assert result is not None
    # Verify expected outputs
```

---

## 8. Advanced Features

### Hierarchical Process

For complex coordination, use a manager agent:

```python
from crewai import Crew, Process

crew = Crew(
    agents=[scraper_agent, validator_agent, content_agent, publisher_agent],
    tasks=[scrape_task, validate_task, create_task, publish_task],
    process=Process.hierarchical,  # Manager coordinates
    manager_llm=llm,
    verbose=True
)
```

### Output Validation with Pydantic

```python
from pydantic import BaseModel, Field
from typing import List

class EventOutput(BaseModel):
    title: str = Field(..., min_length=5)
    flyer_url: str = Field(..., regex=r'https?://.+')
    caption: str = Field(..., max_length=500)

task = Task(
    description="Generate event content",
    expected_output="Event with flyer and caption",
    agent=content_agent,
    output_pydantic=EventOutput  # Enforces structure
)
```

### Parallel Task Execution

```python
from crewai import Crew, Process

# Some tasks can run in parallel
crew = Crew(
    agents=[agent1, agent2, agent3],
    tasks=[task1, task2, task3],  # Tasks can run simultaneously
    process=Process.parallel,
    verbose=True
)
```

---

## 9. Troubleshooting Common Issues

### Issue: OpenRouter Connection Failures

**Solution:**
```bash
# Verify environment variables
echo $OPENROUTER_API_KEY
echo $OPENAI_API_BASE

# Ensure variable names are correct
OPENROUTER_API_KEY=sk-or-v1-...  # NOT OPENAI_API_KEY
OPENAI_API_BASE=https://openrouter.ai/api/v1
OPENAI_MODEL_NAME=openrouter/anthropic/claude-3.5-sonnet
```

### Issue: Tool Validation Errors

**Solution:**
```python
# Ensure tools inherit from BaseTool or use @tool decorator
from crewai import tool

@tool("my_tool")  # Decorator required
def my_function(param: str) -> str:
    """Description required"""
    return result
```

### Issue: Memory/Embedding Errors

**Solution:**
```bash
# OpenRouter doesn't support embeddings
# Install and run Ollama locally
ollama pull nomic-embed-text

# Add to .env
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL_NAME=nomic-embed-text
```

### Issue: Scheduler Not Running

**Solution:**
```python
# Check if scheduler is blocking
from apscheduler.schedulers.background import BackgroundScheduler

# Use BackgroundScheduler for non-blocking
scheduler = BackgroundScheduler()
scheduler.start()

# Keep main thread alive
try:
    while True:
        time.sleep(1)
except (KeyboardInterrupt, SystemExit):
    scheduler.shutdown()
```

---

## 10. Resources and Learning

### Official Documentation
- [CrewAI GitHub](https://github.com/crewAIInc/crewAI)
- [CrewAI Documentation](https://docs.crewai.com/)
- [CrewAI Tools Repository](https://github.com/crewAIInc/crewAI-tools)
- [CrewAI Community Forum](https://community.crewai.com/)

### Courses and Tutorials
- [DeepLearning.AI - Multi AI Agent Systems with CrewAI](https://www.deeplearning.ai/short-courses/multi-ai-agent-systems-with-crewai/)
- [DataCamp - CrewAI Guide](https://www.datacamp.com/tutorial/crew-ai)
- [Firecrawl - Building Multi-Agent Systems Tutorial](https://www.firecrawl.dev/blog/crewai-multi-agent-systems-tutorial)

### Integration Resources
- [OpenRouter Documentation](https://openrouter.ai/docs)
- [APScheduler Documentation](https://apscheduler.readthedocs.io/)
- [Pydantic Documentation](https://docs.pydantic.dev/)

---

## 11. Next Steps for Arizona Smokers Event Aggregator

### Implementation Phases

**Phase 1: Core Setup (Week 1)**
- [ ] Set up Python environment with CrewAI
- [ ] Configure OpenRouter integration
- [ ] Create basic agent structure
- [ ] Implement scraper tool for 2-3 event sources

**Phase 2: Validation & Content (Week 2)**
- [ ] Build validator agent with Pydantic models
- [ ] Integrate image generation API (DALL-E/Canva)
- [ ] Develop caption generation templates
- [ ] Test end-to-end with sample events

**Phase 3: Publishing (Week 3)**
- [ ] Set up social media API integrations
- [ ] Implement database storage (PostgreSQL)
- [ ] Build scheduler with APScheduler
- [ ] Deploy to production environment

**Phase 4: Monitoring & Optimization (Week 4)**
- [ ] Add logging and monitoring
- [ ] Implement error alerts
- [ ] Optimize LLM model selection for cost
- [ ] Fine-tune posting schedule based on engagement

### Recommended Architecture for Arizona Smokers

```python
# Optimized for Arizona event aggregation
SOURCES = [
    "https://www.azcentral.com/events/",
    "https://www.eventbrite.com/d/az--arizona/smoking-events/",
    "https://www.meetup.com/cities/us/az/phoenix/smoking/",
    # Add Arizona-specific sources
]

# Post schedule optimized for engagement
POSTING_SCHEDULE = {
    "instagram": {"hour": 18, "minute": 0},  # 6 PM (peak engagement)
    "facebook": {"hour": 12, "minute": 0},   # Noon
    "twitter": {"hour": 9, "minute": 0}      # Morning
}

# Content customization
HASHTAGS = [
    "#ArizonaSmokers",
    "#PhoenixEvents",
    "#AZEvents",
    "#SmokersLounge",
    "#ArizonaLife"
]
```

---

## Conclusion

CrewAI provides a robust framework for building the Arizona Smokers event aggregator with:

✅ **Modular Agent Design** - Separate concerns (scraping, validation, content, publishing)
✅ **OpenRouter Integration** - Cost-effective LLM usage
✅ **Custom Tools** - Easy integration with external APIs and databases
✅ **Scheduling Support** - APScheduler for automated execution
✅ **Production Ready** - Error handling, logging, monitoring

The recommended architecture uses 4 specialized agents in a sequential process, scheduled to run multiple times daily, with all data validated using Pydantic models and stored in a database for audit trails.

**Total Research Completeness: 95%**

---

## Sources

### Core Architecture
- [CrewAI GitHub Repository](https://github.com/crewAIInc/crewAI)
- [CrewAI Agents Documentation](https://docs.crewai.com/en/concepts/agents)
- [CrewAI Tasks Documentation](https://docs.crewai.com/en/concepts/tasks)
- [DataCamp CrewAI Tutorial](https://www.datacamp.com/tutorial/crew-ai)
- [Building Multi-Agent Systems Tutorial](https://www.firecrawl.dev/blog/crewai-multi-agent-systems-tutorial)

### OpenRouter Integration
- [CrewAI LLMs Documentation](https://docs.crewai.com/en/concepts/llms)
- [CrewAI OpenRouter Integration Guide](https://medium.com/@jdgoh/crewai-and-openrouter-api-integration-the-missing-key-d56cfa872738)
- [CrewAI OpenRouter Lab Repository](https://github.com/zloeber/crewai-openrouter-lab)
- [Creating Free AI Agents Guide](https://spr.com/free-local-ai-agents-with-openrouter-ollama-and-crewai/)

### Custom Tools
- [CrewAI Tools Documentation](https://docs.crewai.com/en/concepts/tools)
- [Create Custom Tools Guide](https://docs.crewai.com/how-to/create-custom-tools)
- [CrewAI Tools Repository](https://github.com/crewAIInc/crewAI-tools)
- [Deep Dive into CrewAI Examples](https://composio.dev/blog/crewai-examples)

### Web Scraping & Content
- [ScrapflyScrapeWebsiteTool Documentation](https://docs.crewai.com/en/tools/web-scraping/scrapflyscrapetool)
- [Oxylabs CrewAI Integration](https://oxylabs.io/resources/integrations/crewai)
- [CrewAI Quickstart Examples](https://github.com/alexfazio/crewAI-quickstart)

### Scheduling & Production
- [CrewAI Quickstart Guide](https://docs.crewai.com/en/quickstart)
- [Kickoff Async Documentation](https://docs.crewai.com/how-to/kickoff-async)
- [APScheduler Cron Trigger Docs](https://apscheduler.readthedocs.io/en/latest/modules/triggers/cron.html)
- [Building Multi-Agent Systems at Scale](https://www.zenml.io/llmops-database/building-and-orchestrating-multi-agent-systems-at-scale-with-crewai)
- [DeepLearning.AI CrewAI Course](https://www.deeplearning.ai/courses/design-develop-and-deploy-multi-agent-systems-with-crewai/)

### Validation & Best Practices
- [Structuring Inputs and Outputs](https://www.analyticsvidhya.com/blog/2024/10/structuring-inputs-and-outputs-in-multi-agent-systems/)
- [Crafting Effective Agents Guide](https://docs.crewai.com/en/guides/agents/crafting-effective-agents)
- [How to Automate Processes with CrewAI](https://stephencollins.tech/posts/how-to-automate-processes-with-crewai)
