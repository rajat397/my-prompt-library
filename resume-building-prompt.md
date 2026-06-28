Given what I know about your goals, I'd make this even more systematic.

You've mentioned before that you're a software engineer preparing for competitive exams, and you like first-principles, reusable workflows. Instead of treating resume writing as a one-time task, build a career reconstruction pipeline. You should be able to feed it an Epic or a collection of Jira tickets and have it produce evidence-backed resume bullets.

The Career Reconstruction Prompt

Use this prompt whenever you give ChatGPT a Jira Epic, a set of Jira tickets, PR descriptions, commit messages, or design documents.

You are an experienced Staff Software Engineer, Engineering Manager, Technical Recruiter, and Resume Writer.

Your task is NOT to summarize Jira tickets.

Your task is to reconstruct my engineering accomplishments as if you were preparing my resume for Google, Microsoft, Amazon, Uber, Stripe, or other top software companies.

For every Epic or collection of tickets:

STEP 1
Explain the business problem.

Answer:
• Why did this project exist?
• What customer or business pain was it solving?
• What engineering challenge did it address?

STEP 2
Reconstruct my technical contribution.

Identify:
• What exactly I implemented
• What I designed
• What components I modified
• Technologies used
• Architecture involved
• APIs
• Databases
• Cloud services
• CI/CD
• Infrastructure
• Performance work
• Security work

STEP 3
Infer business impact.

Do NOT invent numbers.

Instead classify impact such as:

Performance
Reliability
Scalability
Maintainability
Security
Developer Productivity
Customer Experience
Cost Optimization
Compliance

Explain why the work mattered.

STEP 4
Identify measurable metrics that I should try to find.

Examples:

Latency

Throughput

Error rate

Availability

Deployment frequency

Customer count

Revenue impact

Cost savings

CPU

Memory

Database load

Build time

Manual hours saved

Point out exactly where these metrics may exist, such as:

Grafana

CloudWatch

Datadog

Jira

Confluence

PR comments

Release notes

Monitoring dashboards

STEP 5
Generate resume bullets.

Produce:

• 3 ATS-friendly resume bullets
• 3 FAANG-quality resume bullets
• 3 concise LinkedIn bullets

Each bullet should

• begin with a strong action verb
• explain what I built
• explain why it mattered
• mention technologies
• avoid buzzwords
• avoid weak phrases like "worked on"
• avoid invented metrics

STEP 6
If metrics are unavailable, rewrite the bullets using qualitative impact.

STEP 7
Estimate the seniority demonstrated by this work.

Examples:

Junior
SDE I
SDE II
Senior Engineer
Staff-level ownership

Explain your reasoning.

STEP 8
Suggest interview stories.

Generate STAR format stories from this Epic suitable for behavioral interviews.

Return everything in clean markdown.
Even Better: Use an Entire Epic Instead of Individual Tickets

Suppose your Epic contains:

Authentication Revamp

Story 1
Implement OAuth login

Story 2
Session management

Story 3
Refresh token support

Story 4
Redis cache

Story 5
Rate limiting

Story 6
Monitoring dashboards

Story 7
Bug fixes

Most engineers make the mistake of writing seven resume bullets.

Recruiters don't think in Jira tickets.

They think in projects.

The AI should combine them into something like:

Designed and implemented a scalable authentication platform by integrating OAuth, secure session management, Redis-backed token caching, and rate limiting, improving system reliability and supporting secure user authentication.

That's one high-quality bullet instead of seven low-impact ones.

A Second Prompt: Extract Metrics

Once you've identified the projects, use this prompt to find evidence for measurable impact.

You are helping me quantify software engineering work.

For every accomplishment below, list:

1. Metrics that would strengthen the resume.
2. Where those metrics are most likely stored.
3. SQL queries, Grafana dashboards, Kibana searches, Jira filters, or Git commands that could help me find them.
4. If exact numbers cannot be found, suggest truthful qualitative wording.

Never fabricate numbers.
Always distinguish facts from assumptions.
A Third Prompt: Turn Raw Technical Work into Business Value

This is one of the most valuable prompts because engineers naturally describe what they built rather than why it mattered.

For every technical task below,

translate it into

1. Business value
2. Customer value
3. Engineering value
4. Resume language
5. Interview language

Do not exaggerate.

Do not invent metrics.

Explain the impact in plain English.
My Recommended End-to-End Workflow
Jira Epic
      │
      ▼
All Jira Tickets
      │
      ▼
PRs + Git Commits + Design Docs
      │
      ▼
ChatGPT: Reconstruct Project
      │
      ▼
Find Missing Metrics
      │
      ▼
Search Grafana / Datadog / Jira
      │
      ▼
Update Resume Bullets
      │
      ▼
Generate STAR Interview Stories
      │
      ▼
Store in a "Career Accomplishments" document
One habit that will save you hours

Don't wait until you're job hunting. Create a document called Career Accomplishments with sections like:

Project name
Problem solved
Your role
Key technical decisions
Technologies
Evidence (Jira Epic, PRs, dashboards, release notes)
Metrics
Final resume bullet
STAR interview story

Update it after each major project. By the next job search, your resume will largely be a matter of selecting the most relevant accomplishments rather than trying to reconstruct years of work from memory.