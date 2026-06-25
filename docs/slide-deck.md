# Niagara AI Application - Slide Deck

> AI-powered Niagara automation via oBIX + REST API

---

## Slide 1: Niagara N4 API Ecosystem Comparison

Comparison of four API approaches for Niagara 4:
- oBIX (free, built-in)
- Gline REST API (free, no SMA required)
- Commercial solutions (BTIB, Tridium REST)
- Open source alternatives

---

## Slide 2: oBIX Protocol - Built-in, Free, Feature Complete

**Port:** 80/443 | **Auth:** SCRAM-SHA-256 / HTTP Basic | **Protocol:** REST + XML

**Capabilities:**
- Read point values: `GET /obix/config/.../out/`
- Write values: `POST .../set/`, `POST .../override/`, `POST .../auto/`
- History & Alarms: HistoryService, AlarmService (query + ACK)
- Create Links: `POST .../Link/` with auto type conversion
- Configure proxyExt: PUT address/regType/dataType

**Limitations:**
- Cannot create/delete components
- Cannot rename
- No batch operations

**Best for:** Daily read/write, link creation, alarm queries, Modbus address configuration

---

## Slide 3: Gline REST API (Port 8081) - Free, No SMA, Batch Management

**8 Operations:**

| Operation | Description |
|-----------|-------------|
| ADD | Create points (BooleanWritable, NumericWritable, EnumWritable, StringWritable) |
| REMOVE | Delete components |
| RENAME | Rename components |
| DUPLICATE | Copy points/templates (auto-adds _copy suffix) |
| LIST | List child components |
| LINK | Create connections |
| UNLINK | Remove connections |
| MOD | Modify property values |

**Batch example:**
```json
[
  {"OP":"ADD","PATH":"/Drivers/...","N":"P1","T":"control:NumericWritable"},
  {"OP":"RENAME","PATH":"/Drivers/.../P1_copy","N":"RoomTemp"},
  {"OP":"DUPLICATE","PATH":"/Drivers/.../RoomTemp"}
]
```

---

## Slide 4: Commercial Solutions

**BTIB ACTIVE-API ($462-$1,001):**
- Light: 500 points $462
- Normal: Unlimited $1,001
- Create/delete/copy components, read/write, history, alarms
- Swagger/Postman docs included

**Tridium Niagara REST API (requires SMA):**
- Full RESTful CRUD
- Access by ord path
- History queries
- Requires niagaraRestApi module + SMA license

---

## Slide 5: Full Comparison Matrix

Six solutions compared across: price, auth, read/write, create/delete, rename/copy, link management, history, alarms, SMA requirement.

**Recommendation:**
- Daily read/write -> oBIX (free, stable, verified)
- Batch point creation/template copy -> Gline REST API (free, JSON batch, no SMA)
- Large commercial projects -> BTIB ACTIVE-API ($462+, documented)
- Existing SMA + full REST -> Tridium Niagara REST API

---

## Slide 6: AI Automation Demo - T8600 Modbus Thermostat

![AI Modbus Point Collection](slide6_img0.png)

**AI automatically collected Modbus points:**

| Point Name | Register | Type | Value |
|------------|----------|------|-------|
| Set_Temp | 40001 (Holding) | NumericWritable | 25.0C |
| Zone_Temp | 30001 (Input) | NumericReadOnly | 24.3C |
| Fan_Speed | 40002 (Holding) | EnumWritable | High |
| On_Off | 00001 (Coil) | BooleanWritable | ON |
| Filter_Status | 10001 (Discrete) | BooleanReadOnly | Normal |
| Supply_Air_T | 30002 (Input) | NumericReadOnly | 22.8C |

![AI PX Binding Result](slide6_img1.png)

**AI workflow:**
REST API (scan Modbus) -> ADD (create points) -> MOD (configure registers) -> COPY (batch) -> RENAME (name) -> oBIX PUT (proxyExt) -> Link (bind to PX) -> Browser view result

- Zero manual binding
- Fully automated closed loop
- Remote write support (ON/OFF, SetTemp, FanSpeed)

---

## Slide 7: Technical Operation Flow

```
REST API 8081 (no auth):
  1. LIST - Scan structure
  2. ADD - Create network/device
  3. MOD - Configure parameters
  4. CP - Copy template points
  5. RENAME - Name points

oBIX PUT (80/443, Basic Auth):
  1. GET - Authenticate
  2. PUT addr - Write proxyExt address
  3. PUT regType - Configure register type
  4. GET verify - Read config and live values

Verification:
  REST API READ out - Check status (ok/stale/down)
  oBIX GET out/ - Detailed status with display and status info
```

---

## Slide 8: PX Binding & Deployment Flow

**PX XML Template Structure:**
```
px -> import (module declarations) -> content (ScrollPane -> CanvasPane -> components)
```

**Deployment via StringToFile:**
1. MOD -> inputString (POST XML content to REST API)
2. INVOKE execute -> Write file to station
3. Browser opens: http://station/obix/px?file=^Px/MyPage.px

**Known Pitfalls:**
- Use ObjectToString converter (INumericFormat causes HX Web crash)
- Use single quotes, not double quotes (double quotes eaten by REST API MOD)
- Must import converters module
- No XML declaration
- No emoji (4-byte UTF-8)
- No BOM in saved files

---

*Generated from Niagara_AI_Application_探讨.pptx - Slides 1-8*
*Shanghai Gline Net Co., Ltd. - www.glineweb.com*
