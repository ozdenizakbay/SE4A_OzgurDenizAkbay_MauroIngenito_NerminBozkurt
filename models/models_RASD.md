# Models RASD

Mermaid code - Class Diagram:

classDiagram
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
    class User {
        +String userID
        +String name
        +String email
        +createRoutine()
        +makeBooking()
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
    class WorkoutRoutine {
        +String routineID
        +String name
        +Int totalDuration
    }
    class MachineType {
        +String typeID
        +String categoryName
        +Int avgExerciseDuration
    }
    class Machine {
        +String machineID
        +String status
    }

    GymOwner "1" -- "1..*" Gym : manages >
    Gym "1" *-- "1..*" Machine : contains >
    Machine "1..*" -- "1" MachineType : categorized as >
    User "1" -- "0..*" WorkoutRoutine : creates >
    User "1" -- "0..*" Booking : makes >
    Booking "0..*" -- "1" WorkoutRoutine : executes >
    WorkoutRoutine "1..*" -- "1..*" MachineType : requires >

---------------------------------------------------------------

Mermaid Code - Sequence Diagram (Routine Booking):

sequenceDiagram
    participant User
    participant System
    participant Database
    User->>System: Selects Routine and Time Slot
    System->>Database: Query operational machines (M) & avg duration
    Database-->>System: Return MachineType metrics
    System->>Database: Query aggregated demand for time slot
    Database-->>System: Return current occupancy
    System->>System: Calculate C_max and evaluate availability
    System-->>User: Display Occupancy & Expected Wait
    
    alt User accepts wait time
        User->>System: Confirm Booking
        System->>Database: Save Booking Record
        System-->>User: Confirmation Notification
    else User declines wait time
        User->>System: Cancel Booking Request
        System-->>User: Return to Dashboard
    end

---------------------------------------------------------------

Mermaid Code - Sequence Diagram (Machine Lifecycle Management):

sequenceDiagram
    participant Owner
    participant System
    participant Database
    participant User
    Owner->>System: Update Physical Machine Status (Broken)
    System->>Database: Save Machine Status & decrement M
    System->>Database: Fetch upcoming bookings for MachineType
    Database-->>System: Return bookings list
    System->>System: Recalculate C_max with new M
    
    opt Demand > New C_max
        System->>Database: Invalidate bookings exceeding new C_max
        System-->>User: Send Cancellation & Reschedule Notification
    end
    
    System-->>Owner: Confirmation of Update

---------------------------------------------------------------

Mermaid Code - Sequence Diagram (Routine Optimization):

sequenceDiagram
    participant User
    participant System
    participant Database
    User->>System: Request Routine during Peak Hour
    System->>Database: Query MachineType capacity (C_max) & demand
    Database-->>System: Return High Load Data
    System->>System: Detect Demand > C_max (Bottleneck)
    System->>System: Calculate Optimization Logic (Time/Order)
    System-->>User: Suggest Alternative Slot or Routine Order
    
    alt User Accepts Suggestion
        User->>System: Accept Suggestion
        System->>Database: Update and Save Booking
        System-->>User: Booking Confirmed
    else User Rejects Suggestion
        User->>System: Decline Suggestion
        System-->>User: Abort Booking / Return to Selection
    end

---------------------------------------------------------------

Mermaid Code - Machine FSM:

stateDiagram-v2
    [*] --> Available
    Available --> Broken : Damage Reported
    Available --> UnderMaintenance : Service Start
    Broken --> Available : Repaired
    UnderMaintenance --> Available : Service Complete

---------------------------------------------------------------

Mermaid Code - Booking FSM:

stateDiagram-v2
    [*] --> Pending
    Pending --> Confirmed : Validation Success
    Pending --> Cancelled : User Cancel
    Pending --> Cancelled : Capacity Exceeded (M Reduced)
    Confirmed --> Completed : Time Ends
    Confirmed --> Cancelled : Capacity Exceeded (M Reduced)
    Confirmed --> Cancelled : User Cancel