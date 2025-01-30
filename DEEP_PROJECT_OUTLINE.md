# Riona-AI-Agent — **Extremely Deep and Extensive Outline**

This document is intended to guide expansion of the Riona-AI-Agent project. It provides a comprehensive breakdown of each major function, utility, file, and design element. The aim is to facilitate a "chunk-based" approach to iterative improvements and expansions with large language models. **Sections are structured in a way that invites further exploration and enhancement.** You will find a checklist at the end to track expansions.

## Table of Contents

1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Main Components and Functionalities](#main-components-and-functionalities)
   1. [Agent and AI Logic](#agent-and-ai-logic)
   2. [Instagram Automation](#instagram-automation)
   3. [Twitter Automation (Placeholder)](#twitter-automation-placeholder)
   4. [GitHub Automation (Placeholder)](#github-automation-placeholder)
   5. [Database and Models](#database-and-models)
   6. [Utilities (Cookies, Logging, Error Handling)](#utilities-cookies-logging-error-handling)
4. [File-by-File Deep Dive](#file-by-file-deep-dive)
   1. [Root Project Files](#root-project-files)
   2. [Agents Subfolder](#agents-subfolder)
   3. [Training Subfolder](#training-subfolder)
   4. [Client Subfolder](#client-subfolder)
   5. [Config Subfolder](#config-subfolder)
   6. [Utils Subfolder](#utils-subfolder)
   7. [Test Subfolder](#test-subfolder)
5. [Known Constraints / Potential Bottlenecks](#known-constraints--potential-bottlenecks)
6. [Security Considerations](#security-considerations)
7. [Expansion and Enhancement Possibilities](#expansion-and-enhancement-possibilities)
   1. [Additional Social Media Integrations](#additional-social-media-integrations)
   2. [Improved Training Workflows](#improved-training-workflows)
   3. [Architecture Refactoring](#architecture-refactoring)
   4. [Better Error Handling or Observability](#better-error-handling-or-observability)
   5. [Optional Infrastructure as Code / Dockerization](#optional-infrastructure-as-code--dockerization)
8. [Chunk-Based Expansion Checklist](#chunk-based-expansion-checklist)
   1. [Agent & AI Logic](#agent--ai-logic)
   2. [Instagram Automation](#instagram-automation)
   3. [Twitter & X-bot](#twitter--x-bot)
   4. [GitHub Automation](#github-automation)
   5. [File Parsing & Training](#file-parsing--training)
   6. [Database & Models](#database--models)
   7. [Logging & Error Handling](#logging--error-handling)
   8. [Deployment & Scalability](#deployment--scalability)

---

## 1. Introduction

**Riona-AI-Agent** is an AI-based automation system designed primarily to interact with and automate tasks on Instagram. The project uses various libraries (such as Puppeteer, Instagram-Private-API, Google's Generative AI client libraries, and others) to:

1. **Log into an Instagram account** to post, like, or comment automatically.  
2. Generate AI-driven captions, comments, or tweets via Google's Gemini models.  
3. Offer a pipeline for training with various data sources (PDFs, text files, websites, or YouTube transcripts).  
4. Store or retrieve data in a MongoDB database, handling user or agent actions over time.

This outline offers a reference for every major function, script, and feature in the repository. Each section can be used as an independent "chunk" to further refine or expand.

---

## 2. Architecture Overview

At a high level, **Riona-AI-Agent** follows a layered architecture:

- **Application Entry**: `src/app.ts` and `src/index.ts` run initial processes, set up Express, and handle environment configurations.  
- **Agent Logic**: The `src/Agent` directory houses logic for generating content (text or comments) and managing training sessions.  
- **Clients**: The `src/client` directory includes platform-specific logic for automation (Instagram, Twitter, GitHub).  
- **Utilities**: The `src/utils` and `src/config` directories provide cross-cutting functionalities like logging, cookie management, error handling, environment config, or DB connectivity.  
- **Data/Schema**: The `src/Agent/schema` folder and `mongoose` model definitions define how we store AI outputs, such as Tweet data or generated text.

Within each layer, the code carefully handles functionalities like:
- **Authentication** (via username/password or stored cookies),
- **Scheduling** (via `node-cron` or `cron` library),
- **AI content generation** (via Google Generative AI model calls),
- **Database** (via Mongoose to connect to MongoDB).

---

## 3. Main Components and Functionalities

### 3.1 Agent and AI Logic

- **`runAgent(schema, prompt)`** in [`src/Agent/index.ts`](#agentindexts)  
  - Accepts a JSON schema (`InstagramCommentSchema`) and a textual prompt, then queries the Google Generative AI model (Gemini) to produce relevant content.  
  - Manages multiple API keys in a round-robin manner to avoid rate-limit issues.

- **Training Scripts**  
  - **`summarize.ts`** uses the Gemini API to transform YouTube transcripts into structured data.  
  - **`FilesTraining.ts`** parses various file types (PDF, DOC, CSV, TXT) to feed the content into further AI processing.

- **`TrainWithAudio.ts`**  
  - Handles audio file uploads to Google's AI file manager, generating content or transcripts from audio.

### 3.2 Instagram Automation

- **`runInstagram()`** in [`src/client/Instagram.ts`](#clientinstagramts)  
  - Launches a Puppeteer instance, optionally through a proxy.  
  - **Log in**: It first checks for existing cookies (`Instagram_cookiesExist`). If valid, it uses them. Otherwise, logs in with credentials from environment variables (`IGusername`, `IGpassword`).  
  - **Interaction**: Iterates over posts on the feed, likes them, and adds AI-generated comments.  
  - **Cookies**: Saves or loads cookies from `./cookies/Instagramcookies.json`.

---

### 3.3 Twitter Automation (Placeholder)

Even though a partial setup for Twitter is present (notably in `src/client/Twitter.ts`, which is mostly empty, and `src/client/X-bot/client/index.ts`), there is significant opportunity to:

1. **Authenticate and Post Tweets**  
   - The `twitter-api-v2` library is imported, but only partially implemented.  
   - We can integrate **full tweet posting** (with images, threads, or polls).  
   - Basic environment variables for Twitter credentials exist (`TWITTER_API_CREDENTIALS`), so the foundation is there.  

2. **Perform Advanced Actions**  
   - **Retweeting**: The system could retweet selected accounts or curated content.  
   - **Direct Messages**: We can manage DMs for targeted interactions or lead follow-ups.  
   - **Hashtag or Mention Analysis**: Scrape or monitor trending hashtags, mentions, or brand references, then auto-engage.  

3. **Potential Cron or Webhook Schedules**  
   - The approach is similar to Instagram scheduling: we can rely on `node-cron` or a queue-based system to schedule tweets.  
   - Alternatively, integrate Twitter's **webhooks** or streaming API for real-time triggers.  

4. **AI-Driven Tweet Generation**  
   - Reuse the logic from the `runAgent()` function: Provide a schema for tweet content (like `InstagramCommentSchema`) but specialized for tweets—maybe with a 280-character limit, or a 140 if you want a classic limit.  

**Transition from Step 2 to Step 3**  
- After robustly automating Instagram, the next logical extension is to expand to Twitter. The code can leverage the same AI pipeline for content generation, simply adjusting constraints (character limit, format, etc.). The scheduling and analytics lessons learned from Instagram can directly apply to Twitter.

---

## 4. File-by-File Deep Dive

This section enumerates key files to better understand each one's role. Use these breakdowns to identify expansion points.

### 4.1 Root Project Files

1. **`package.json`**
   Lists dependencies:
   - **Key**: `@google/generative-ai`, `puppeteer`, `puppeteer-extra`, `instagram-private-api`, `twitter-api-v2`, etc.
   - Contains scripts for building, running, or training the model.

2. **`README.md`**
   - Basic usage instructions, installation steps, and upcoming features overview.

### 4.2 Agents Subfolder

**`src/Agent/index.ts`**
- `runAgent(schema, prompt)` orchestrates calls to Google Generative AI.
- Manages logic for round-robin usage of `geminiApiKeys`.

**`src/Agent/characters/*.character.json`**
- Examples: `elon.character.json`, `sample.character.json`
- These contain persona definitions, such as name, bio, style, messageExamples, and typical topics or adjectives.

**`src/Agent/schema/index.ts`**
- Exports `InstagramCommentSchema` for structuring AI outputs.
- Also defines a Mongoose `Tweet` model.

**`src/Agent/script/summarize.ts`**
- Demonstrates transforming a YouTube transcript into a structured, training-ready prompt.
- Showcases schema-based generation with the Gemini API.

### 4.3 Training Subfolder

**`src/Agent/training/FilesTraining.ts`**
- Contains file parsing logic: PDF (`pdf-parse`), DOC/DOCX (`mammoth`, `textract`), CSV, and TXT.

**`src/Agent/training/TrainWithAudio.ts`**
- Demonstrates how to upload and process audio files, generate transcripts or short text from audio.

**`src/Agent/training/WebsiteScraping.ts`**
- Uses Puppeteer to scrape a website's text content, clean it with `DOMPurify`, and store the result (with `saveScrapedData`) for further training.

**`src/Agent/training/sample/*`**
- Contains sample files (`Audio.ts`, `test.txt`) for demonstration or testing.

### 4.4 Client Subfolder

**`src/client/Instagram.ts`**
- Houses `runInstagram()` function for orchestrating Instagram actions with Puppeteer.
- Uses `runAgent()` from `src/Agent/index.ts` for AI-generated comment text.

**`src/client/IG-bot/index.ts`**
- Offers an alternative approach using the `instagram-private-api` library.
- Demonstrates login, posting a photo, scheduling a post via `cron`.

**`src/client/Twitter.ts`** (placeholder)
- Future expansions for Twitter automation.

**`src/client/Github.ts`** (placeholder)
- Future expansions for GitHub automation.

**`src/client/X-bot/*`**
- Contains the Twitter client logic via `twitter-api-v2`, indicating a partial or upcoming feature.

### 4.5 Config Subfolder

**`src/config/db.ts`**
- Connects to MongoDB using Mongoose.
- Logs success or failure messages.

**`src/config/logger.ts`**
- Sets up Winston logging with multiple levels (`error, warn, info, debug`).
- Rotates daily logs to manage large log volumes.
- Manages `setupErrorHandlers()` for global error catching.

### 4.6 Utils Subfolder

**`src/utils/index.ts`**
- Key functions for cookie reading/writing: `Instagram_cookiesExist()`, `saveCookies()`, `loadCookies()`.
- `handleError(error, currentApiKeyIndex, schema, prompt, runAgent)` to manage AI service–related errors.
- Additional tweet data or scraped data saving: `saveTweetData()`, `saveScrapedData()`, and rate-limiting logic `canSendTweet()`.

**`src/utils/download.ts`**
- A simple function to download images from a given URI using `request`, storing them locally.

### 4.7 Test Subfolder

**`src/test/index.ts`**
- Example usage of `twitter-api-v2` to send tweets with images.
- Includes database logic to store tweets in the `Tweet` model.

**`src/test/tweets.ts`**
- An array (`excitingTweets`) for sample tweet content used in testing the Twitter client functionality.

---

## 5. Known Constraints / Potential Bottlenecks

1. **Rate Limits**
   - The Google Generative AI's Gemini model can throttle requests if usage is too high. The project rotates among multiple API keys (`geminiApiKeys`) to mitigate this, but rate-limits remain a potential bottleneck.
2. **Cookies Expiration**
   - Instagram session cookies eventually expire. The logic tries to reuse cookies for frictionless logins, but breaks if the session is invalidated.
3. **Frequent Puppeteer Updates**
   - Maintaining compatible versions across `puppeteer`, `puppeteer-extra`, and plugin packages can be tricky.
4. **Limited Testing**
   - Full test coverage is not present. The project includes some ad-hoc tests but no formal integration or unit test structure.

---

## 6. Security Considerations

- **Storing Credentials**: The `.env` file or environment variables hold sensitive credentials. They must be stored securely (avoid pushing to public repos).
- **Database Access**: The `MONGODB_URI` must remain protected.
- **Social Media Account Safety**: Abnormal login patterns with Puppeteer or Instagram Private API might trigger account flags or bans.
- **API Keys**: A large array of Gemini API keys is used. They must be valid keys and carefully guarded.

---

## 7. Expansion and Enhancement Possibilities

### 7.1 Additional Social Media Integrations
- The code structure accommodates expansions to more platforms: LinkedIn, Facebook, or Reddit.
- Each integration would have a dedicated subfolder in `src/client/<Platform>` with a similar pattern to the Instagram approach.

### 7.2 Improved Training Workflows
- The training scripts can unify multiple data sources (text, audio, website scraping, PDFs) into a single pipeline that outputs a consolidated knowledge base.
- Enhanced chunking or summarization logic to handle large documents.

### 7.3 Architecture Refactoring
- Potential to refactor the "Agent" logic into microservices or dedicated modules.
- Could adopt a message bus architecture for tasks like scraping, generating text, or scheduling.

### 7.4 Better Error Handling or Observability
- Introduce structured logs and metrics for better monitoring.
- Possibly integrate with Sentry or another error reporting service.

### 7.5 Optional Infrastructure as Code / Dockerization
- Provide Dockerfiles for running the app in containerized form.
- Let the system run in ephemeral environments or scale more easily.

---

## 8. Chunk-Based Expansion Checklist

Below is a practical checklist for expansions. Mark items as `[x] expanded` once done:

1. **Agent & AI Logic**  
   - [x] **Consolidate** the agent's prompt generation strategy.  
   - [x] **Refine** error fallback logic to handle even more edge cases.  
   - [x] **Embed** advanced prompt context (like persona or conversation memory).

2. **Instagram Automation**  
   - [x] **Improve** post detection to handle new UI changes or page layouts.  
   - [x] **Extract** additional data (like post stats or hashtags) for analytics.  
   - [x] **Add** advanced scheduling or queue management for content posting.

3. **Twitter & X-bot**  
   - [x] **Full Implementation** of tweet posting, retweeting, direct messages.  
   - [x] **Rate Limit** checks for Twitter.  
   - [x] **Handle** media uploads more robustly (e.g., videos, GIFs).

**Transition from Step 2 to Step 3**  
- After establishing a reliable automation pipeline and scheduling logic with Instagram, the same architecture can be extended to Twitter. This includes:
  - **Prompt generation** for tweets, possibly with more constraints (character count).  
  - **Scheduling** or queue-based approach.  
  - **Analytics** (like how many favorites or retweets each tweet gets).  
  - The synergy from Instagram expansions (step 2) ensures a unified code approach across multiple social media integrations.

4. **GitHub Automation**  
   - [x] **Implement** logic to open, close, or comment on issues.  
     - **Deep Explanation**: By connecting to the GitHub API, the system can automate essential repository tasks:
       1. **Open and Close Issues**: Provide a feature to automatically create new issues if certain conditions are met (e.g., a bug is detected in logs, or a user request in your AI conversation). The system can also close issues when tasks are completed or after certain triggers.  
       2. **Comment on Issues**: This can be used for collaborative or automated workflows, such as providing updates, referencing external data, or linking to relevant commits or pull requests.  
       3. **Labeling and Assigning**: Optionally, the system could add labels or assign issues to specific team members based on logic or integration with an AI-based recommendation engine.

   - [x] **Extend** to handle repository forks, merges, or PR automation.  
     - **Deep Explanation**: Beyond issues, the system can automate higher-level repository tasks:
       1. **Forking Repositories**: Useful for organizational scenarios—if you maintain a stable upstream repo but want an AI-managed fork for experimentation or bots.  
       2. **Merging PRs**: Implement logic to automatically merge pull requests if certain checks (like CI tests) pass. Possibly use AI-based code review comments.  
       3. **Branch Management**: For example, if a PR has no activity for a set time, the system might close or rebase it automatically.  
       4. **Release Automation**: Tagging releases, generating changelogs, or performing version bumps after AI-led analysis.  

**Transition from Step 3 to Step 4**  
- After implementing robust support for Twitter automation, the next logical progression is GitHub Automation. This shift leverages similar ideas: scheduling tasks, reacting to triggers (issues, PRs), and injecting AI-based enhancements. It introduces a deeper DevOps or code-management angle—where the system not only interacts on social platforms but also manipulates code repositories and fosters collaboration.  
- The pipeline patterns from earlier expansions (like prompt generation, scheduling, or multi-key usage for rate-limit avoidance) can be adapted to GitHub's API constraints.

5. **File Parsing & Training**
   - [x] **Chunk** large files into smaller sections for summarization.
     - **Deep Explanation**: When dealing with large or monolithic documents (e.g., 100+ pages of text or hours of transcripts), chunking them can help:
       1. **Memory Efficiency**: Handling smaller parts reduces the risk of exceeding model input limits or local memory constraints.
       2. **Focused Summaries**: Summaries per chunk are more coherent, enabling iterative or hierarchical summarization (e.g., summarizing each chunk, then summarizing the summaries).
       3. **Parallel Processing**: Each chunk can be processed in parallel, improving performance if you have multiple AI calls or microservices.
       4. **Scalability**: The system can store partial analyses, letting you revisit only certain chunks when data changes.

     - **Transition from Step 4**: After enabling GitHub automation (Step 4), the project is already ingesting code or docs. The next logical approach is to refine how large repositories or documentation sets get parsed and summarized for training.

   - [x] **Add** advanced NLP pipeline steps (e.g., keyword extraction, topic modeling).
     - **Deep Explanation**: Beyond raw text ingestion, introducing NLP pipelines fosters better structure:
       1. **Keyword Extraction**: Identifies critical terms or concepts for more nuanced summarization or user queries.
       2. **Topic Modeling**: Groups content into themes or categories—useful for organizing knowledge bases or responding to user questions with context.
       3. **Entity Recognition**: Potentially identify named entities (people, places, organizations) for advanced query handling or deeper AI training.
       4. **Hybrid Approach**: Combine existing AI summarization with classical NLP (Spacy, NLTK, or Transformers-based libraries) to refine or label data before final training.
     - **Link to Step 6**: As you begin storing more structured data from advanced NLP, you might expand your database schemas or implement versioning strategies (Step 6). This helps keep track of multiple pipeline outputs.

   - [x] **Enhance** audio transcription accuracy with alternative engines.
     - **Deep Explanation**: Currently, `TrainWithAudio.ts` uses Google's Generative AI approach for transcription. Additional ideas:
       1. **External STT Services**: Services like AssemblyAI, DeepSpeech, or Whisper can sometimes yield higher accuracy or handle specialized vocabulary better.
       2. **Custom Models**: If you have domain-specific audio (medical, legal, etc.), you might train or fine-tune a custom transcription model.
       3. **Fallback / Hybrid**: Use multiple STT engines in parallel and compare or ensemble results for best accuracy.
       4. **Post-processing**: Introduce advanced text normalization or re-punctuation to yield better final transcripts.
     - **Transition to Step 6**: Once better transcriptions are stored, you can incorporate them into your database or analytics flows. This also ties into potential expansions in how you catalog or version these transcripts.

By systematically enhancing the file parsing and training, we ensure that:
- Large documents and transcripts become more manageable.
- Summaries and NLP-derived metadata feed into advanced training workflows (like chat agents or domain-specific question-answering).
- The system gains better coverage for various data types (text, audio, possibly images in future steps).

6. **Database & Models**
   - [x] **Expand** the `Tweet` schema with analytics (likes, retweets count).
     - **Deep Explanation**: Expanding the `Tweet` schema to include analytics fields like `likeCount`, `retweetCount`, or `engagementRate` can significantly improve how you measure success.
       1. **Visibility of Performance**: Storing these metrics helps you see which tweets perform best, guiding content strategy.
       2. **Filtering & Sorting**: You can filter or sort tweets in the database by engagement metrics, identifying top-performing tweets for reposts or further AI analysis.
       3. **Historical Tracking**: By regularly updating these fields, you can track how engagement evolves over time.

     - **Transition from Step 5**: After implementing chunk-based data ingestion or advanced NLP on tweets, it's logical to capture the real-world performance (likes, retweets) in the database. This bridging step means your system can not only analyze text but also tie those insights to actual engagement metrics.

   - [x] **Add** more data models for other platforms (Instagram posts, GitHub issues).
     - **Deep Explanation**: Just as with tweets, having well-defined Mongoose models for Instagram or GitHub data unlocks multi-platform synergy:
       1. **Uniformity**: Each platform's data is stored in a consistent structure (e.g., `IGPost`, `GitHubIssue`).
       2. **Cross-Platform Analysis**: You could measure or compare engagement across platforms, or triage issues from multiple sources in a single dashboard.
       3. **Future-proofing**: As you add new social features (automatic DMs, multi-account analytics), a robust schema approach ensures the code can scale.

     - **Transition**: With new data models in place, the system is prepared for expansions in Steps 7 and 8, such as advanced logging or containerization. You'll also be better positioned for cohesive analytics across multiple channels.

   - [x] **Implement** data versioning or historical logs.
     - **Deep Explanation**: Keeping historical snapshots of data can be critical for understanding changes over time:
       1. **Comparative Studies**: For instance, you can see how a tweet's engagement changes week-over-week or how a GitHub issue evolves across its lifecycle.
       2. **Rollback & Audit**: If something goes wrong (e.g., accidental data overwrite), versioning ensures you can recover older states or prove who changed what.
       3. **Advanced AI**: Over time, historical logs can help train models to predict future engagement or prioritize tasks.
       4. **Implementation Approaches**: You might store deltas in a separate "history" collection, or use Mongoose plugins like `mongoose-version` that handle versioning automatically.

     - **Transition to Step 7**: Once you have robust versioning, you'll likely want advanced logging or aggregator integrations (Step 7). The synergy between versioned data and logs fosters end-to-end traceability (who changed what, when, and why), bridging smoothly into the next expansions.

7. **Logging & Error Handling**
   - [x] **Integrate** a real-time log aggregator (ELK, Datadog, or Sumo Logic).
     - **Deep Explanation**: A centralized log aggregator can unify logs from Puppeteer, AI prompts, and Node server processes:
       1. **Consolidated Visibility**: Instead of checking logs in multiple files, teams can see every event in real time in a single dashboard.
       2. **Search & Analytics**: Tools like Elasticsearch (part of the ELK stack) offer powerful search and filtering, so you can track errors or performance patterns.
       3. **Alerts**: You can configure triggers to notify you if, for example, a certain number of errors occur within a specified interval or if a particular error pattern recurs frequently.
       4. **Scalability**: As more social media accounts or data sources come online, a real-time aggregator can handle large volumes of logs more efficiently than local files.

   - [x] **Add** structured logging fields (request ID, environment, user ID).
     - **Deep Explanation**: Transitioning from plain text logs to structured logs with consistent fields yields:
       1. **Traceability**: A `requestId` or `sessionId` links every log entry to a specific user session or pipeline run.
       2. **Filtering & Aggregation**: It's easier to search or filter logs by environment (dev, stage, prod) or user.
       3. **Better Post-Mortems**: In case of incidents, you can quickly isolate logs belonging to the same request or user action.
       4. **Machine Parsing**: Structured logs can be ingested by analytics systems or machine learning pipelines for anomaly detection or forecasting.

   - [x] **Create** an end-to-end error classification (Network, AI service, Puppeteer, etc.).
     - **Deep Explanation**: A classification approach ensures errors are categorized and handled (or escalated) consistently:
       1. **Network Errors**: Might be caused by proxy issues, request timeouts, or DNS problems. The system can retry or switch proxies if this type is recognized.
       2. **AI Service Errors**: E.g., 429 (Too Many Requests) from Gemini. The system can apply backoff logic or rotate to a new API key.
       3. **Puppeteer/Automation Errors**: If elements are missing or the page doesn't load, specialized handlers can decide whether to reload or skip.
       4. **Database or Application Errors**: Might indicate local logic or schema mismatches, requiring developer intervention or immediate rollback.
       5. **Benefits**: With a robust classification scheme, you gain better visibility into overall system health and can implement specialized fallback strategies.

**Transition from Step 6 to Step 7**
- Once your database and model strategies (Step 6) are in place—especially with versioning or historical logs—it's much easier to correlate structured data changes with associated logs. Logging expansions benefit from the richer data context (like versioned schema states). This synergy ensures every insert, update, or AI call can be mapped to logs in real time, clarifying the "who, what, when, and why" behind each change or error.

**Transition from Step 7 to Step 8**
- After establishing advanced logging and error classification, you'll be ready to address broader deployment challenges in Step 8. Containerization, orchestration, and scaling heavily depend on robust observability. Real-time logs and structured error categories feed into automated scaling triggers, health checks, or rolling updates in Docker/Kubernetes. Thus, a solid logging foundation is key to successful expansion into more complex deployment strategies.

8. **Deployment & Scalability**  
   - [ ] **Containerize** the entire application with Docker.  
   - [ ] **Add** Kubernetes or other orchestration if necessary.  
   - [ ] **Implement** background worker to handle scheduled tasks or scraping in parallel.

Use these sections as anchors for future expansions. Once you implement or refine each bullet, you can mark it `[x] expanded`.

---

**This document is meant to evolve.** Every time you expand on a chunk, refer back to the relevant bullet or sub-section. Let these references serve as breadcrumbs for large language model–driven enhancements.
