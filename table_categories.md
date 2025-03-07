# Table Categories

This document organizes the tables in the ViciDial database into functional categories to provide a clearer understanding of the database structure.

## Call Management Tables

These tables are related to the core call handling functionality:

| Table Name | Description | Key Fields |
|------------|-------------|------------|
| vicidial_log | Outbound call records | uniqueid, lead_id, call_date, status |
| vicidial_closer_log | Inbound call records | closecallid, lead_id, call_date, status |
| vicidial_xfer_log | Call transfer records | xfercallid, lead_id, closer |
| park_log | Call parking records | parked_time, channel, server_ip |
| recording_log | Call recording records | recording_id, lead_id, filename |
| vicidial_outbound_ivr_log | Outbound IVR interaction records | event_date, lead_id, campaign_id |
| vicidial_drop_log | Dropped call records | drop_date, lead_id, campaign_id |

## Lead Management Tables

These tables store information about leads and contacts:

| Table Name | Description | Key Fields |
|------------|-------------|------------|
| vicidial_list | Main lead/contact information | lead_id, phone_number, status |
| vicidial_list_alt_phones | Alternative phone numbers for leads | alt_phone_id, lead_id, phone_number |
| vicidial_dnc | Do Not Call list | phone_number |
| vicidial_lead_recycle | Lead recycling configuration | recycle_id, campaign_id, status |
| vicidial_lead_filters | Lead filtering rules | lead_filter_id, filter_name |

## Agent Management Tables

These tables track agent activity and performance:

| Table Name | Description | Key Fields |
|------------|-------------|------------|
| vicidial_agent_log | Detailed agent activity records | agent_log_id, user, event_time |
| vicidial_users | Agent/user accounts | user, pass, full_name |
| vicidial_timeclock_log | Agent time clock records | timeclock_id, user, event |
| vicidial_user_log | User activity log | user_log_id, user, event |
| vicidial_live_agents | Currently active agents | user, conf_exten, campaign_id |
| vicidial_pause_codes | Agent pause reason codes | pause_code, pause_code_name |

## Campaign Management Tables

These tables configure and track outbound calling campaigns:

| Table Name | Description | Key Fields |
|------------|-------------|------------|
| vicidial_campaigns | Campaign configuration | campaign_id, campaign_name |
| vicidial_campaign_stats | Campaign statistics | campaign_id, calls_today |
| vicidial_hopper | Lead hopper for auto-dialing | hopper_id, lead_id, campaign_id |
| vicidial_lists | Lead lists | list_id, list_name, campaign_id |
| vicidial_statuses | Call status codes | status, status_name |
| vicidial_campaign_statuses | Campaign-specific status codes | campaign_id, status |
| vicidial_auto_calls | Currently active calls | auto_call_id, lead_id, status |

## Inbound Call Management Tables

These tables configure and track inbound call handling:

| Table Name | Description | Key Fields |
|------------|-------------|------------|
| vicidial_inbound_groups | Inbound call group configuration | group_id, group_name |
| vicidial_inbound_dids | DID (Direct Inward Dial) configuration | did_id, did_pattern |
| vicidial_did_log | DID call log | uniqueid, did_id, call_date |
| vicidial_did_agent_log | DID agent activity | uniqueid, did_id, user |

## System Configuration Tables

These tables store system-wide configuration settings:

| Table Name | Description | Key Fields |
|------------|-------------|------------|
| servers | Server configuration | server_id, server_ip |
| phones | Phone extension configuration | extension, server_ip |
| vicidial_conferences | Conference configuration | conf_exten, server_ip |
| system_settings | Global system settings | setting_id, setting_name, setting_value |
| vicidial_settings_containers | Configuration containers | container_id, container_name |

## Reporting and Analysis Tables

These tables support reporting and analysis functions:

| Table Name | Description | Key Fields |
|------------|-------------|------------|
| vicidial_carrier_log | Carrier performance tracking | uniqueid, call_date, server_ip |
| vicidial_api_log | API usage tracking | api_id, user, function |
| vicidial_report_log | Report execution tracking | report_log_id, user, report_name |
| vicidial_admin_log | Administrative action tracking | admin_log_id, user, event_date |
| vicidial_log_extended | Extended call data | uniqueid, call_date, lead_id |

## Script and Form Tables

These tables store scripts and forms used by agents:

| Table Name | Description | Key Fields |
|------------|-------------|------------|
| vicidial_scripts | Agent scripts | script_id, script_name, script_text |
| vicidial_lists_fields | Custom fields for lead lists | field_id, list_id, field_label |
| vicidial_screen_labels | Custom screen labels | label_id, label_name |

## Quality Control Tables

These tables support quality control and monitoring:

| Table Name | Description | Key Fields |
|------------|-------------|------------|
| vicidial_monitor_log | Call monitoring records | monitor_id, user, monitor_time |
| quality_control_checkpoints | QC checkpoint definitions | checkpoint_id, checkpoint_text |
| quality_control_queue | Calls queued for QC | qc_id, lead_id, status |
| vicidial_qc_codes | Quality control result codes | qc_code_id, qc_code, qc_code_name |

## Functional Relationships Between Categories

The database categories are interconnected in the following ways:

1. **Lead Management → Campaign Management**: Leads are organized into lists, which are assigned to campaigns.

2. **Campaign Management → Call Management**: Campaigns generate outbound calls, which are tracked in call logs.

3. **Call Management → Agent Management**: Calls are handled by agents, whose activities are tracked in agent logs.

4. **Inbound Call Management → Agent Management**: Inbound calls are routed to agents based on skills and availability.

5. **System Configuration → All Categories**: System settings affect the behavior of all other components.

6. **Script and Form Tables → Agent Management**: Agents use scripts and forms when interacting with leads.

7. **Quality Control → Call Management**: Calls are monitored and evaluated for quality control purposes.

8. **All Categories → Reporting and Analysis**: Data from all categories feeds into reporting and analysis.

This categorization helps understand the functional organization of the database and how different components interact with each other.
