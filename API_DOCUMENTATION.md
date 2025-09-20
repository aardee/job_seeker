# Job Seeker AI - API Documentation

## Overview

The Job Seeker AI is an intelligent system that processes resumes, extracts structured data, and assists with job searching using AI agents. The system leverages OpenAI's GPT models to analyze resumes and create structured profiles for job matching.

## Table of Contents

1. [Data Models](#data-models)
2. [Core Functions](#core-functions)
3. [Agent System](#agent-system)
4. [Usage Examples](#usage-examples)
5. [Configuration](#configuration)
6. [Dependencies](#dependencies)

## Data Models

The system uses Pydantic models to ensure data validation and type safety.

### ResumeModel

Main model representing a candidate's complete resume profile.

```python
class ResumeModel(BaseModel):
    name: str                           # Full name of the candidate
    email: EmailStr                     # Email address (validated)
    phone: Optional[str]                # Phone number
    linkedin: Optional[str]             # LinkedIn profile URL
    github: Optional[str]               # GitHub profile URL
    skills: List[str]                   # List of technical and soft skills
    experiences: List[Experience]       # Professional experience history
    education: List[Education]          # Educational qualifications
    projects: List[Project]             # Personal/academic projects
    certifications: List[Certification] # Professional certifications
    publications: List[Publication]     # Research publications
    patents: List[Patent]               # Patent holdings
    summary: Optional[str]              # 2-3 sentence professional summary
    keywords: List[str]                 # Job search keywords
    target_company_profile: Optional[str] # Target company sector/profile
```

### Experience

Represents a single work experience entry.

```python
class Experience(BaseModel):
    title: str                          # Job title or position held
    company: Optional[str]              # Company or organization name
    start_date: Optional[str]           # Start date (flexible format)
    end_date: Optional[str]             # End date (flexible format)
    description: Optional[str]          # Role responsibilities and achievements
```

### Education

Represents educational qualifications.

```python
class Education(BaseModel):
    degree: str                         # Degree or qualification name
    institution: Optional[str]          # Educational institution name
    start_date: Optional[str]           # Start date
    end_date: Optional[str]             # End date
    description: Optional[str]          # Coursework or achievements description
```

### Project

Represents personal or academic projects.

```python
class Project(BaseModel):
    name: str                           # Project name
    description: Optional[str]          # Project description
    link: Optional[str]                 # URL or link to project
```

### Certification

Represents professional certifications.

```python
class Certification(BaseModel):
    name: str                           # Certification name
    issuer: Optional[str]               # Issuing organization
    date: Optional[str]                 # Date obtained
```

### Publication

Represents research publications.

```python
class Publication(BaseModel):
    title: str                          # Publication title
    publisher: Optional[str]            # Publisher or journal name
    date: Optional[str]                 # Publication date
    link: Optional[str]                 # URL or link to publication
```

### Patent

Represents patent holdings.

```python
class Patent(BaseModel):
    title: str                          # Patent title
    number: Optional[str]               # Patent number
    date: Optional[str]                 # Date granted
    description: Optional[str]          # Patent description
```

### JobPosting

Represents a job posting found during job search.

```python
class JobPosting(BaseModel):
    title: str                          # Job title
    company: Optional[str]              # Company name
    location: Optional[str]             # Job location
    description: Optional[str]          # Job description
    link: Optional[str]                 # Job posting URL
    apply_link: Optional[str]           # Application URL
    hiring_manager_name: Optional[str]  # Hiring manager name
    keywords: List[str]                 # Matching keywords
```

### JobSearchPlan

Represents a job search strategy with URLs.

```python
class JobSearchPlan(BaseModel):
    job_search_urls: List[str]          # List of job search site URLs
```

## Core Functions

### Resume Processing

#### `profile_resume(resume_text: str) -> str`

Processes raw resume text and returns structured JSON data conforming to the ResumeModel schema.

**Parameters:**
- `resume_text` (str): Plain text extracted from PDF resume

**Returns:**
- JSON string containing structured resume data

**Usage:**
```python
resume_text = "John Doe\nemail@example.com\n..."
profiled_data = profile_resume(resume_text)
resume_json = json.loads(profiled_data)
```

### Prompt Generation

#### `system_prompt_4_resume_profiler() -> str`

Generates the system prompt for the ResumeProfiler AI agent.

**Returns:**
- System prompt string with instructions and JSON schema

#### `user_prompt_4_resume_profiler(resume_text: str) -> str`

Generates the user prompt containing the resume text for analysis.

**Parameters:**
- `resume_text` (str): Resume text to be analyzed

**Returns:**
- Formatted user prompt with resume content

### Agent Creation

#### `get_job_search_panner_agent(profiled_resume_data_json: dict) -> Agent`

Creates a job search planning agent that generates job search URLs based on resume profile.

**Parameters:**
- `profiled_resume_data_json` (dict): Structured resume data from ResumeModel

**Returns:**
- Configured Agent instance for job search planning

## Agent System

The system uses OpenAI Agents framework for intelligent job searching.

### JobSearchPlannerAgent

Analyzes candidate profile and creates strategic job search URLs.

**Capabilities:**
- Extracts relevant keywords from resume
- Identifies target company profiles
- Generates job search site URLs
- Optimizes search strategy based on candidate skills

### Suggested Job Search Tools

The system recommends integration with the following job search APIs:

1. **SerpAPI**: Google Search-based job discovery
2. **LinkedIn Job Search API**: Professional network job opportunities
3. **Indeed Job Search API**: Aggregated job postings
4. **Glassdoor Job Search API**: Jobs with company reviews

## Usage Examples

### Complete Resume Processing Workflow

```python
from pypdf import PdfReader
import json

# 1. Extract resume text from PDF
reader = PdfReader("resume.pdf")
resume_text = ""
for page in reader.pages:
    text = page.extract_text()
    if text:
        resume_text += text

# 2. Profile the resume
profiled_data = profile_resume(resume_text)
resume_json = json.loads(profiled_data)

# 3. Create job search planner agent
job_planner = get_job_search_panner_agent(resume_json)

# 4. Generate job search plan
search_plan = job_planner.run("Create job search URLs for this candidate")
```

### Direct Model Usage

```python
from pydantic import ValidationError

try:
    # Parse and validate resume data
    resume = ResumeModel.model_validate(resume_json)
    
    # Access structured data
    print(f"Candidate: {resume.name}")
    print(f"Skills: {', '.join(resume.skills)}")
    print(f"Summary: {resume.summary}")
    
except ValidationError as e:
    print(f"Validation error: {e}")
```

### Job Search Integration

```python
# Example job search workflow
def search_jobs(resume_data):
    # Extract search parameters
    keywords = resume_data.get("keywords", [])
    summary = resume_data.get("summary", "")
    
    # Create search queries
    search_queries = []
    for keyword in keywords[:5]:  # Limit to top 5 keywords
        query = f"jobs {keyword} bioinformatics machine learning"
        search_queries.append(query)
    
    return search_queries

# Usage
search_queries = search_jobs(resume_json)
print("Generated search queries:", search_queries)
```

## Configuration

### Environment Variables

The system requires the following environment variables:

```bash
OPENAI_API_KEY=your_openai_api_key_here
```

### Model Configuration

```python
MODEL = "gpt-4-mini"  # Default model for resume processing
```

### OpenAI Client Setup

```python
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv(override=True)
client = OpenAI()  # Uses OPENAI_API_KEY from environment
```

## Dependencies

### Core Dependencies

- **openai**: OpenAI API client for GPT models
- **pydantic[email]**: Data validation with email support
- **pypdf**: PDF text extraction
- **python-dotenv**: Environment variable management
- **openai-agents**: Agent framework for intelligent workflows

### Development Dependencies

- **ipykernel**: Jupyter notebook support

### Optional Integrations

- **requests**: HTTP client for API integrations
- **langchain-openai**: Alternative LLM framework
- **gradio**: Web interface development
- **streamlit**: Dashboard creation

## Error Handling

### Common Validation Errors

```python
try:
    resume = ResumeModel.model_validate(data)
except ValidationError as e:
    for error in e.errors():
        print(f"Field: {error['loc']}, Error: {error['msg']}")
```

### API Error Handling

```python
try:
    response = profile_resume(resume_text)
    data = json.loads(response)
except json.JSONDecodeError:
    print("Invalid JSON response from AI model")
except Exception as e:
    print(f"API error: {e}")
```

## Best Practices

1. **Resume Text Quality**: Ensure PDF text extraction is clean and readable
2. **Validation**: Always validate data against Pydantic models
3. **Error Handling**: Implement proper exception handling for API calls
4. **Rate Limiting**: Be mindful of OpenAI API rate limits
5. **Data Privacy**: Handle resume data securely and in compliance with privacy regulations

## API Versioning

Current API version: **v0.1.0**

The API follows semantic versioning. Breaking changes will increment the major version number.

## Support

For issues and feature requests, please refer to the project repository or contact the development team.
