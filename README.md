# Niagara AI Automation — oBIX + REST API 🤖

[![Niagara 4.14+](https://img.shields.io/badge/Niagara-4.14%2B-blue)](https://www.tridium.com)
[![AI: DeepSeek](https://img.shields.io/badge/AI-DeepSeek%20(tested)-success)](https://deepseek.com)
[![Free to use](https://img.shields.io/badge/Free%20to%20use-brightgreen)](LICENSE)

> **Use AI to automate your Niagara station — point creation, linking, module deployment, alarm analysis, and history queries. Zero coding, zero license cost.**

---

## What Is This?

We have built and tested an **AI-powered automation pipeline** for Niagara 4 that combines two free APIs:

| API | Port | Auth | Cost |
|-----|:----:|:----:|:----:|
| **oBIX** | 80/443 | SCRAM-SHA-256 / HTTP Basic | 🆓 Free (built-in) |
| **Gline REST API** | 8081 | None (LAN) | 🆓 Free (no SMA required) |

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

AI can deploy and wire new modules into your station program - with one prerequisite.

**Prerequisite:** Drag the required component(s) from the Workbench Palette into a wiresheet manually once. This gives the station a "template" of the module.

**Then AI handles the rest:**

1. AI finds the template component in the wiresheet
2. AI copies it via `DUPLICATE` / `CP` operation
3. AI pastes it into the correct folder location
4. AI configures parameters via `MOD`
5. AI creates Links to wire it into the program logic
6. AI verifies the module is running via `READ`

> **Current limitation:** AI cannot drag components from the Palette like Workbench does. It can only copy, move, configure, and wire existing components. Think of it as a "copy + configure" assistant rather than a "drag from scratch" tool.

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

## Real-World Result

The process above was tested on a **T8600 Modbus thermostat** connected to a Niagara 4.14 station:

- **6 register types** (holding, input, coil, discrete) auto-detected and configured
- **6 points** created, named, and linked in under 2 minutes
- **PX dashboard** deployed and displaying live data
- **Remote write** (set temperature, fan speed, on/off) working through browser
- **Zero manual operations** — AI completed the entire integration autonomously

---

## Pricing

| Service | Price | Notes |
|---------|:-----:|-------|
| AI automation pipeline (current capabilities) | **Free** | Tested and working on DeepSeek |
| Custom AI automation development | Contact us | Tailored to your specific station and workflow |

> The current capability is offered free of charge — we want to demonstrate what's possible. If you see value and want us to build a custom automation pipeline for your Niagara system, we'd be happy to discuss.

---

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
