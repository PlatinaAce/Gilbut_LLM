# Gilbut (길벗) - LLM & Backend Orchestration Server

This repository contains the backend core and LLM orchestration server for **Gilbut**, an AI-powered outdoor guide robot. Built on **Spring Boot** and integrated with the **Google Gemini API**, this server acts as the central middleware and control tower—managing full-stack data orchestration, asynchronous speech intent processing, relation-based context scoring, and dual downstream communication with ROS and TTS modules.

---

## 🚀 Key Technical Features

- **Asynchronous STT Data Ingestion**: Receives real-time speech text via WebSocket protocols. Isolates incoming requests using a dedicated **Thread Pool** to execute heavy LLM business tasks asynchronously, preventing blocking or packet drops.
- **Structured Intent Analysis via LLM**: Pipelines user utterances into dynamic prompts combined with strict System Prompts for the Gemini API. Enforces a precise **JSON Schema output** to maintain deterministic routing.
- **5-Class Intention Classification**: Automatically categorizes user commands into 5 deterministic actions:
  - `NAVIGATION_NEW_ROUTE` (New path guidance request)
  - `NAVIGATION_ADD_WAYPOINT` (Add a stopover to the existing path)
  - `NAVIGATION_CANCEL` (Reset path and stop robot movement immediately)
  - `CHAT` (General conversation / small talk feedback)
  - `ERROR` (Fallback routing for recognition faults)
- **Keyword Extraction & Destination Scoring**: Extracts *Positive Keywords* (features desired) and *Negative Keywords* (features excluded) from vague natural language. Runs a database cross-matching algorithm against destination hints, calculating 가산점 (+ points) and 감산점 (- points) to dynamically isolate the highest-scoring destination.
- **Stateful Context Storage**: Maintains temporary runtime memory (Context storage) of current navigation targets to elegantly re-calculate paths upon receiving composite commands like waypoint additions or cancel requests midway.
- **Persistence Layer Optimization**: Migrated from H2 In-Memory DB to **MySQL** for durable environment tracking. Models spatial locations (`Location`) and variant vocabulary strings (`Hint`) in a strict 1:N Foreign Key relationship using Spring Data JPA with a **Lazy Loading** strategy to maximize query throughput.

---

## 🛠 Tech Stack & Prerequisites

- **Java**: Amazon Corretto JDK 17 (v17.0.11 recommended)
- **Framework**: Spring Boot 3.x / Spring Data JPA
- **Database**: MySQL Server
- **API Orchestration**: Google Gemini API
- **Communication Protocol**: WebSocket / Rosbridge Protocol (`rosbridge_suite`)

---

## 🗄 Database Architecture (Entity Modeling)

The data 무결성 (integrity) is enforced via a relational database layout mapped through JPA entities:

- **Location Table**: Stores spatial coordinate details (`x`, `y`, `z`, `orientation`) and map frame attributes.
- **Hint Table**: Houses diverse, multi-variant keyword strings mapped to specific location IDs.
- **Relationship**: Mapped as a `1:N` relational mapping (`@OneToMany`) configured with `FetchType.LAZY` to drastically suppress redundant join operations during run-time scoring queries.

---

## 🔧 Environment Variables Configuration

Before building and starting the application, ensure the following environment configurations are successfully registered in your system profile:

```bash
# 1. Register Java Home to your system profile
export JAVA_HOME="/path/to/amazon-corretto-17.0.11"
export PATH=$JAVA_HOME/bin:$PATH

# 2. Register your Google AI Studio Gemini API Key
export GEMINI_API_KEY="your_actual_gemini_api_key_here"
```

## 🏃 Execution Instructions (How to Run)
Follow these operational steps sequentially to spin up the orchestrator server:

Download Deployment Files: Retrieve the pre-packaged execution binaries (.jar) from the designated Google Drive shared folder.

Setup Runtime Environment: Install Amazon Corretto JDK 17.0.11 and configure both JAVA_HOME and GEMINI_API_KEY environment variables as explained in the section above.

Launch Downstream ROS Bridge: Start your active ROS Master environment or boot the simulated environment:

```bash
java -jar simpleRosServer.jar
```

Boot the Orchestration Engine: Execute the target Spring Boot application jar:
```bash
# For testing and dry-run validation:
java -jar llmService_test.jar

# For production / full robot deployment:
java -jar llmService.jar
```

Teardown Process Safety: Upon completing your field tests or evaluation runs, you MUST explicitly terminate all background daemon processes using your terminal or process manager:
```bash
# Identify and terminate active java running jobs safely
kill -9 <PID_OF_YOUR_JAR_PROCESS>
```

## 📦 Sample Structured LLM Output Format (JSON)
When an intention is identified as a routing action, the internal Spring Boot parser parses a forced JSON format mirroring this convention:

```json
{
  "type": "NAVIGATION",
  "action": "NEW_ROUTE",
  "message": "안내를 시작할게요.",
  "destinations": [
    {
      "subType": "DESCRIBE",
      "positiveKeywords": ["카페"],
      "negativeKeywords": ["사람이 많은"]
    }
  ]
}
```

## 📄 License
This repository is protected and distributed under the terms of the MIT License.
