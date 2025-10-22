# 🤖 Travel Agent using ADK & MCP Toolbox for Databases

![Google Cloud](https://img.shields.io/badge/Google%20Cloud-%234285F4.svg?style=for-the-badge&logo=google-cloud&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-%23336791.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Gemini](https://img.shields.io/badge/Gemini%20AI-4285F4?style=for-the-badge&logo=google&logoColor=white)
![ADK](https://img.shields.io/badge/Agent%20Development%20Kit-34A853?style=for-the-badge&logo=google&logoColor=white)
![MCP Toolbox](https://img.shields.io/badge/MCP%20Toolbox-0F9D58?style=for-the-badge&logo=google&logoColor=white)

---

## 📚 Table of Contents
- [🧭 Project Overview](#-project-overview)
- [🏗️ Architecture Overview](#️-architecture-overview)
- [⚙️ Implementation Phases](#️-implementation-phases)
  - [Phase 1: Infrastructure Setup (Cloud SQL)](#phase-1-infrastructure-setup-cloud-sql)
  - [Phase 2: MCP Toolbox Configuration (Tool Definition)](#phase-2-mcp-toolbox-configuration-tool-definition)
  - [Phase 3: ADK Agent Integration](#phase-3-adk-agent-integration)
  - [Phase 4: Testing & Deployment](#phase-4-testing--deployment)
- [🧹 Cleanup](#-cleanup)
- [🔗 Reference](#-reference)
- [🧠 Key Takeaways](#-key-takeaways)


# 🤖 Travel Agent using ADK & MCP Toolbox for Databases

### 🧭 Project Overview
This project implements a **data-grounded AI Travel Agent** using the **Agent Development Kit (ADK)** and the **MCP Toolbox for Databases**, inspired by Google’s official codelab.

The objective is to create a **production-grade conversational agent** that can handle complex travel-related queries by securely connecting to a **PostgreSQL Cloud SQL** database, powered by **Gemini models**.

---

## 🏗️ Architecture Overview

The system operates across **three secure layers**:

| Layer | Description |
|--------|--------------|
| 🗄️ **Database Layer (Cloud SQL)** | The immutable source of truth containing hotel and travel data. |
| 🧰 **Toolbox Layer (MCP Toolbox)** | Provides secure, parameterized SQL query execution via predefined tools. |
| 🤖 **Agent Layer (ADK)** | Hosts the Gemini model that reasons about user input and triggers external tool calls. |

---

## ⚙️ Implementation Phases

### **Phase 1: Infrastructure Setup (Cloud SQL)**

1. **Google Cloud Project Setup**
   - Create a new Cloud project.
   - Enable APIs:
     ```
     sqladmin.googleapis.com
     run.googleapis.com
     aiplatform.googleapis.com
     ```

2. **Database Provisioning**
   - Create a PostgreSQL instance named `hoteldb-instance` on Cloud SQL.
   - Connect via **Cloud SQL Studio**.

3. **Schema & Data Setup**
   ```sql
   CREATE TABLE hotels (
       id SERIAL PRIMARY KEY,
       name VARCHAR(255),
       location VARCHAR(255),
       price_tier VARCHAR(50),
       booking_status VARCHAR(50)
   );

   INSERT INTO hotels (name, location, price_tier, booking_status)
   VALUES
   ('Seaside Inn', 'Miami', 'Midscale', 'Available'),
   ('Mountain View Lodge', 'Denver', 'Upscale', 'Booked'),
   ('Cityscape Hotel', 'New York', 'Luxury', 'Available');
   ```

### Phase 2: MCP Toolbox Configuration (Tool Definition)

## Install MCP Toolbox
```bash
curl -O https://path.to/toolbox-binary
chmod +x toolbox
```


## Define Tools (tools.yaml)
```yaml
sources:
  my-cloud-sql-source:
    kind: cloud-sql-postgres
    # Add connection details: project, instance, user, password

tools:
  search-hotels-by-name:
    description: Search for hotels by name.
    statement: SELECT * FROM hotels WHERE name ILIKE '%' || $1 || '%';

  search-hotels-by-location:
    description: Search for hotels by location, sorted by price.
    statement: |
      SELECT * FROM hotels WHERE location ILIKE '%' || $1 || '%'
      ORDER BY CASE price_tier
        WHEN 'Midscale' THEN 1
        WHEN 'Upscale' THEN 2
        WHEN 'Luxury' THEN 3
      END;

toolsets:
  my_first_toolset:
    - search-hotels-by-name
    - search-hotels-by-location
```

## Run the MCP Server
```bash
./toolbox --tools_file "tools.yaml"
```

### Phase 3: ADK Agent Integration

Use the Agent Development Kit (ADK) to integrate Gemini with the MCP Toolbox.

Sample: agent.py
```python
from google.adk.agents import Agent
from toolbox_core import ToolboxSyncClient

# Connect to MCP Toolbox server
toolbox = ToolboxSyncClient("http://127.0.0.1:5000")

# Load defined toolset
tools = toolbox.load_toolset('my_first_toolset')

# Define Gemini-powered travel agent
root_agent = Agent(
    name="hotel_agent",
    model="gemini-2.5-flash",
    description="Agent to answer questions about hotels in a city or by name.",
    instruction="Always use the tools to ground responses in real data.",
    tools=tools,
)
```

### Phase 4: Testing & Deployment

# Local Testing
```bash
adk run
```

Run the agent and the MCP server together.

Verify tool-call generation, execution, and grounded responses.

# Deploy to Cloud Run (Optional)

Deploy MCP Toolbox server:
```bash
gcloud run deploy mcp-toolbox --source . --region=us-central1
```

Update agent.py with the Cloud Run URL.

# Deploy the agent:
```bash
adk deploy cloud_run
```

## 🧹 Cleanup

After testing, delete the following resources to avoid unnecessary costs:

Cloud SQL instance (hoteldb-instance)

Cloud Run services

Any associated storage or network resources

## 🔗 Reference

Original Codelab Source

For the full, detailed implementation steps and deep dives into the Gemini CLI integration, refer to the official codelab:

https://codelabs.developers.google.com/travel-agent-mcp-toolbox-adk#0

For detailed steps and official documentation, refer to:

- [Google Codelab: Travel Agent with MCP Toolbox & ADK](https://codelabs.developers.google.com/travel-agent-mcp-toolbox-adk#0)
- [Google Cloud SQL Documentation](https://cloud.google.com/sql/docs)
- [Agent Development Kit (ADK) Overview](https://cloud.google.com/vertex-ai/docs/agents/overview)
- [MCP Toolbox for Databases Guide](https://developers.google.com/mcp)
- [Gemini Models Overview](https://deepmind.google/technologies/gemini/)


## 🧠 Key Takeaways

The MCP Toolbox abstracts database logic into secure, callable tools.

The ADK Agent intelligently decides when to use these tools.

The Gemini model provides natural, context-aware reasoning grounded in real data.


