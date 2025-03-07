# Table Schemas

This document contains detailed information about the structure of each table in the database.

## Call Management Tables

### vicidial_log

The `vicidial_log` table stores information about outbound calls made through the system.

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| uniqueid | varchar | PRI | NO | Unique identifier for the call |
| lead_id | int | MUL | NO | Reference to the lead/contact |
| list_id | bigint | | YES | Reference to the list the lead belongs to |
| campaign_id | varchar | | YES | Campaign identifier |
| call_date | datetime | MUL | YES | Date and time of the call |
| start_epoch | int | | YES | Call start time in Unix epoch format |
| end_epoch | int | | YES | Call end time in Unix epoch format |
| length_in_sec | int | | YES | Duration of the call in seconds |
| status | varchar | | YES | Call status/disposition |
| phone_code | varchar | | YES | Country or area code |
| phone_number | varchar | | YES | Phone number called |
| user | varchar | | YES | Agent who handled the call |
| comments | varchar | | YES | Call comments |
| processed | enum | | YES | Processing status |
| user_group | varchar | | YES | User/agent group |
| term_reason | enum | | YES | Termination reason |
| alt_dial | varchar | | YES | Alternative dial information |
| called_count | smallint | | YES | Number of times the lead has been called |

#### Key Observations:
- Primary key is `uniqueid`, which uniquely identifies each call
- `lead_id` is indexed, suggesting frequent joins with a leads table
- `call_date` is indexed, likely for time-based queries
- Contains both call metadata (times, duration) and outcome information (status)
- Links to campaigns, lists, and users, suggesting relationships with other tables

#### Potential Relationships:
- `lead_id` relates to `vicidial_list.lead_id`
- `list_id` relates to `vicidial_lists.list_id`
- `campaign_id` relates to `vicidial_campaigns.campaign_id`
- `user` relates to `vicidial_users.user`
- `uniqueid` relates to `recording_log.vicidial_id`

### vicidial_closer_log

The `vicidial_closer_log` table stores information about inbound calls handled by the system.

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| closecallid | int | PRI | NO | Unique identifier for the inbound call |
| lead_id | int | MUL | NO | Reference to the lead/contact |
| list_id | bigint | | YES | Reference to the list the lead belongs to |
| campaign_id | varchar | MUL | YES | Campaign/inbound group identifier |
| call_date | datetime | MUL | YES | Date and time of the call |
| start_epoch | int | | YES | Call start time in Unix epoch format |
| end_epoch | int | | YES | Call end time in Unix epoch format |
| length_in_sec | int | | YES | Duration of the call in seconds |
| status | varchar | | YES | Call status/disposition |
| phone_code | varchar | | YES | Country or area code |
| phone_number | varchar | MUL | YES | Phone number of the caller |
| user | varchar | | YES | Agent who handled the call |
| comments | varchar | | YES | Call comments |
| processed | enum | | YES | Processing status |
| queue_seconds | decimal | | YES | Time spent in queue before being answered |
| user_group | varchar | | YES | Agent's user group |
| xfercallid | int | | YES | ID of the call if transferred |
| term_reason | enum | | YES | Termination reason |
| uniqueid | varchar | MUL | NO | Unique call identifier |
| agent_only | varchar | | YES | Agent-only flag |
| queue_position | smallint | | YES | Position in queue |
| called_count | smallint | | YES | Number of times the lead has been called |

#### Key Observations:
- Primary key is `closecallid`, which uniquely identifies each inbound call
- Structure is similar to `vicidial_log` but with additional fields for queue handling
- Contains both call metadata and outcome information
- Includes queue-specific fields like `queue_seconds` and `queue_position`
- Has a field for tracking transfers (`xfercallid`)

#### Potential Relationships:
- `lead_id` relates to `vicidial_list.lead_id`
- `campaign_id` relates to `vicidial_inbound_groups.group_id`
- `user` relates to `vicidial_users.user`
- `uniqueid` relates to `recording_log.vicidial_id`
- `xfercallid` relates to other call logs for transferred calls

### recording_log

The `recording_log` table stores information about call recordings.

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| recording_id | int | PRI | NO | Unique identifier for the recording |
| channel | varchar | | YES | Channel on which the recording was made |
| server_ip | varchar | | YES | IP of the server that made the recording |
| extension | varchar | | YES | Extension that was recorded |
| start_time | datetime | | YES | Start time of the recording |
| start_epoch | int | | YES | Start time in Unix epoch format |
| end_time | datetime | | YES | End time of the recording |
| end_epoch | int | | YES | End time in Unix epoch format |
| length_in_sec | mediumint | | YES | Duration of the recording in seconds |
| length_in_min | double | | YES | Duration of the recording in minutes |
| filename | varchar | MUL | YES | Name of the recording file |
| location | varchar | | YES | Path to the recording file |
| lead_id | int | MUL | YES | Reference to the lead/contact |
| user | varchar | MUL | YES | Agent who was recorded |
| vicidial_id | varchar | MUL | YES | Reference to the call ID |

#### Key Observations:
- Primary key is `recording_id`, which uniquely identifies each recording
- Contains metadata about the recording (time, duration, location)
- Links to calls and agents
- `vicidial_id` field links to either `vicidial_log.uniqueid` or `vicidial_closer_log.uniqueid`

#### Potential Relationships:
- `lead_id` relates to `vicidial_list.lead_id`
- `user` relates to `vicidial_users.user`
- `vicidial_id` relates to `vicidial_log.uniqueid` or `vicidial_closer_log.uniqueid`

## Lead Management Tables

### vicidial_list

The `vicidial_list` table stores lead/contact information for outbound and inbound calling.

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| lead_id | int | PRI | NO | Unique identifier for the lead |
| entry_date | datetime | | YES | Date the lead was entered into the system |
| modify_date | timestamp | | NO | Date the lead was last modified |
| status | varchar | MUL | YES | Current status of the lead |
| user | varchar | | YES | User who last modified the lead |
| vendor_lead_code | varchar | | YES | External vendor code for the lead |
| source_id | varchar | | YES | Source of the lead |
| list_id | bigint | MUL | NO | List the lead belongs to |
| gmt_offset_now | decimal | MUL | YES | GMT offset for the lead's timezone |
| called_since_last_reset | enum | MUL | YES | Whether the lead has been called since last reset |
| phone_code | varchar | | YES | Country or area code |
| phone_number | varchar | MUL | NO | Phone number of the lead |
| title | varchar | | YES | Title (Mr, Mrs, etc.) |
| first_name | varchar | | YES | First name |
| middle_initial | varchar | | YES | Middle initial |
| last_name | varchar | | YES | Last name |
| address1 | varchar | | YES | Address line 1 |
| address2 | varchar | | YES | Address line 2 |
| address3 | varchar | | YES | Address line 3 |
| city | varchar | | YES | City |
| state | varchar | | YES | State |
| province | varchar | | YES | Province |
| postal_code | varchar | MUL | YES | Postal/ZIP code |
| country_code | varchar | | YES | Country code |
| gender | enum | | YES | Gender |
| date_of_birth | date | | YES | Date of birth |
| alt_phone | varchar | | YES | Alternative phone number |
| email | varchar | | YES | Email address |
| security_phrase | varchar | | YES | Security phrase |
| comments | varchar | | YES | Comments |
| called_count | smallint | | YES | Number of times the lead has been called |
| last_local_call_time | datetime | MUL | YES | Time of the last local call |
| rank | smallint | MUL | NO | Priority rank |
| owner | varchar | MUL | YES | Owner of the lead |
| entry_list_id | bigint | | NO | Original list ID the lead was entered into |

#### Key Observations:
- Primary key is `lead_id`, which uniquely identifies each lead
- Contains comprehensive contact information
- Includes status tracking and call history information
- Has fields for lead source and ownership tracking
- Central table that links to most call-related tables

#### Potential Relationships:
- `lead_id` is referenced by `vicidial_log.lead_id`, `vicidial_closer_log.lead_id`, `vicidial_agent_log.lead_id`, `recording_log.lead_id`, and `vicidial_call_notes.lead_id`
- `list_id` relates to `vicidial_lists.list_id`
- `user` relates to `vicidial_users.user`
- `owner` might relate to `vicidial_users.user` or a different ownership table

## Agent Activity Tables

### vicidial_agent_log

The `vicidial_agent_log` table tracks detailed agent activity and performance metrics.

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| agent_log_id | int | PRI | NO | Unique identifier for the log entry |
| user | varchar | MUL | YES | Agent user ID |
| server_ip | varchar | | NO | Server IP address |
| event_time | datetime | MUL | YES | Time of the event |
| lead_id | int | MUL | YES | Lead being worked on |
| campaign_id | varchar | | YES | Campaign the agent is working on |
| pause_epoch | int | | YES | Time when agent entered pause state |
| pause_sec | smallint | | YES | Duration of pause in seconds |
| wait_epoch | int | | YES | Time when agent entered wait state |
| wait_sec | smallint | | YES | Duration of wait in seconds |
| talk_epoch | int | | YES | Time when agent began talking |
| talk_sec | smallint | | YES | Duration of talk in seconds |
| dispo_epoch | int | | YES | Time when agent began disposition |
| dispo_sec | smallint | | YES | Duration of disposition in seconds |
| status | varchar | | YES | Status/disposition code |
| user_group | varchar | | YES | Agent's user group |
| comments | varchar | | YES | Comments |
| sub_status | varchar | | YES | Sub-status during pause |
| dead_epoch | int | | YES | Time when call went dead |
| dead_sec | smallint | | YES | Duration of dead time in seconds |
| processed | enum | | YES | Processing status |
| uniqueid | varchar | | YES | Unique call identifier |
| pause_type | enum | | YES | Type of pause |

#### Key Observations:
- Primary key is `agent_log_id`, which uniquely identifies each log entry
- Tracks detailed time segments of agent activity (pause, wait, talk, disposition)
- Contains performance metrics that can be used for agent evaluation
- Includes status and sub-status information
- Links to leads, campaigns, and calls

#### Potential Relationships:
- `user` relates to `vicidial_users.user`
- `lead_id` relates to `vicidial_list.lead_id`
- `campaign_id` relates to `vicidial_campaigns.campaign_id`
- `uniqueid` relates to `vicidial_log.uniqueid` or `vicidial_closer_log.uniqueid`

## Campaign Management Tables

### vicidial_campaigns

The `vicidial_campaigns` table stores configuration settings for outbound calling campaigns.

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| campaign_id | varchar | PRI | NO | Unique identifier for the campaign |
| campaign_name | varchar | | YES | Name of the campaign |
| active | enum | | YES | Whether the campaign is active |
| dial_status_a | varchar | | YES | Primary dial status |
| dial_status_b | varchar | | YES | Secondary dial status |
| dial_status_c | varchar | | YES | Tertiary dial status |
| dial_status_d | varchar | | YES | Quaternary dial status |
| dial_status_e | varchar | | YES | Quinary dial status |
| lead_order | varchar | | YES | Order to pull leads |
| park_ext | varchar | | YES | Park extension |
| park_file_name | varchar | | YES | Park file name |
| web_form_address | text | | YES | Web form URL |
| allow_closers | enum | | YES | Whether to allow closers |
| hopper_level | int | | YES | Number of leads to keep in the hopper |
| auto_dial_level | varchar | | YES | Auto-dial level |
| next_agent_call | varchar | | YES | Next agent call strategy |
| local_call_time | varchar | | YES | Local call time restrictions |
| voicemail_ext | varchar | | YES | Voicemail extension |
| dial_timeout | tinyint | | YES | Dial timeout in seconds |
| dial_prefix | varchar | | YES | Dial prefix |
| campaign_cid | varchar | | YES | Campaign caller ID |

#### Key Observations:
- Primary key is `campaign_id`, which uniquely identifies each campaign
- Contains extensive configuration settings for outbound dialing
- Includes settings for call handling, recording, and agent behavior
- Has fields for web forms, scripts, and other campaign resources
- Includes settings for adaptive dialing and call pacing

#### Potential Relationships:
- `campaign_id` is referenced by `vicidial_log.campaign_id` and `vicidial_agent_log.campaign_id`
- References to other tables through various configuration fields

## Call Notes and Documentation

### vicidial_call_notes

The `vicidial_call_notes` table stores detailed notes and documentation for calls.

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| notesid | int | PRI | NO | Unique identifier for the note |
| lead_id | int | MUL | NO | Reference to the lead/contact |
| vicidial_id | varchar | MUL | YES | Reference to the call ID |
| call_date | datetime | | YES | Date and time of the call |
| order_id | varchar | | YES | Order ID if applicable |
| appointment_date | date | | YES | Date of appointment if scheduled |
| appointment_time | time | | YES | Time of appointment if scheduled |
| call_notes | text | | YES | Detailed notes about the call |

#### Key Observations:
- Primary key is `notesid`, which uniquely identifies each note
- Contains detailed text notes about calls
- Includes fields for tracking appointments
- Links to leads and calls
- Provides a way to document call outcomes and follow-up actions

#### Potential Relationships:
- `lead_id` relates to `vicidial_list.lead_id`
- `vicidial_id` relates to `vicidial_log.uniqueid` or `vicidial_closer_log.uniqueid`
