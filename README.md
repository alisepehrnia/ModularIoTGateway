# 🌐 IoTGateway – A Modular IoT Gateway (.NET 5)

**BSc final project:** a modular, extensible **IoT gateway** that accepts device data over **MQTT and HTTP**, decodes it into a common message format, runs it through a **pluggable processing pipeline**, and persists everything to **MongoDB** — all configurable through a REST API.

> **Repository description:** *“BSc Final Project (Modular IoT Gateway)”* [reference:0]
> **Stack:** C# · .NET 5 · ASP.NET Core · MQTTnet · MongoDB · Swagger/OpenAPI

---

## 📖 Overview

`IoTGateway` is a **host application** built on ASP.NET Core that listens on two protocols **in the same process**:

| Port | Protocol | Purpose |
|------|----------|---------|
| **1883** | MQTT | Embedded MQTT broker (via `MQTTnet.AspNetCore`) — devices publish here |
| **5000** | HTTP | REST API + Swagger UI — for configuration and management |

Incoming device traffic is normalised into `Message` objects, decoded by the **Message Conversion (MC)** layer, and passed through a **module pipeline (MP)** where each module implements `IModule` and can inspect, transform, or act on the message. The **DB** layer stores devices, hubs, and servers in MongoDB.

The “modular” part is the design goal: the gateway core is a class library (`IoTGatewayCore`) with **no dependency on the web host**, so new connection providers or processing modules can be added without touching the ASP.NET Core project.

---

## ✨ Features

- **Dual-protocol ingress** — MQTT broker on `:1883` and HTTP API on `:5000` from a single Kestrel host [reference:1].
- **Embedded MQTT server** — `MQTTnet` with persistent sessions, a 10-second communication timeout, and client ID `IoTGateway-MqttServer` [reference:2].
- **Pluggable processing pipeline** — `IModule` contract with example implementations (`LoopbackModule`, `ModuleA`) and a `Processor` that orchestrates them [reference:3].
- **Message decoding layer** — dedicated converters for different message/transport shapes (`DMC`, `HMC`, `SMC`, `Decoder`, `ToExternal`) [reference:4].
- **Swagger / OpenAPI** — interactive API docs at `/swagger` in Development [reference:5][reference:6].
- **MongoDB persistence** — device, hub, and server repositories behind provider interfaces [reference:7].
- **Configuration through JSON** — `config.json` binds to a strongly-typed `AppConfig` section [reference:8][reference:9].
- **Documented design** — architecture, DFD, use-case diagrams and a full Persian-language BSc report in `docs/` [reference:10].

---

## 🏗️ Architecture

```text
        Devices (MQTT :1883)                Devices / Clients (HTTP :5000)
                 │                                      │
                 ▼                                      ▼
        ┌─────────────────────────────────────────────────────┐
        │  IoTGateway  (ASP.NET Core host, Kestrel)           │
        │  ├─ MQTT pipeline  (MQTTnet.AspNetCore)             │
        │  └─ REST controllers (Device / Hub / Server / Test) │
        └────────────────────────┬────────────────────────────┘
                                 │
                                 ▼
        ┌─────────────────────────────────────────────────────┐
        │  IoTGatewayCore  (class library)                    │
        │                                                     │
        │  Connection/   HttpProvider · MqttProvider · MS     │
        │  MC/           Decoder · DMC · HMC · SMC            │
        │  MP/           IModule · Processor · Security ·     │
        │                Profiler · Terminal · Modules        │
        │  IS/           DeviceIS · HubIS · ServerIS          │
        │  DB/           DBProvider · DeviceDB · HubDB ·      │
        │                ServerDB                             │
        │  Models/       Device · Hub · Server · Transceiver  │
        └────────────────────────┬────────────────────────────┘
                                 │
                                 ▼
                            MongoDB
                   (localhost:27017 / IoTGateway)
```

## 🗂️ Repository Structure

```text
IoTGateway/
├── docs/                          # Project documentation
│   ├── Diagrams/
│   │   ├── arch.vsdx              # Architecture diagram
│   │   ├── dfd.vsdx               # Data-flow diagram
│   │   ├── dfd-activity.vsdx      # DFD activity diagram
│   │   ├── useCase.vsdx           # Use-case diagram
│   │   ├── DataModelsClassDiagram.png
│   │   ├── CodeMap1.dgml          # Visual Studio code map
│   │   ├── Output/                # Exported diagram images
│   │   └── Present/               # Presentation assets
│   ├── Intro.docx / Intro.pdf     # Project introduction
│   ├── دفاع.pptx                   # Defense presentation (Persian)
│   ├── گزارش پروژه کارشناسی.docx    # BSc project report (Persian)
│   └── گزارش پروژه کارشناسی (پیشنویس).pdf  # Report draft PDF
├── src/
│   ├── IoTGateway.sln
│   ├── IoTGateway/                # ASP.NET Core host (net5.0)
│   │   ├── Controllers/
│   │   │   ├── DeviceController.cs
│   │   │   ├── HubController.cs
│   │   │   ├── ServerController.cs
│   │   │   └── TestController.cs
│   │   ├── Properties/
│   │   ├── Program.cs             # Kestrel + MQTT/HTTP endpoints
│   │   ├── Startup.cs             # DI, MQTT server, Swagger
│   │   ├── appsettings.json
│   │   ├── appsettings.Development.json
│   │   ├── config.json            # AppConfig (MongoDB settings)
│   │   └── IoTGateway.csproj      # net5.0
│   └── IoTGatewayCore/            # Core library (net5.0)
│       ├── Connection/            # HttpProvider, MqttProvider, MS, _archive/
│       ├── DB/                    # DBProvider, DeviceDB, HubDB, ServerDB
│       ├── IS/                    # DeviceIS, HubIS, ServerIS, IS, dis, sis
│       ├── MC/                    # Decoder, DMC, HMC, SMC, Helper, ToExternal
│       ├── MP/                    # IModule, Processor, Security, Profiler,
│       │                          #   Terminal, LoopbackModule, ModuleA
│       ├── Models/
│       │   ├── Config/            # AppConfig etc.
│       │   ├── Data/              # Message, MessageType, ConfigMessage(+Type)
│       │   ├── Object/            # Device, Hub, Server, Transceiver(+Type)
│       │   └── Database.cs
│       ├── U/                     # AppCS, CS, LS utilities
│       ├── CodeMap.dgml
│       └── IoTGatewayCore.csproj  # net5.0
├── .gitignore
└── README.md
```

> The web project file is `IoTGateway/IoTGateway.csproj` and targets `net5.0` ; the core library targets the same framework .[](https://raw.githubusercontent.com/alisepehrnia/IoTGateway/master/src/IoTGateway/IoTGateway.csproj)[](https://raw.githubusercontent.com/alisepehrnia/IoTGateway/master/src/IoTGatewayCore/IoTGatewayCore.csproj)

---
## 🧩 Core Components

### 1. Connection layer (`IoTGatewayCore/Connection`)

Transport-facing providers that talk to the outside world:

| File              | Role                                                             |
| ----------------- | ---------------------------------------------------------------- |
| `MqttProvider.cs` | MQTT client/server interaction for device data                   |
| `HttpProvider.cs` | HTTP-based device communication                                  |
| `MS.cs`           | Messaging / session helper                                       |
| `_archive/`       | Older/experimental connection code (not part of the active path) |

### 2. Message conversion (`IoTGatewayCore/MC`)

Turns raw device payloads into the gateway’s internal `Message` model.

| File                         | Likely role                                               |
| ---------------------------- | --------------------------------------------------------- |
| `Decoder.cs`                 | Base/entry decoding logic                                 |
| `DMC.cs`, `HMC.cs`, `SMC.cs` | Specific decoders (different message/transport kinds)     |
| `ToExternal.cs`              | Serialises internal messages back out to external formats |
| `Helper.cs`                  | Shared conversion helpers                                 |

### 3. Module pipeline (`IoTGatewayCore/MP`)

The extensibility point of the gateway.

| File                | Role                                                      |
| ------------------- | --------------------------------------------------------- |
| `IModule.cs`        | **The module contract** — implement this to add behaviour |
| `Processor.cs`      | Runs messages through the registered modules              |
| `LoopbackModule.cs` | Example module: echoes messages back                      |
| `ModuleA.cs`        | Second example module                                     |
| `Security.cs`       | Authentication / authorisation concerns                   |
| `Profiler.cs`       | Timing / performance instrumentation                      |
| `Terminal.cs`       | Operator-facing command/terminal handling                 |

### 4. Data access (`IoTGatewayCore/DB`)

| File            | Role                                |
| --------------- | ----------------------------------- |
| `DBProvider.cs` | MongoDB connection/session provider |
| `DeviceDB.cs`   | Device collection access            |
| `HubDB.cs`      | Hub collection access               |
| `ServerDB.cs`   | Server collection access            |

### 5. Service contracts (`IoTGatewayCore/IS`)

Interfaces (`DeviceIS`, `HubIS`, `ServerIS`, plus `IS`, `dis`, `sis`) that decouple controllers and modules from the concrete DB implementations.

### 6. Domain models (`IoTGatewayCore/Models`)

| Group        | Types                                                          |
| ------------ | -------------------------------------------------------------- |
| **Object**   | `Device`, `Hub`, `Server`, `Transceiver`, `TransceiverType`    |
| **Data**     | `Message`, `MessageType`, `ConfigMessage`, `ConfigMessageType` |
| **Config**   | `AppConfig` (bound from `config.json`)                         |
| **Database** | `Database.cs` (connection model)                               |

---
## 🚀 Getting Started

### Prerequisites

- **.NET 5 SDK** (the projects target `net5.0`)[](https://raw.githubusercontent.com/alisepehrnia/IoTGateway/master/src/IoTGateway/IoTGateway.csproj)[](https://raw.githubusercontent.com/alisepehrnia/IoTGateway/master/src/IoTGatewayCore/IoTGatewayCore.csproj)
- **MongoDB** running locally (default `localhost:27017`)
- **Visual Studio 2019/2022** or the `dotnet` CLI
- An **MQTT client** for testing (MQTT Explorer, `mosquitto_pub`, etc.) — optional

> ⚠️ .NET 5 is **out of support**. Building may require the .NET 5 targeting pack; upgrading to a supported LTS (e.g. .NET 8) is advisable.

### 1. Configure MongoDB

Edit `src/IoTGateway/config.json`:

```json
{
  "AppConfig": {
    "Database": {
      "Host": "localhost",
      "Port": 27017,
      "DatabaseName": "IoTGateway"
    }
  }
}
```

This is the default committed in the repository .[](https://raw.githubusercontent.com/alisepehrnia/IoTGateway/master/src/IoTGateway/config.json)
### 2. Restore and build

```bash
git clone https://github.com/alisepehrnia/IoTGateway.git
cd IoTGateway/src
dotnet restore
dotnet build
```
### 3. Run

```bash
cd IoTGateway
dotnet run
```

The host starts **both** listeners :[](https://raw.githubusercontent.com/alisepehrnia/IoTGateway/master/src/IoTGateway/Program.cs)
- `mqtt://localhost:1883` — MQTT broker endpoint
- `http://localhost:5000` — REST API

### 4. Explore the API

In the **Development** environment, Swagger is enabled :[](https://raw.githubusercontent.com/alisepehrnia/IoTGateway/master/src/IoTGateway/Startup.cs)

```text
http://localhost:5000/swagger
```

---
## 🔌 API Surface

The host exposes four controllers :[](https://github.com/alisepehrnia/IoTGateway/tree/master/src/IoTGateway/Controllers)

| Controller         | Likely responsibility                   |
| ------------------ | --------------------------------------- |
| `DeviceController` | CRUD / management of registered devices |
| `HubController`    | CRUD / management of hubs               |
| `ServerController` | CRUD / management of upstream servers   |
| `TestController`   | Diagnostic / test endpoints             |

Exact routes and verbs are best read from Swagger once the app is running, or from the controller sources — the repository does not include a static endpoint table.

### MQTT

The broker is configured with :[](https://raw.githubusercontent.com/alisepehrnia/IoTGateway/master/src/IoTGateway/Startup.cs)

- **Default endpoint** on port `1883`
- **Communication timeout:** 10 seconds
- **Persistent sessions** enabled
- **Client ID:** `IoTGateway-MqttServer`

Devices publish to this broker; the gateway's connection layer picks the traffic up and feeds it into the decoding/pipeline path.

---

## 📚 Documentation (`docs/`)

The repository ships a full academic documentation set :[](https://github.com/alisepehrnia/IoTGateway/tree/master/docs)[](https://github.com/alisepehrnia/IoTGateway/tree/master/docs/Diagrams)

| File                                           | Contents                          |
| ---------------------------------------------- | --------------------------------- |
| `Intro.docx` / `Intro.pdf`                     | Project introduction              |
| `گزارش پروژه کارشناسی.docx` / `(پیش‌نویس).pdf` | Full BSc project report (Persian) |
| `دفاع.pptx`                                    | Defense presentation (Persian)    |
| `Diagrams/arch.vsdx`                           | Architecture diagram              |
| `Diagrams/dfd.vsdx`, `dfd-activity.vsdx`       | Data-flow diagrams                |
| `Diagrams/useCase.vsdx`                        | Use-case diagram                  |
| `Diagrams/DataModelsClassDiagram.png`          | Class diagram of the data models  |
| `Diagrams/CodeMap1.dgml`                       | Visual Studio code map            |

> The `.vsdx` files require **Microsoft Visio** (or an importer such as [draw.io](https://draw.io/)); the `.dgml` file opens in Visual Studio’s **Code Map**.

---
## 🧠 Design Notes

- **Two projects, one direction of dependency.** `IoTGateway` → `IoTGatewayCore`. The core never references [ASP.NET](https://asp.net/) Core, so the gateway logic can be hosted elsewhere (console service, different web stack) without modification.
- **Singletons everywhere.** `Startup.ConfigureServices` registers the core services as singletons , which fits a long-running gateway with shared state (broker, DB provider, module pipeline).[](https://raw.githubusercontent.com/alisepehrnia/IoTGateway/master/src/IoTGateway/Startup.cs)
- **Interfaces over implementations.** The `IS/` folder defines contracts (`DeviceIS`, `HubIS`, `ServerIS`) that sit between controllers/modules and the `DB/` repositories — the classic seam for swapping storage.
- **Modules as the extension point.** Adding a capability means writing an `IModule` and registering it; `Processor` handles the invocation order.
- **Configuration is layered.** `config.json` is added explicitly in `Program.cs` on top of the default `appsettings.json`, and `AppConfig` is bound in `Startup` .[](https://raw.githubusercontent.com/alisepehrnia/IoTGateway/master/src/IoTGateway/Program.cs)[](https://raw.githubusercontent.com/alisepehrnia/IoTGateway/master/src/IoTGateway/Startup.cs)

---


- **.NET 5 is end-of-life.** The projects target `net5.0`; expect friction on modern SDKs and no security patches. Migrating to a supported LTS is the main modernisation task.
- **No test project.** The solution contains only the host and the core library — no unit or integration tests.
- **Documentation is primarily Persian.** The report and defense deck are in Farsi; the code and identifiers are English.    
- **Exact endpoint contracts are not documented statically.** Use Swagger at runtime or read the controllers.
- **`Connection/_archive/`** contains legacy code that is not part of the active connection path — don't mistake it for current behaviour.
- **No Docker/CI.** There are no container definitions or pipeline files; build and run are manual.

---
## 🛣️ Possible Next Steps

- Migrate to **.NET 8 (LTS)** and re-validate MQTTnet compatibility.
- Add **unit tests** around `MC` decoding and `MP` module execution.
- Document the REST contract with **Swagger examples** / OpenAPI annotations.
- Replace the `_archive` connection code with a clean provider interface.
- Add a **Docker Compose** file (gateway + MongoDB) for one-command startup.
- Add an **Apache-2.0 or MIT license** so others can reuse the design.

---
## 🤝 Contributing

This is an academic final project, but issues and pull requests that improve portability, testing, or documentation are welcome.

---
## 📄 License

Feel free to use :)
