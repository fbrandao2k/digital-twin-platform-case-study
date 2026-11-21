# Architecture

This page expands the overview diagram in the [README](../README.md#architecture). It covers how the viewer is put together, how access is controlled, how the back end reaches Google Sheets without a stored key, and how a change travels to production.

## System overview

```mermaid
flowchart LR
    subgraph Browser["Browser: React + TypeScript + Vite"]
        Pages["Pages<br/>Projects · Users · Map · Viewer"]
        Viewer["BIM viewer<br/>That Open Engine · web-ifc · Three.js"]
        Globe["Globe<br/>CesiumJS · 3D Tiles"]
        Agent["AI agent panel<br/>user's own key"]
    end

    subgraph Firebase["Firebase (separate dev and production projects)"]
        Auth["Authentication<br/>Google · email"]
        Firestore[("Cloud Firestore<br/>users · projects · access grants")]
        Storage[("Cloud Storage<br/>IFC · FEM · damage · reports")]
        Functions["Cloud Functions<br/>Sheets · map assistant"]
        Rules{{"Security rules<br/>roles and per-bridge grants"}}
    end

    Pages --> Auth
    Viewer --> Storage
    Pages --> Firestore
    Firestore -.-> Rules
    Storage -.-> Rules
    Pages --> Functions
    Functions -->|"short-lived token"| Sheets["Google Sheets and Drive"]
    Functions -->|"quota-bound fallback"| Claude["Claude API"]
    Agent --> Claude
    Globe --> Ion["Cesium ion"]
    Viewer -.->|"MQTT over WSS, under review"| IoT["AWS IoT Core<br/>live sensor readings"]
```

The browser talks to Firebase directly. Firestore and Storage requests are checked by the security rules, so the rules, not the interface, decide what each user can read or write. Cloud Functions handle the work that needs server-side identity: calling Google APIs and the optional Claude fallback of the map assistant. The viewer's AI agent calls Claude straight from the browser with the user's own key. The dashed link is the live-sensor feed, still under review: the viewer subscribes to AWS IoT Core with the user's Firebase ID token, and the broker grants subscribe-only access.

## Viewer composition

```mermaid
flowchart TB
    Layout["Project viewer<br/>grid layout · side panels · viewport toolbar"]

    subgraph Panels["Panels"]
        direction LR
        Models["Models and<br/>selection data"]
        Views["Smart Views<br/>and queries"]
        Gis["GIS layers"]
        Clouds["Point clouds<br/>and meshes"]
        Fem["AnalyticalView<br/>FEM modes"]
        Damage["Damage<br/>Assessment"]
        Risk["Risk"]
        Sensors["IoT/Sensors"]
        SheetsPanel["Sheets"]
        AgentPanel["AI agent"]
        Perms["Permissions"]
        Todos["To-dos"]
    end

    subgraph Core["Shared viewer state"]
        direction LR
        ThatOpen["That Open components<br/>world · camera · IFC loader · fragments"]
        Custom["Custom components<br/>SmartViews · DataEnhancer · GisLayers · MCP · AIAgent"]
    end

    Layout --> Panels
    Panels --> Core
    Custom --> ThatOpen
```

Every panel works on the same That Open world, so a selection made in one panel (an IFC element, a FEM element, a crack) is the selection the others see. Custom components extend That Open Engine for the lab's needs: coloring by query, joining external data to IFC elements, placing the model on GIS layers, and exposing viewer actions to the AI agent and the MCP server. Panels clean up after themselves: the sensors overlay, for example, disposes its markers, animation callback and event listeners when the user switches projects.

## Access control

```mermaid
flowchart LR
    SignIn["First sign-in"] --> Profile[("User profile<br/>role: Viewer")]
    Profile -->|"promoted by an Administrator"| Admin["Administrator<br/>read and write all bridges<br/>manage roles and grants"]
    Profile --> Viewer["Viewer<br/>read the catalog and projects"]
    Viewer --> ReportGrant{"Report grant<br/>for this bridge?"}
    Viewer --> DamageGrant{"Damage grant<br/>for this bridge?"}
    ReportGrant -->|"yes"| Reports["Read this bridge's<br/>inspection reports"]
    DamageGrant -->|"yes"| DamageData["Read this bridge's<br/>damage data"]
```

- A new account always starts as a Viewer. The rules trust only the role already stored in the profile, never a role sent with a request.
- Report and damage grants are separate records per bridge and per user. Neither allows uploads, edits or project administration, and a grant for one bridge says nothing about another.
- Storage follows the same model: each bridge's files sit under their own path, and its damage files require that bridge's damage grant.
- The rules are tested on every pull request against local Firestore and Storage emulators with synthetic users.

## Google Sheets without a stored key

```mermaid
sequenceDiagram
    autonumber
    participant B as Browser (signed-in user)
    participant F as Callable Cloud Function
    participant IAM as IAM Credentials API
    participant G as Sheets and Drive APIs
    B->>F: Request with the user's Firebase ID token
    F->>F: Check the caller's stored role
    F->>IAM: Runtime identity asks for a short-lived token for the Sheets service account
    IAM-->>F: Short-lived access token
    F->>G: Call with the impersonated token
    G-->>F: Only spreadsheets shared with that account
    F-->>B: Result
```

The function never holds a downloaded key. Its runtime identity has permission to create tokens for one dedicated service account in the same project, and the function refuses an account from any other project. Signing in to the app with Google does not expose the user's own Drive: the app reaches only the spreadsheets explicitly shared with the service account.

## From a change to production

```mermaid
flowchart LR
    Branch["Short-lived branch"] --> PR["Pull request"]
    PR --> CI{"GitHub Actions<br/>types · lint · tests · build<br/>Functions build<br/>rules on emulators"}
    CI -->|"pass"| Merge["Merge"]
    Merge --> Dev["Deploy to development<br/>browser checks"]
    Dev --> Prod["Production release<br/>1. rules<br/>2. Functions<br/>3. Hosting"]
```

A merged pull request does not publish anything. Hosting, rules and Functions each have their own explicit deployment command per environment, and production releases follow a fixed order so that the security rules are always in place before the code that depends on them.
