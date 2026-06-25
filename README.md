# Niagara AI Automation Sample Process 🤖

[![Niagara 4.14+](https://img.shields.io/badge/Niagara-4.14%2B-blue)](https://www.tridium.com)
[![AI: DeepSeek](https://img.shields.io/badge/AI-DeepSeek%20(tested)-success)](https://deepseek.com)
[![Free modules](https://img.shields.io/badge/Gline%20Modules-Free-brightgreen)](https://github.com/jason912)

> **Sample process: use AI + Gline modules (REST API, oBIX, FTP Server) to automate Niagara data point collection and web dashboard creation. A reference for integration projects.**

---

## What Is This?

> **Key enabler:** The Gline REST API can configure **Modbus client proxyExt addresses** (dataAddress, regType, dataType) remotely. This means AI can create a Modbus point AND set its register address in one automated flow — no manual Workbench steps required for Modbus point configuration.

This repository demonstrates a **sample AI automation process** for Niagara 4 using Gline's free modules. It shows how AI can:

- Auto-discover Modbus devices and collect data points
- Create and configure points via REST API
- Generate PX/HTML web pages automatically
- Deploy and verify pages via FTP Server (auto-login via SCRAM, verify bindings, fix errors)

The entire workflow runs on **DeepSeek V4 Flash + OpenClaw** and costs only cents in AI tokens.

| API | Port | Auth | Cost |
|-----|:----:|:----:|:----:|
| **oBIX** | 80/443 | SCRAM-SHA-256 / HTTP Basic | 🆓 Free (built-in) |
| **Gline REST API** | Configurable (default 8081) | None (LAN) | 🆓 Free (no SMA required) |

The AI (tested on **DeepSeek**) acts as a Niagara engineer — it reads the station structure, decides what operations to perform, then executes them via REST API and oBIX calls. No human in the loop.

---

## What AI Can Do Today

### ✅ Automated Point Collection

AI scans a Modbus device (or any protocol), reads the register map, and creates all necessary points automatically:

1. AI sends `LIST /Drivers/` via REST API to understand current station structure
2. AI determines the correct point types (NumericWritable, BooleanWritable, etc.)
3. AI batch-creates points via `ADD` operations
4. AI configures each point's Modbus register address, data type, and function code via oBIX `PUT`

### ✅ Automated Point Linking

After creating points, AI binds them to PX visualization pages:

1. AI generates the PX XML template with proper bindings
2. AI writes the XML to station via `StringToFile` module
3. AI establishes `Link` connections between points and UI components
4. Result: a fully functional web dashboard — zero manual binding

### ✅ Automated Module Addition

AI can deploy new modules and services to the station:

1. Copy JAR to modules directory
2. AI creates the required service component via `ADD`
3. AI configures service parameters via `MOD`
4. AI verifies the service is running via `READ`

### ✅ Automated Alarm & History Queries

AI reads alarms and historical data on demand:

- **Alarms**: `alarm:|bql:select ... where alarmClass = 'Red_High'`
- **History**: `history:|slot:/History/xxx|bql:select ... where timestamp > '...'`
- **Real-time points**: `station:|slot:/Drivers/...|bql:select name,out.value,out.status`
- Results returned as structured JSON

---

## Technical Flow

```
┌──────────────────────────────────────────────────────┐
│                    AI (DeepSeek)                      │
│   Understands Niagara structure, decides operations   │
└──────────┬───────────────────────────────┬───────────┘
           │                               │
     REST API (8081)                  oBIX (80/443)
     ┌──────────┐                  ┌──────────────┐
     │ LIST     │                  │ GET /out     │
     │ ADD      │                  │ PUT /address │
     │ REMOVE   │                  │ PUT /regType │
     │ RENAME   │                  │ Link create  │
     │ DUPLICATE│                  │ Alarm query  │
     │ LINK     │                  │ History query│
     │ MOD      │                  └──────────────┘
     │ READ     │
     └──────────┘
           │                               │
           └───────────┬───────────────────┘
                       ▼
           ┌─────────────────────┐
           │   Niagara Station   │
           │ - Points created    │
           │ - Links established │
           │ - PX pages deployed │
           │ - Alarms monitored  │
           └─────────────────────┘
```

### Step-by-Step Example: AI Adds a Modbus Thermostat

| Step | Operation | API | Description |
|:----:|-----------|:---:|-------------|
| 1 | `LIST /Drivers` | REST | AI scans existing drivers and services |
| 2 | `ADD ModbusTcpNetwork` | REST | Creates Modbus TCP network if not present |
| 3 | `MOD ipAddress=192.168.2.140` | REST | Configures device IP address |
| 4 | `CP template→6 points` | REST | Copies 6 point templates from a pre-configured source |
| 5 | `RENAME each point` | REST | Names: Set_Temp, Zone_Temp, Fan_Speed, On_Off, Filter_Status, Supply_Air_T |
| 6 | `PUT proxyExt/address` | oBIX | Sets Modbus register addresses for each point |
| 7 | `PUT proxyExt/regType` | oBIX | Sets holding/input/coil/discrete for each point |
| 8 | `GET /out` | oBIX | Verifies all points are reading live values |
| 9 | Generate PX XML | AI | AI creates a PX page with bound labels |
| 10 | `StringToFile write PX` | REST | Deploys the PX page to the station |
| 11 | Browser open | Web | Real-time dashboard is live |

---

## Real-World Case Study: JCI T8600 Modbus Thermostat

This is our flagship AI automation demonstration — a fully autonomous integration of a **Johnson Controls T8600 Modbus thermostat** into a Niagara 4.14 station.

### Tools Used

| Tool | Role |
|------|------|
| **DeepSeek V4 Flash** | AI reasoning engine |
| **OpenClaw** | AI agent framework |
| **Gline REST API** | Component lifecycle (ADD, MOD, CP, RENAME, LINK) |
| **oBIX** | Configure proxyExt (register address, type, function code) |
| **Gline FTP Server** | Deploy generated PX/HTML pages to station |

### The AI Workflow (Fully Autonomous)

```
Step 1  User tells AI: "Integrate JCI T8600 thermostat at 192.168.x.x"
         ↓
Step 2  AI searches the internet for T8600 Modbus register map
         (discovers holding/input/coil/discrete registers automatically)
         ↓
Step 3  AI connects to the Niagara station via REST API
         ↓
Step 4  AI scans existing Modbus network - finds the thermostat device
         ↓
Step 5  AI adds a new Device under the Modbus network via REST API ADD
         ↓
Step 6  AI creates ModbusClient points with correct register addresses
         (Set_Temp, Zone_Temp, Fan_Speed, On_Off, Filter_Status, Supply_Air_T)
         ↓
Step 7  AI configures each point's proxyExt via oBIX PUT
         (dataAddress, regType, dataType)
         ↓
Step 8  AI names all points properly via REST API RENAME
         ↓
Step 9  AI generates a PX dashboard (or BajaScript HTML page)
         with bound labels showing all 6 point values
         ↓
Step 10 AI uploads the page to station via FTP Server
         ↓
Step 11 AI authenticates via SCRAM (same as a human logging into the web UI),
         inspects the PX/HTML file, checks bindings, verifies data displays correctly
         ↓
Step 12 If errors found, AI generates a corrected page and re-uploads via FTP Server
         ↓
        ✅ Complete! Real-time dashboard live, remote writable
```

### What This Means

| Before AI Automation | After AI Automation |
|----------------------|---------------------|
| Manual Modbus register mapping | AI searches spec sheets automatically |
| Manual point creation (drag-drop in Workbench) | AI creates via REST API in milliseconds |
| Manual proxyExt configuration | AI configures via oBIX in seconds |
| Manual PX binding and UI design | AI generates PX/HTML pages autonomously |
| Manual FTP upload and testing | AI uploads, verifies, and fixes errors |
| **Hours to days** per device | **Minutes** per device |
| **$500-$2,000+** in engineering labor | **$0.03-0.05** in AI token cost |

### Cost

On **DeepSeek V4 Flash**, the entire T8600 integration consumes approximately **a few cents (USD)** in AI tokens. The same work traditionally requires hours of a Niagara engineer's time.

> **The bottom line:** Point binding, UI creation, and Modbus device integration are no longer time-consuming tasks. AI handles the entire pipeline — from register discovery to live dashboard deployment.

---

## Pricing

| Service | Price | Notes |
|---------|:-----:|-------|
| AI automation pipeline (current capabilities) | **Free** | Tested and working on DeepSeek |
| Custom AI automation development | Contact us | Tailored to your specific station and workflow |

> The current capability is offered free of charge — we want to demonstrate what's possible. If you see value and want us to build a custom automation pipeline for your Niagara system, we'd be happy to discuss.

---

> **Need a custom REST API for your AI workflow?** The free Gline REST API covers common operations. If you need custom endpoints, advanced security, or specialized data pipelines for AI automation — we can build it. Contact us for a discussion.

## Requirements

| Component | Requirement |
|-----------|-------------|
| **Niagara** | 4.14 or later |
| **Gline REST API** | Free module — [get it here](https://github.com/jason912/Niagara-Rest-API-service---Free) |
| **oBIX** | Built-in (enabled by default) |
| **AI Model** | DeepSeek (tested) or any OpenAI-compatible model |
| **Network** | AI host must reach station REST API and oBIX ports |

---

## Want to Collaborate?

If you're interested in AI-powered Niagara automation for your projects:

- We can build custom pipelines for your specific use case
- We can deploy the AI agent inside your network
- We can integrate with your existing tools and workflows

**Contact**: [jason.zhang@gline-net.com](mailto:jason.zhang@gline-net.com) | WhatsApp: +86 1380 190 9968

---

© 2026 Shanghai Gline Net Co., Ltd. All rights reserved.
