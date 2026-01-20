# Act Use Cases & Real-World Scenarios

> **Version**: 1.0  
> **Last Updated**: 2026-01-20  
> **Purpose**: Document real-life use cases that Act enables for everyday users

---

## Executive Summary

Act transforms complex, multi-app workflows into simple voice or text commands. This document catalogs use cases organized by:

1. **Daily Life & Personal Organization**
2. **Family & Social Coordination**
3. **Learning & Productivity**
4. **Health & Wellness**
5. **Students & Job Seekers**
6. **Small Business & Freelancers**
7. **Complex Multi-Step Workflows**

Each use case includes the user command, what Act does behind the scenes, and the toolkits involved.

---

## 1. Daily Life & Personal Organization

### 1.1 Morning Routine

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "What's my day look like?" | Shows calendar events, weather, commute time, pending reminders | Google Calendar, Weather API |
| "Tell me today's news headlines" | Fetches top stories from preferred sources | News API, Web Search |
| "Add milk, eggs, and bread to my shopping list" | Updates shopping list | Todoist, Apple Reminders, Notion |
| "Remind me to call mom at 6pm" | Creates timed reminder | Google Calendar, Apple Reminders |
| "What's the traffic to office?" | Checks real-time commute | Maps API |

### 1.2 Shopping & Finance

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "How much did I spend on Amazon this month?" | Scans Gmail for receipts, calculates total | Gmail |
| "Track this expense: ₹500 for groceries" | Logs to budget spreadsheet | Google Sheets |
| "What's the best price for iPhone 16 right now?" | Compares prices across sites | Web Search |
| "What's my bank balance?" | Fetches from banking integration | Banking API (if available) |
| "Pay my electricity bill" | Initiates payment (with confirmation) | Payment Integrations |

### 1.3 Travel Planning

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Find flights to Goa for next weekend under ₹5000" | Searches flight aggregators | Flight APIs, Web Search |
| "Book an Uber to the airport for 6am tomorrow" | Schedules ride | Uber API |
| "Add my flight details to calendar" | Parses confirmation email, creates event | Gmail, Google Calendar |
| "What's the weather in Mumbai next week?" | 7-day forecast lookup | Weather API |
| "Find hotels near Marina Beach for under ₹3000/night" | Hotel search with filters | Hotel APIs, Web Search |

---

## 2. Family & Social Coordination

### 2.1 Communication

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Text my wife: I'll be home in 20 minutes" | Sends message | WhatsApp, SMS |
| "Send birthday wishes to Rahul" | Composes and sends greeting | WhatsApp, Email |
| "Email the school about my child's absence tomorrow" | Drafts and sends formal email | Gmail |
| "Post on Instagram: Beautiful sunset today" | Creates post with recent photo | Instagram API |
| "Reply to all unread WhatsApp messages" | Summarizes and drafts responses | WhatsApp |

### 2.2 Event Coordination

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Create a party invite for Saturday at 7pm and send to friends" | Creates event, sends invites | Google Calendar, WhatsApp, Gmail |
| "Find a good restaurant near Koramangala for 6 people" | Searches with ratings | Google Maps, Yelp, Zomato |
| "Book a table at that Italian place for tomorrow 8pm" | Makes reservation | Restaurant booking APIs |
| "Share the restaurant location with family group" | Sends location | WhatsApp |
| "Send reminder to everyone about tomorrow's party" | Bulk reminder message | WhatsApp, Gmail |

### 2.3 Family Management

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Add dentist appointment for kids on Friday 4pm" | Creates calendar event with reminders | Google Calendar |
| "What activities do the kids have this week?" | Lists family calendar events | Google Calendar |
| "Order groceries for the week" | Places order from saved list | Grocery delivery APIs |
| "Find a plumber available today" | Searches local services | Urban Company, Local search |
| "Track the Amazon package for my son's birthday gift" | Checks delivery status | Amazon, Shipping APIs |

---

## 3. Learning & Productivity

### 3.1 Research & Learning

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Summarize this YouTube video for me" | Transcribes and summarizes | YouTube, AI Summarization |
| "Find the top 5 books on productivity" | Curated book recommendations | Web Search, Goodreads |
| "Save this article to read later" | Bookmarks content | Pocket, Notion, Readwise |
| "What's the capital of Kazakhstan?" | Instant knowledge lookup | Web Search, Wikipedia |
| "Translate 'How are you?' to Japanese" | Real-time translation | Translation API |

### 3.2 Focus & Deep Work

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Start my focus timer for 2 hours" | Sets Pomodoro timer, enables DND | Timer, System DND |
| "Block social media for the next hour" | Activates website blocker | Focus mode apps |
| "Take meeting notes" | Transcribes and summarizes | Voice transcription, Notion |
| "What tasks are due today?" | Lists pending items | Todoist, Linear, Asana |
| "Reschedule my afternoon meetings to tomorrow" | Bulk reschedule with notifications | Google Calendar |

### 3.3 Knowledge Management

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Add this to my quotes collection" | Saves to Notion database | Notion |
| "What did I save about machine learning last month?" | Searches personal notes | Notion, Obsidian |
| "Create a mind map for my business idea" | Generates outline | Whimsical, Miro, Notion |
| "Summarize the key points from my last 5 saved articles" | Aggregates and summarizes | Pocket, Readwise |

---

## 4. Health & Wellness

### 4.1 Health Tracking

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Log my workout: 30 minutes running, 5km" | Adds to fitness tracker | Apple Health, Google Fit, Sheets |
| "What did I eat this week?" | Reviews food log entries | MyFitnessPal, Health apps |
| "Remind me to take medicine every 8 hours" | Creates recurring reminders | Reminders, Calendar |
| "How many steps did I walk today?" | Fetches health data | Apple Health, Google Fit |
| "Log my weight: 72kg" | Updates health records | Health tracking apps |

### 4.2 Medical & Appointments

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Book a doctor appointment for Thursday" | Finds slots, schedules | Practo, Booking APIs |
| "Find a cardiologist near me with good reviews" | Doctor search | Practo, Google Maps |
| "Remind me about my health checkup next month" | Creates reminder | Calendar, Reminders |
| "What vaccinations are due for my daughter?" | Checks health records | Health apps, Calendar |

### 4.3 Mental Wellness

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Start my morning meditation" | Launches meditation timer | Meditation apps, Timer |
| "How was my sleep this week?" | Analyzes sleep patterns | Sleep tracking apps |
| "Journal entry: Today I felt grateful for my family" | Adds to journal | Day One, Notion |
| "What's my mood trend this month?" | Shows mood analysis | Mood tracking apps |

---

## 5. Students & Job Seekers

### 5.1 Academic Life

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "When is my next exam?" | Checks academic calendar | Google Calendar, Student apps |
| "Find research papers on climate change from 2024" | Academic search | Google Scholar, Web Search |
| "Create flashcards from my biology notes" | Generates study cards | Anki, Quizlet |
| "Calculate my CGPA if I get 85 in Math" | GPA calculation | Custom calculation |
| "Set study reminders for the next 2 weeks" | Creates study schedule | Calendar, Reminders |

### 5.2 Assignments & Projects

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "What assignments are due this week?" | Lists pending work | Notion, Todoist, Calendar |
| "Create an outline for my history essay" | Generates structure | AI assistance, Notion |
| "Find 5 sources about renewable energy" | Research assistance | Web Search, Google Scholar |
| "Share my project document with study group" | File sharing | Google Drive, Dropbox |

### 5.3 Job Hunting

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Find remote product manager jobs in India" | Job search | LinkedIn, Indeed, Naukri |
| "Update my LinkedIn status to Open to Work" | Profile update | LinkedIn |
| "Draft a cover letter for the Google position" | Creates customized letter | AI assistance, Gmail |
| "Track my job applications in a spreadsheet" | Updates tracker | Google Sheets |
| "Prepare me for a software engineering interview" | Interview prep resources | Web Search, AI coaching |

---

## 6. Small Business & Freelancers

### 6.1 Client Management

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Send invoice to Sharma & Co for ₹50,000" | Creates and emails invoice | Zoho Invoice, Gmail |
| "Follow up with leads who haven't responded in 3 days" | Drafts follow-up emails | Gmail, CRM |
| "Schedule a call with new client for next week" | Sends calendar invite | Google Calendar, Gmail |
| "What's my accounts receivable this month?" | Calculates pending payments | Sheets, Accounting software |
| "Add new client: Ravi Kumar, +91-9876543210" | Creates contact and record | CRM, Contacts |

### 6.2 Content Creators

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "Draft 5 tweet ideas about productivity" | Generates content ideas | AI assistance, Twitter |
| "Schedule Instagram posts for the week" | Batches posts with timing | Instagram, Buffer, Later |
| "What's trending on Twitter in tech?" | Fetches trending topics | Twitter API |
| "Create a thumbnail for my new video" | Generates image | Canva, AI image tools |
| "How many YouTube subscribers did I gain this week?" | Analytics check | YouTube Analytics |

### 6.3 Operations

| Voice Command | What Act Does | Toolkits |
|---------------|---------------|----------|
| "What meetings do I have with clients this week?" | Filters calendar | Google Calendar |
| "Send weekly report to all clients" | Generates and sends reports | Gmail, Sheets |
| "Track time spent on Project Alpha" | Time logging | Toggl, Clockify |
| "Create a proposal for the new website project" | Generates proposal | Notion, Google Docs |

---

## 7. Complex Multi-Step Workflows

These advanced workflows showcase Act's ability to chain multiple tools and execute sophisticated tasks.

### 7.1 Complete Trip Planning

**User Command:**
> "Plan a 3-day trip to Jaipur next month for 2 people, budget ₹30,000"

**Act Execution (10 steps):**

| Step | Action | Toolkit |
|------|--------|---------|
| 1 | Check calendar for available 3-day windows | Google Calendar |
| 2 | Search for best flight/train deals | Travel APIs |
| 3 | Find top-rated hotels within budget | Hotel APIs |
| 4 | Create day-by-day itinerary with attractions | Web Search, AI |
| 5 | Book transportation (with confirmation) | Booking APIs |
| 6 | Book accommodation (with confirmation) | Hotel APIs |
| 7 | Add all bookings to calendar | Google Calendar |
| 8 | Create packing checklist | Todoist, Notion |
| 9 | Share trip details with travel companions | WhatsApp |
| 10 | Set weather alerts for travel dates | Weather API |

**Time Saved:** 4+ hours  
**Confirmation Required:** Yes (for bookings)

---

### 7.2 Birthday Party Orchestrator

**User Command:**
> "Organize my daughter's 7th birthday party on March 15th for 20 kids"

**Act Execution (8 steps):**

| Step | Action | Toolkit |
|------|--------|---------|
| 1 | Create party event in calendar | Google Calendar |
| 2 | Design digital invitation | Canva |
| 3 | Send invites to parent contacts | WhatsApp, Email |
| 4 | Set up RSVP tracking | Google Sheets |
| 5 | Order themed cake from bakery | Local ordering |
| 6 | Find and book party entertainer | Local search |
| 7 | Create shopping list for decorations | Todoist |
| 8 | Send reminder to guests 2 days before | WhatsApp |

**Time Saved:** 3+ hours  
**Confirmation Required:** Yes (for bookings and bulk messages)

---

### 7.3 House/Apartment Hunting

**User Command:**
> "Help me find a 2BHK apartment in Bangalore HSR Layout under ₹25,000/month"

**Act Execution (7 steps):**

| Step | Action | Toolkit |
|------|--------|---------|
| 1 | Search Housing, MagicBricks, NoBroker | Real estate APIs |
| 2 | Filter by criteria (amenities, location) | Data processing |
| 3 | Create comparison spreadsheet | Google Sheets |
| 4 | Schedule site visits with owners/agents | Gmail, Calendar |
| 5 | Add visits to calendar with addresses | Google Calendar, Maps |
| 6 | Calculate total move-in costs | Sheets calculation |
| 7 | Draft inquiry emails to shortlisted | Gmail |

**Time Saved:** 5+ hours  
**Confirmation Required:** Yes (for contacting owners)

---

### 7.4 Financial Health Review

**User Command:**
> "Give me a complete financial review for January"

**Act Execution (8 steps):**

| Step | Action | Toolkit |
|------|--------|---------|
| 1 | Fetch transaction emails/statements | Gmail |
| 2 | Categorize spending (food, transport, bills, shopping) | AI categorization |
| 3 | Compare to previous month | Sheets analysis |
| 4 | Identify unused subscriptions | Pattern detection |
| 5 | Check investment portfolio (if linked) | Investment APIs |
| 6 | Calculate savings rate | Computation |
| 7 | Create visual spending report | Sheets, Charts |
| 8 | Generate expense reduction suggestions | AI recommendations |

**Time Saved:** 2+ hours  
**Confirmation Required:** No (read-only analysis)

---

### 7.5 Event Photo Organizer

**User Command:**
> "Organize photos from my Goa trip and share the best ones with family"

**Act Execution (5 steps):**

| Step | Action | Toolkit |
|------|--------|---------|
| 1 | Identify trip photos by date/location metadata | Photos API |
| 2 | Group into albums (beaches, food, friends) | AI categorization |
| 3 | Select best 10 photos using quality scoring | AI selection |
| 4 | Create collage or slideshow | Design tools |
| 5 | Share to family WhatsApp group | WhatsApp |

**Time Saved:** 1+ hour  
**Confirmation Required:** Yes (for sharing)

---

### 7.6 Wedding Planning Assistant

**User Command:**
> "Help me plan my wedding on December 15th in Mumbai"

**Act Execution (12+ steps over multiple sessions):**

| Step | Action | Toolkit |
|------|--------|---------|
| 1 | Create master wedding checklist (100+ items) | Notion, Todoist |
| 2 | Set up budget tracker | Google Sheets |
| 3 | Research and compare venues | Web Search |
| 4 | Create vendor contact database | Sheets, Notion |
| 5 | Set up RSVP tracking | Google Sheets |
| 6 | Design and send save-the-dates | Canva, Email |
| 7 | Schedule vendor meetings | Calendar, Email |
| 8 | Track guest dietary preferences | Sheets |
| 9 | Create wedding day timeline | Sheets, Notion |
| 10 | Book honeymoon travel | Travel APIs |
| 11 | Send final reminders to guests | WhatsApp, Email |
| 12 | Prepare thank-you note templates | AI assistance |

**Time Saved:** 50+ hours (over weeks)  
**Confirmation Required:** Yes (for bookings and communications)

---

### 7.7 Job Application Campaign

**User Command:**
> "Apply to 10 product manager positions at top tech companies this week"

**Act Execution (6 steps):**

| Step | Action | Toolkit |
|------|--------|---------|
| 1 | Search for PM roles at target companies | LinkedIn, Company sites |
| 2 | Filter by location, experience, salary | Data filtering |
| 3 | Customize resume for each role | AI assistance, Docs |
| 4 | Draft tailored cover letters | AI assistance |
| 5 | Submit applications (with confirmation) | Job portals |
| 6 | Track all applications in spreadsheet | Google Sheets |

**Time Saved:** 6+ hours  
**Confirmation Required:** Yes (for each submission)

---

### 7.8 Home Renovation Coordinator

**User Command:**
> "Coordinate my kitchen renovation project"

**Act Execution (8 steps):**

| Step | Action | Toolkit |
|------|--------|---------|
| 1 | Create project timeline with milestones | Notion, Sheets |
| 2 | Find and compare contractors | Local search, Reviews |
| 3 | Schedule consultations | Calendar, Email |
| 4 | Create materials shopping list | Sheets |
| 5 | Track expenses against budget | Sheets |
| 6 | Send progress updates to family | WhatsApp |
| 7 | Schedule inspections | Calendar |
| 8 | Coordinate delivery schedules | Email, Calendar |

**Time Saved:** 10+ hours  
**Confirmation Required:** Yes (for scheduling)

---

## 8. Complexity Reference Matrix

### By Number of Steps

| Complexity | Steps | Example Use Cases |
|------------|-------|-------------------|
| **Simple** | 1-2 | Send email, check calendar, search web |
| **Moderate** | 3-5 | Schedule meeting with agenda, create and share document |
| **Complex** | 6-10 | Trip planning, event organization, financial review |
| **Advanced** | 10+ | Wedding planning, multi-week projects |

### By Toolkits Required

| Complexity | Toolkits | Example |
|------------|----------|---------|
| **Single** | 1 | "Check my calendar" (Calendar only) |
| **Paired** | 2 | "Email meeting summary to participants" (Calendar + Gmail) |
| **Multi** | 3-5 | "Plan a trip" (Calendar + Travel + Maps + Sheets + WhatsApp) |
| **Orchestrated** | 6+ | "Wedding planning" (8+ toolkits) |

### By Confirmation Requirements

| Type | Confirmation? | Examples |
|------|---------------|----------|
| **Read-only** | Never | Search, check, summarize, fetch |
| **Create** | Sometimes | Create task, add reminder |
| **Send/Share** | Always | Email, message, post |
| **Book/Pay** | Always | Reservations, purchases |
| **Delete** | Always | Remove, cancel, archive |

---

## 9. Implementation Priority

### MVP (Phase 1) - Must Have

| Category | Use Cases |
|----------|-----------|
| Email | Send, read, search, reply |
| Calendar | Check, create, reschedule |
| Tasks | Create, complete, list |
| Search | Web, news, weather |
| Messaging | Send WhatsApp/Slack |

### Growth (Phase 2) - Should Have

| Category | Use Cases |
|----------|-----------|
| Finance | Expense tracking, budgets |
| Travel | Search flights, hotels |
| Documents | Create, share, edit |
| Social | Post to social media |

### Advanced (Phase 3) - Nice to Have

| Category | Use Cases |
|----------|-----------|
| Shopping | Price comparison, orders |
| Health | Appointment booking, tracking |
| Home | Smart home control |
| Multi-step | Complex workflows |

---

## 10. Success Metrics by Use Case

| Use Case Category | Success Metric | Target |
|-------------------|----------------|--------|
| Email actions | Successful send rate | >99% |
| Calendar operations | Conflict-free scheduling | >95% |
| Search queries | Relevant first result | >90% |
| Multi-step workflows | Full completion rate | >85% |
| Confirmation prompts | User approval rate | >80% |

---

## Appendix: Toolkit Coverage

### Fully Supported (MVP)

- Gmail
- Google Calendar
- Slack
- WhatsApp (via bridge)
- Google Sheets
- Todoist
- Linear
- GitHub
- Web Search

### Planned Support

- Instagram
- Twitter/X
- Spotify
- Uber
- Zomato/Swiggy
- Amazon
- Banking integrations
- Smart home (HomeKit)

---

**Document Status:** Living document - will be updated as new use cases are identified and implemented.

**Feedback:** Document real user requests and successful completions to expand this catalog.
