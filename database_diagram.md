# Database Diagram

This document provides a visual representation of the key tables in the ViciDial database and their relationships.

## Core Tables and Relationships

```mermaid
erDiagram
    vicidial_list ||--o{ vicidial_log : "lead_id"
    vicidial_list ||--o{ vicidial_closer_log : "lead_id"
    vicidial_list ||--o{ vicidial_agent_log : "lead_id"
    vicidial_log ||--o{ vicidial_agent_log : "uniqueid"
    vicidial_closer_log ||--o{ vicidial_agent_log : "uniqueid"
    vicidial_log ||--o{ recording_log : "vicidial_id"
    vicidial_closer_log ||--o{ recording_log : "vicidial_id"
    vicidial_log ||--o{ vicidial_call_notes : "vicidial_id"
    vicidial_closer_log ||--o{ vicidial_call_notes : "vicidial_id"
    vicidial_campaigns ||--o{ vicidial_log : "campaign_id"
    vicidial_campaigns ||--o{ vicidial_agent_log : "campaign_id"
    vicidial_users ||--o{ vicidial_agent_log : "user"
    vicidial_users ||--o{ vicidial_log : "user"
    vicidial_users ||--o{ vicidial_closer_log : "user"
    vicidial_users ||--o{ recording_log : "user"
    vicidial_lists ||--o{ vicidial_list : "list_id"
    vicidial_lists }|--|| vicidial_campaigns : "campaign_id"
    
    vicidial_list {
        int lead_id PK
        datetime entry_date
        varchar status
        varchar phone_number
        bigint list_id FK
        varchar user FK
    }
    
    vicidial_log {
        varchar uniqueid PK
        int lead_id FK
        varchar campaign_id FK
        datetime call_date
        varchar status
        varchar phone_number
        varchar user FK
    }
    
    vicidial_closer_log {
        int closecallid PK
        varchar uniqueid
        int lead_id FK
        varchar campaign_id
        datetime call_date
        varchar status
        varchar phone_number
        varchar user FK
    }
    
    vicidial_agent_log {
        int agent_log_id PK
        varchar user FK
        datetime event_time
        int lead_id FK
        varchar campaign_id FK
        varchar uniqueid FK
        varchar status
    }
    
    recording_log {
        int recording_id PK
        datetime start_time
        varchar filename
        varchar location
        int lead_id FK
        varchar user FK
        varchar vicidial_id FK
    }
    
    vicidial_call_notes {
        int notesid PK
        int lead_id FK
        varchar vicidial_id FK
        datetime call_date
        text call_notes
    }
    
    vicidial_campaigns {
        varchar campaign_id PK
        varchar campaign_name
        enum active
        varchar dial_method
    }
    
    vicidial_users {
        varchar user PK
        varchar full_name
        varchar user_group
    }
    
    vicidial_lists {
        bigint list_id PK
        varchar list_name
        varchar campaign_id FK
    }
```

## Call Flow Diagrams

### Outbound Call Flow

```mermaid
flowchart TD
    A[Lead in vicidial_list] -->|Dialed| B[Call recorded in vicidial_log]
    B -->|Answered| C[Agent activity in vicidial_agent_log]
    B -->|Not Answered| D[Status updated in vicidial_list]
    C -->|Call Completed| E[Status updated in vicidial_list]
    C -->|Call Recorded| F[Recording in recording_log]
    C -->|Notes Added| G[Notes in vicidial_call_notes]
    C -->|Call Transferred| H[Transfer recorded in vicidial_xfer_log]
    H -->|Transferred Call| I[Closer activity in vicidial_closer_log]
    I -->|Call Completed| J[Final status updated in vicidial_list]
```

### Inbound Call Flow

```mermaid
flowchart TD
    A[Inbound Call] -->|DID Routing| B[DID recorded in vicidial_did_log]
    B -->|Routed to Group| C[Call in vicidial_closer_log]
    C -->|Assigned to Agent| D[Agent activity in vicidial_agent_log]
    D -->|Call Completed| E[Status updated in vicidial_list]
    D -->|Call Recorded| F[Recording in recording_log]
    D -->|Notes Added| G[Notes in vicidial_call_notes]
    D -->|New Lead| H[New record in vicidial_list]
```

## Call Recording and Documentation

```mermaid
erDiagram
    vicidial_log ||--o{ recording_log : "vicidial_id"
    vicidial_closer_log ||--o{ recording_log : "vicidial_id"
    vicidial_log ||--o{ vicidial_call_notes : "vicidial_id"
    vicidial_closer_log ||--o{ vicidial_call_notes : "vicidial_id"
    vicidial_list ||--o{ recording_log : "lead_id"
    vicidial_list ||--o{ vicidial_call_notes : "lead_id"
    vicidial_users ||--o{ recording_log : "user"
    
    vicidial_log {
        varchar uniqueid PK
        int lead_id FK
        datetime call_date
        varchar user FK
    }
    
    vicidial_closer_log {
        int closecallid PK
        varchar uniqueid
        int lead_id FK
        datetime call_date
        varchar user FK
    }
    
    recording_log {
        int recording_id PK
        datetime start_time
        datetime end_time
        mediumint length_in_sec
        varchar filename
        varchar location
        int lead_id FK
        varchar user FK
        varchar vicidial_id FK
    }
    
    vicidial_call_notes {
        int notesid PK
        int lead_id FK
        varchar vicidial_id FK
        datetime call_date
        date appointment_date
        time appointment_time
        text call_notes
    }
    
    vicidial_list {
        int lead_id PK
        varchar phone_number
        varchar first_name
        varchar last_name
    }
    
    vicidial_users {
        varchar user PK
        varchar full_name
    }
```

## Campaign Configuration Relationships

```mermaid
erDiagram
    vicidial_campaigns ||--o{ vicidial_lists : "campaign_id"
    vicidial_campaigns ||--o{ vicidial_campaign_statuses : "campaign_id"
    vicidial_campaigns ||--o{ vicidial_hopper : "campaign_id"
    vicidial_campaigns ||--o{ vicidial_auto_calls : "campaign_id"
    vicidial_campaigns ||--o{ vicidial_log : "campaign_id"
    vicidial_lists ||--o{ vicidial_list : "list_id"
    
    vicidial_campaigns {
        varchar campaign_id PK
        varchar campaign_name
        enum active
        varchar dial_method
    }
    
    vicidial_lists {
        bigint list_id PK
        varchar list_name
        varchar campaign_id FK
    }
    
    vicidial_campaign_statuses {
        varchar campaign_id FK
        varchar status
        varchar status_name
    }
    
    vicidial_hopper {
        int hopper_id PK
        int lead_id FK
        varchar campaign_id FK
        varchar status
    }
    
    vicidial_auto_calls {
        varchar auto_call_id PK
        varchar campaign_id FK
        int lead_id FK
        varchar status
    }
    
    vicidial_list {
        int lead_id PK
        bigint list_id FK
        varchar status
    }
    
    vicidial_log {
        varchar uniqueid PK
        varchar campaign_id FK
        int lead_id FK
    }
```

## Agent Activity Tracking

```mermaid
erDiagram
    vicidial_users ||--o{ vicidial_agent_log : "user"
    vicidial_users ||--o{ vicidial_log : "user"
    vicidial_users ||--o{ vicidial_closer_log : "user"
    vicidial_users ||--o{ vicidial_user_log : "user"
    vicidial_users ||--o{ vicidial_timeclock_log : "user"
    vicidial_users ||--o{ vicidial_live_agents : "user"
    
    vicidial_users {
        varchar user PK
        varchar full_name
        varchar user_group
    }
    
    vicidial_agent_log {
        int agent_log_id PK
        varchar user FK
        datetime event_time
        int lead_id FK
        varchar campaign_id FK
        smallint talk_sec
        smallint wait_sec
        smallint dispo_sec
    }
    
    vicidial_log {
        varchar uniqueid PK
        varchar user FK
        datetime call_date
    }
    
    vicidial_closer_log {
        int closecallid PK
        varchar user FK
        datetime call_date
    }
    
    vicidial_user_log {
        int user_log_id PK
        varchar user FK
        datetime event_date
        varchar event
    }
    
    vicidial_timeclock_log {
        int timeclock_id PK
        varchar user FK
        datetime event_date
        varchar event_type
    }
    
    vicidial_live_agents {
        varchar user FK
        varchar campaign_id FK
        varchar status
        int lead_id FK
    }
```

## Inbound and Outbound Call Relationship

```mermaid
erDiagram
    vicidial_list ||--o{ vicidial_log : "lead_id (Outbound)"
    vicidial_list ||--o{ vicidial_closer_log : "lead_id (Inbound)"
    vicidial_log ||--o{ vicidial_xfer_log : "uniqueid"
    vicidial_xfer_log ||--o{ vicidial_closer_log : "closecallid"
    
    vicidial_list {
        int lead_id PK
        varchar phone_number
        varchar status
    }
    
    vicidial_log {
        varchar uniqueid PK
        int lead_id FK
        datetime call_date
        varchar status
        varchar term_reason
    }
    
    vicidial_closer_log {
        int closecallid PK
        varchar uniqueid
        int lead_id FK
        datetime call_date
        varchar status
        int xfercallid FK
    }
    
    vicidial_xfer_log {
        int xfercallid PK
        varchar uniqueid FK
        varchar campaign_id
        varchar closecallid FK
    }
```

## Notes on the Diagrams

1. These diagrams focus on the core tables and their primary relationships. The actual database contains many more tables and relationships.

2. The entity-relationship diagrams (ERD) show the key fields and relationships between tables, while the flowcharts illustrate the logical flow of data through the system.

3. Cardinality notation:
   - `||--o{` indicates a one-to-many relationship
   - `}|--||` indicates a many-to-one relationship
   - `||--||` would indicate a one-to-one relationship (not shown in these diagrams)
   - `}|--|{` would indicate a many-to-many relationship (not shown in these diagrams)

4. Primary keys are marked with PK, and foreign keys with FK.

5. These diagrams can be rendered using any tool that supports Mermaid markdown syntax.

6. The diagrams highlight the distinction between outbound calls (`vicidial_log`) and inbound calls (`vicidial_closer_log`), and how they relate to recordings, notes, and agent activity.
