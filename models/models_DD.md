# Models DD

Mermaid Code - Component Diagram:

graph TD
    subgraph Client ["Client Tier (Web Browser)"]
        spa["Responsive Web Application (Flutter Web)"]
        userUI["User Interface Module"]
        ownerUI["Owner Dashboard Module"]
        spa --- userUI
        spa --- ownerUI
    end

    subgraph Server ["Application Tier (Node.js)"]
        gateway["API Gateway"]
        auth["Auth Manager"]
        booking["Booking Engine"]
        capacity["Capacity & Optimization Engine"]
        lifecycle["Lifecycle Manager"]
    end

    subgraph Data ["Data Tier"]
        db[("PostgreSQL Database")]
    end

    %% Client to Server Connections
    spa --> gateway

    %% Internal Server Routing
    gateway --> auth
    gateway --> booking
    gateway --> lifecycle

    %% Internal Server Logic
    booking <--> capacity
    lifecycle --> capacity

    %% Server to Database Connections
    auth --> db
    booking --> db
    capacity --> db
    lifecycle --> db

    classDef tier fill:#f9f9f9,stroke:#333,stroke-width:2px;
    class Client,Server,Data tier;

    ---------------------------------------------------------------

    Mermaid Code - Deployment Diagram:

    graph TD
    subgraph ClientDevice ["Client Device (Mobile, Tablet, or PC)"]
        browser[<<artifact>> Web Browser]
        spa[<<artifact>> Q-OP Web App]
        browser --- spa
    end

    subgraph CloudServer ["Application Server (Node.js Environment)"]
        gateway[<<artifact>> API Gateway]
        booking[<<artifact>> Booking Engine]
        capacity[<<artifact>> Capacity Engine]
    end

    subgraph DBServer ["Database Server (Managed Cloud)"]
        db[<<artifact>> PostgreSQL DBMS]
    end

    %% Network Connections
    ClientDevice -- "HTTPS (REST)" --> CloudServer
    CloudServer -- "TCP/IP" --> DBServer

    classDef node fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    class ClientDevice,CloudServer,DBServer node;

    ---------------------------------------------------------------

    Mermaid Code - Runtime Optimization:

    sequenceDiagram
    participant UI as User Interface Module
    participant API as API Gateway
    participant BE as Booking Engine
    participant CE as Capacity & Optimization Engine
    participant DB as PostgreSQL Database

    UI->>API: POST /api/bookings (Routine, TimeSlot)
    API->>BE: routeRequest()
    BE->>CE: calculateAvailability(MachineTypes, TimeSlot)
    
    CE->>DB: SELECT operational_machines (M)
    DB-->>CE: return M
    CE->>DB: SELECT current_demand
    DB-->>CE: return Demand
    
    CE->>CE: Calculate C_max (Demand > C_max)
    
    rect rgb(255, 243, 224)
        Note right of CE: Active Search Loop Initiated
        CE->>DB: SELECT demand for adjacent slots (+/- 30 mins)
        DB-->>CE: return Adjacent Demand
        CE->>CE: Calculate C_max for adjacent slots
        CE->>CE: Identify optimal alternative slot
    end
    
    CE-->>BE: return OptimizationSuggestion (Alternative Slot)
    BE-->>API: 409 Conflict + Suggestion Payload
    API-->>UI: Display Alternative Slot Prompt

    --------------------------------------------------------------- 

    Mermaid Code - Runtime Lifecycle:

    sequenceDiagram
    participant UI as Owner Dashboard Module
    participant API as API Gateway
    participant LM as Lifecycle Manager
    participant CE as Capacity & Optimization Engine
    participant BE as Booking Engine
    participant DB as PostgreSQL Database

    UI->>API: PUT /api/machine/status (Broken)
    API->>LM: routeUpdateRequest()
    LM->>DB: updateMachineStatus() & decrement M
    DB-->>LM: confirm update
    
    LM->>CE: triggerCapacityReview(MachineType)
    CE->>DB: SELECT upcoming_bookings for MachineType
    DB-->>CE: return Bookings List
    CE->>CE: Recalculate C_max with new M
    
    opt Demand > New C_max
        CE->>BE: requestCancellation(Overbooked Sessions)
        BE->>DB: updateBookingStatus(Cancelled)
        BE->>API: triggerUserNotification()
    end
    
    LM-->>API: 200 OK
    API-->>UI: Display Success Message