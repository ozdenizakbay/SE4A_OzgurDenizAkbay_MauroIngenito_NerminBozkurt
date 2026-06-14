Mermaid code - Class Diagram:

classDiagram
    direction TB

    class GymOwner {
        +String ownerID
        +String name
        +String contactInfo
        +registerGym()
        +updateMachineStatus()
    }

    class Gym {
        +String gymID
        +String location
        +Int capacity
    }

    class Machine {
        +String machineID
        +String type
        +String status
    }

    class User {
        +String userID
        +String name
        +String email
        +createRoutine()
        +makeBooking()
    }

    class WorkoutRoutine {
        +String routineID
        +String name
        +Int estimatedDuration
    }

    class Booking {
        +String bookingID
        +Date date
        +Time startTime
        +Time endTime
        +String status
        +confirm()
        +cancel()
    }

    %% Relationships
    GymOwner "1" -- "1..*" Gym : manages >
    Gym "1" *-- "1..*" Machine : contains >
    User "1" -- "0..*" WorkoutRoutine : creates >
    WorkoutRoutine "1..*" -- "1..*" Machine : requires >
    User "1" -- "0..*" Booking : makes >
    Booking "1..*" -- "1" WorkoutRoutine : executes >

---------------------------------------------------------------

Mermaid Code - Sequence Diagram (Routine Booking):

sequenceDiagram
    participant User
    participant System
    participant Database

    User->>System: Selects Routine and Time Slot
    System->>Database: Query Machine Occupancy
    Database-->>System: Return Availability Data
    System->>System: Calculate Predicted Waiting Time
    System-->>User: Display Occupancy & Alternatives
    User->>System: Confirm Booking
    System->>Database: Save Booking Record
    System-->>User: Confirmation Notification

---------------------------------------------------------------

Mermaid Code - Sequence Diagram (Machine Lifecycle Management):

sequenceDiagram
    participant Owner
    participant System
    participant Database

    Owner->>System: Update Machine Status (Broken)
    System->>Database: Save Machine Status
    System->>Database: Fetch Bookings involving Machine
    Database-->>System: Return affected Bookings
    System->>Database: Invalidate/Cancel Bookings
    System-->>Owner: Confirmation of Update
    System-->>Database: Send Notification to affected Users

---------------------------------------------------------------

Mermaid Code - Sequence Diagram (Routine Optimization):

sequenceDiagram
    participant User
    participant System
    participant Database

    User->>System: Request Routine during Peak Hour
    System->>Database: Query Real-time Occupancy
    Database-->>System: Return High Load Data
    System->>System: Calculate Optimization Logic
    System-->>User: Suggest Alternative Slot/Order
    User->>System: Accept Suggestion
    System->>Database: Update Booking

---------------------------------------------------------------

Mermaid Code - Machine FSM:

stateDiagram-v2
    [*] --> Available
    Available --> Occupied: Booked
    Occupied --> Available: Slot Ends
    Available --> Broken: Damage Reported
    Occupied --> Broken: Damage Reported
    Broken --> Available: Repaired
    Available --> UnderMaintenance: Service Start
    UnderMaintenance --> Available: Service Complete

---------------------------------------------------------------

Mermaid Code - Booking FSM:

stateDiagram-v2
    [*] --> Pending
    Pending --> Confirmed: Validation Success
    Pending --> Cancelled: Machine Broken
    Pending --> Cancelled: User Cancel
    Confirmed --> Completed: Time Ends
    Confirmed --> Cancelled: Machine Broken