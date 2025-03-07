# Sample Queries

This document provides practical SQL query examples for common tasks using the ViciDial database. These queries demonstrate how to join tables and extract useful information.

## Call Reporting Queries

### 1. Basic Outbound Call Statistics by Campaign

```sql
SELECT 
    vc.campaign_id,
    vc.campaign_name,
    COUNT(vlog.uniqueid) AS total_calls,
    SUM(CASE WHEN vlog.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) AS successful_calls,
    SUM(vlog.length_in_sec) AS total_talk_time,
    AVG(vlog.length_in_sec) AS avg_talk_time,
    COUNT(DISTINCT vlog.user) AS total_agents
FROM 
    vicidial_campaigns vc
LEFT JOIN 
    vicidial_log vlog ON vc.campaign_id = vlog.campaign_id
WHERE 
    vlog.call_date BETWEEN '2025-01-01' AND '2025-01-31'
GROUP BY 
    vc.campaign_id, vc.campaign_name
ORDER BY 
    total_calls DESC;
```

### 2. Basic Inbound Call Statistics by Group

```sql
SELECT 
    vcl.campaign_id AS inbound_group,
    COUNT(vcl.closecallid) AS total_calls,
    SUM(CASE WHEN vcl.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) AS successful_calls,
    SUM(vcl.length_in_sec) AS total_talk_time,
    AVG(vcl.length_in_sec) AS avg_talk_time,
    AVG(vcl.queue_seconds) AS avg_queue_time,
    COUNT(DISTINCT vcl.user) AS total_agents
FROM 
    vicidial_closer_log vcl
WHERE 
    vcl.call_date BETWEEN '2025-01-01' AND '2025-01-31'
GROUP BY 
    vcl.campaign_id
ORDER BY 
    total_calls DESC;
```

### 3. Hourly Call Distribution (Combined Inbound and Outbound)

```sql
-- Outbound calls by hour
SELECT 
    'Outbound' AS call_type,
    HOUR(vlog.call_date) AS hour_of_day,
    COUNT(vlog.uniqueid) AS total_calls,
    SUM(CASE WHEN vlog.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) AS successful_calls,
    (SUM(CASE WHEN vlog.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) / COUNT(vlog.uniqueid)) * 100 AS success_rate
FROM 
    vicidial_log vlog
WHERE 
    vlog.call_date BETWEEN '2025-01-01' AND '2025-01-31'
GROUP BY 
    HOUR(vlog.call_date)

UNION ALL

-- Inbound calls by hour
SELECT 
    'Inbound' AS call_type,
    HOUR(vcl.call_date) AS hour_of_day,
    COUNT(vcl.closecallid) AS total_calls,
    SUM(CASE WHEN vcl.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) AS successful_calls,
    (SUM(CASE WHEN vcl.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) / COUNT(vcl.closecallid)) * 100 AS success_rate
FROM 
    vicidial_closer_log vcl
WHERE 
    vcl.call_date BETWEEN '2025-01-01' AND '2025-01-31'
GROUP BY 
    HOUR(vcl.call_date)

ORDER BY 
    call_type, hour_of_day;
```

### 4. Call Outcomes by Disposition (Outbound vs Inbound)

```sql
-- Outbound call dispositions
SELECT 
    'Outbound' AS call_type,
    vlog.status,
    COUNT(vlog.uniqueid) AS total_calls,
    (COUNT(vlog.uniqueid) / (SELECT COUNT(*) FROM vicidial_log WHERE call_date BETWEEN '2025-01-01' AND '2025-01-31')) * 100 AS percentage
FROM 
    vicidial_log vlog
WHERE 
    vlog.call_date BETWEEN '2025-01-01' AND '2025-01-31'
GROUP BY 
    vlog.status

UNION ALL

-- Inbound call dispositions
SELECT 
    'Inbound' AS call_type,
    vcl.status,
    COUNT(vcl.closecallid) AS total_calls,
    (COUNT(vcl.closecallid) / (SELECT COUNT(*) FROM vicidial_closer_log WHERE call_date BETWEEN '2025-01-01' AND '2025-01-31')) * 100 AS percentage
FROM 
    vicidial_closer_log vcl
WHERE 
    vcl.call_date BETWEEN '2025-01-01' AND '2025-01-31'
GROUP BY 
    vcl.status

ORDER BY 
    call_type, total_calls DESC;
```

## Call Recording and Notes Queries

### 1. Call Recording Analysis

```sql
-- Outbound call recordings
SELECT 
    'Outbound' AS call_type,
    vl.lead_id,
    vl.first_name,
    vl.last_name,
    vl.phone_number,
    vlog.call_date,
    vlog.length_in_sec AS call_length,
    vlog.status,
    vlog.user AS agent,
    rec.recording_id,
    rec.length_in_sec AS recording_length,
    rec.location AS recording_path,
    rec.filename
FROM 
    vicidial_list vl
JOIN 
    vicidial_log vlog ON vl.lead_id = vlog.lead_id
JOIN 
    recording_log rec ON vlog.uniqueid = rec.vicidial_id
WHERE 
    vlog.call_date BETWEEN '2025-01-01' AND '2025-01-31'
ORDER BY 
    vlog.call_date DESC
LIMIT 100;

-- Inbound call recordings
SELECT 
    'Inbound' AS call_type,
    vl.lead_id,
    vl.first_name,
    vl.last_name,
    vl.phone_number,
    vcl.call_date,
    vcl.length_in_sec AS call_length,
    vcl.status,
    vcl.user AS agent,
    rec.recording_id,
    rec.length_in_sec AS recording_length,
    rec.location AS recording_path,
    rec.filename
FROM 
    vicidial_list vl
JOIN 
    vicidial_closer_log vcl ON vl.lead_id = vcl.lead_id
JOIN 
    recording_log rec ON vcl.uniqueid = rec.vicidial_id
WHERE 
    vcl.call_date BETWEEN '2025-01-01' AND '2025-01-31'
ORDER BY 
    vcl.call_date DESC
LIMIT 100;
```

### 2. Call Notes Analysis

```sql
-- Outbound call notes
SELECT 
    'Outbound' AS call_type,
    vl.lead_id,
    vl.first_name,
    vl.last_name,
    vl.phone_number,
    vlog.call_date,
    vlog.status,
    vlog.user AS agent,
    vcn.notesid,
    vcn.call_notes,
    vcn.appointment_date,
    vcn.appointment_time
FROM 
    vicidial_list vl
JOIN 
    vicidial_log vlog ON vl.lead_id = vlog.lead_id
JOIN 
    vicidial_call_notes vcn ON vlog.uniqueid = vcn.vicidial_id
WHERE 
    vlog.call_date BETWEEN '2025-01-01' AND '2025-01-31'
ORDER BY 
    vlog.call_date DESC
LIMIT 100;

-- Inbound call notes
SELECT 
    'Inbound' AS call_type,
    vl.lead_id,
    vl.first_name,
    vl.last_name,
    vl.phone_number,
    vcl.call_date,
    vcl.status,
    vcl.user AS agent,
    vcn.notesid,
    vcn.call_notes,
    vcn.appointment_date,
    vcn.appointment_time
FROM 
    vicidial_list vl
JOIN 
    vicidial_closer_log vcl ON vl.lead_id = vcl.lead_id
JOIN 
    vicidial_call_notes vcn ON vcl.uniqueid = vcn.vicidial_id
WHERE 
    vcl.call_date BETWEEN '2025-01-01' AND '2025-01-31'
ORDER BY 
    vcl.call_date DESC
LIMIT 100;
```

### 3. Calls with Missing Recordings or Notes

```sql
-- Outbound calls missing recordings
SELECT 
    vl.lead_id,
    vl.first_name,
    vl.last_name,
    vlog.call_date,
    vlog.length_in_sec,
    vlog.status,
    vlog.user AS agent
FROM 
    vicidial_list vl
JOIN 
    vicidial_log vlog ON vl.lead_id = vlog.lead_id
LEFT JOIN 
    recording_log rec ON vlog.uniqueid = rec.vicidial_id
WHERE 
    vlog.call_date BETWEEN '2025-01-01' AND '2025-01-31'
    AND vlog.length_in_sec > 30  -- Only consider calls longer than 30 seconds
    AND rec.recording_id IS NULL
ORDER BY 
    vlog.call_date DESC;

-- Calls missing notes
SELECT 
    vl.lead_id,
    vl.first_name,
    vl.last_name,
    vlog.call_date,
    vlog.length_in_sec,
    vlog.status,
    vlog.user AS agent
FROM 
    vicidial_list vl
JOIN 
    vicidial_log vlog ON vl.lead_id = vlog.lead_id
LEFT JOIN 
    vicidial_call_notes vcn ON vlog.uniqueid = vcn.vicidial_id
WHERE 
    vlog.call_date BETWEEN '2025-01-01' AND '2025-01-31'
    AND vlog.status IN ('SALE', 'XFER', 'CALLBK')  -- Important statuses that should have notes
    AND vcn.notesid IS NULL
ORDER BY 
    vlog.call_date DESC;
```

## Agent Performance Queries

### 1. Agent Call Handling Metrics (Combined Inbound and Outbound)

```sql
-- Outbound call metrics by agent
SELECT 
    'Outbound' AS call_type,
    val.user,
    vu.full_name,
    COUNT(val.lead_id) AS total_calls,
    SUM(val.talk_sec) AS total_talk_time,
    AVG(val.talk_sec) AS avg_talk_time,
    SUM(val.wait_sec) AS total_wait_time,
    AVG(val.wait_sec) AS avg_wait_time,
    SUM(val.dispo_sec) AS total_dispo_time,
    AVG(val.dispo_sec) AS avg_dispo_time,
    SUM(CASE WHEN vlog.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) AS successful_calls,
    (SUM(CASE WHEN vlog.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) / COUNT(val.lead_id)) * 100 AS success_rate
FROM 
    vicidial_agent_log val
JOIN 
    vicidial_users vu ON val.user = vu.user
JOIN 
    vicidial_log vlog ON val.uniqueid = vlog.uniqueid
WHERE 
    val.event_time BETWEEN '2025-01-01' AND '2025-01-31'
GROUP BY 
    val.user, vu.full_name

UNION ALL

-- Inbound call metrics by agent
SELECT 
    'Inbound' AS call_type,
    val.user,
    vu.full_name,
    COUNT(val.lead_id) AS total_calls,
    SUM(val.talk_sec) AS total_talk_time,
    AVG(val.talk_sec) AS avg_talk_time,
    SUM(val.wait_sec) AS total_wait_time,
    AVG(val.wait_sec) AS avg_wait_time,
    SUM(val.dispo_sec) AS total_dispo_time,
    AVG(val.dispo_sec) AS avg_dispo_time,
    SUM(CASE WHEN vcl.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) AS successful_calls,
    (SUM(CASE WHEN vcl.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) / COUNT(val.lead_id)) * 100 AS success_rate
FROM 
    vicidial_agent_log val
JOIN 
    vicidial_users vu ON val.user = vu.user
JOIN 
    vicidial_closer_log vcl ON val.uniqueid = vcl.uniqueid
WHERE 
    val.event_time BETWEEN '2025-01-01' AND '2025-01-31'
GROUP BY 
    val.user, vu.full_name

ORDER BY 
    call_type, success_rate DESC;
```

### 2. Agent Time Utilization

```sql
SELECT 
    val.user,
    vu.full_name,
    SUM(val.pause_sec) AS total_pause_time,
    SUM(val.wait_sec) AS total_wait_time,
    SUM(val.talk_sec) AS total_talk_time,
    SUM(val.dispo_sec) AS total_dispo_time,
    SUM(val.dead_sec) AS total_dead_time,
    SUM(val.pause_sec + val.wait_sec + val.talk_sec + val.dispo_sec + val.dead_sec) AS total_logged_time,
    (SUM(val.talk_sec) / SUM(val.pause_sec + val.wait_sec + val.talk_sec + val.dispo_sec + val.dead_sec)) * 100 AS talk_time_percentage
FROM 
    vicidial_agent_log val
JOIN 
    vicidial_users vu ON val.user = vu.user
WHERE 
    val.event_time BETWEEN '2025-01-01' AND '2025-01-31'
GROUP BY 
    val.user, vu.full_name
ORDER BY 
    talk_time_percentage DESC;
```

### 3. Agent Pause Reasons

```sql
SELECT 
    val.user,
    vu.full_name,
    val.sub_status,
    COUNT(*) AS pause_count,
    SUM(val.pause_sec) AS total_pause_time,
    AVG(val.pause_sec) AS avg_pause_time
FROM 
    vicidial_agent_log val
JOIN 
    vicidial_users vu ON val.user = vu.user
WHERE 
    val.event_time BETWEEN '2025-01-01' AND '2025-01-31'
    AND val.pause_sec > 0
    AND val.sub_status IS NOT NULL
GROUP BY 
    val.user, vu.full_name, val.sub_status
ORDER BY 
    val.user, total_pause_time DESC;
```

## Lead Management Queries

### 1. Lead Status Distribution

```sql
SELECT 
    vl.status,
    COUNT(vl.lead_id) AS lead_count,
    (COUNT(vl.lead_id) / (SELECT COUNT(*) FROM vicidial_list)) * 100 AS percentage
FROM 
    vicidial_list vl
GROUP BY 
    vl.status
ORDER BY 
    lead_count DESC;
```

### 2. Leads by List and Status

```sql
SELECT 
    vli.list_id,
    vli.list_name,
    vl.status,
    COUNT(vl.lead_id) AS lead_count
FROM 
    vicidial_list vl
JOIN 
    vicidial_lists vli ON vl.list_id = vli.list_id
GROUP BY 
    vli.list_id, vli.list_name, vl.status
ORDER BY 
    vli.list_id, lead_count DESC;
```

### 3. Lead Call History (Combined Inbound and Outbound)

```sql
-- Outbound calls
SELECT 
    'Outbound' AS call_type,
    vl.lead_id,
    vl.first_name,
    vl.last_name,
    vl.phone_number,
    vlog.call_date,
    vlog.status,
    vlog.length_in_sec,
    vlog.user,
    vu.full_name AS agent_name
FROM 
    vicidial_list vl
JOIN 
    vicidial_log vlog ON vl.lead_id = vlog.lead_id
LEFT JOIN 
    vicidial_users vu ON vlog.user = vu.user
WHERE 
    vl.lead_id = 12345  -- Replace with actual lead_id

UNION ALL

-- Inbound calls
SELECT 
    'Inbound' AS call_type,
    vl.lead_id,
    vl.first_name,
    vl.last_name,
    vl.phone_number,
    vcl.call_date,
    vcl.status,
    vcl.length_in_sec,
    vcl.user,
    vu.full_name AS agent_name
FROM 
    vicidial_list vl
JOIN 
    vicidial_closer_log vcl ON vl.lead_id = vcl.lead_id
LEFT JOIN 
    vicidial_users vu ON vcl.user = vu.user
WHERE 
    vl.lead_id = 12345  -- Replace with actual lead_id

ORDER BY 
    call_date DESC;
```

## Campaign Management Queries

### 1. Campaign Hopper Status

```sql
SELECT 
    vh.campaign_id,
    vc.campaign_name,
    COUNT(vh.lead_id) AS leads_in_hopper,
    vc.hopper_level AS target_hopper_level,
    (COUNT(vh.lead_id) / vc.hopper_level) * 100 AS hopper_fill_percentage
FROM 
    vicidial_hopper vh
JOIN 
    vicidial_campaigns vc ON vh.campaign_id = vc.campaign_id
GROUP BY 
    vh.campaign_id, vc.campaign_name, vc.hopper_level
ORDER BY 
    hopper_fill_percentage;
```

### 2. Campaign List Distribution

```sql
SELECT 
    vc.campaign_id,
    vc.campaign_name,
    vli.list_id,
    vli.list_name,
    COUNT(vl.lead_id) AS total_leads,
    SUM(CASE WHEN vl.called_since_last_reset = 'Y' THEN 1 ELSE 0 END) AS called_leads,
    (SUM(CASE WHEN vl.called_since_last_reset = 'Y' THEN 1 ELSE 0 END) / COUNT(vl.lead_id)) * 100 AS called_percentage
FROM 
    vicidial_campaigns vc
JOIN 
    vicidial_lists vli ON vc.campaign_id = vli.campaign_id
JOIN 
    vicidial_list vl ON vli.list_id = vl.list_id
GROUP BY 
    vc.campaign_id, vc.campaign_name, vli.list_id, vli.list_name
ORDER BY 
    vc.campaign_id, total_leads DESC;
```

### 3. Campaign Performance Comparison

```sql
SELECT 
    vc.campaign_id,
    vc.campaign_name,
    COUNT(vlog.uniqueid) AS total_calls,
    SUM(CASE WHEN vlog.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) AS successful_calls,
    (SUM(CASE WHEN vlog.status IN ('SALE', 'XFER') THEN 1 ELSE 0 END) / COUNT(vlog.uniqueid)) * 100 AS success_rate,
    AVG(vlog.length_in_sec) AS avg_call_length,
    COUNT(DISTINCT vlog.user) AS total_agents,
    COUNT(vlog.uniqueid) / COUNT(DISTINCT vlog.user) AS calls_per_agent
FROM 
    vicidial_campaigns vc
LEFT JOIN 
    vicidial_log vlog ON vc.campaign_id = vlog.campaign_id
WHERE 
    vlog.call_date BETWEEN '2025-01-01' AND '2025-01-31'
    AND vc.active = 'Y'
GROUP BY 
    vc.campaign_id, vc.campaign_name
ORDER BY 
    success_rate DESC;
```

## Advanced Multi-Table Queries

### 1. Comprehensive Call Analysis

```sql
-- Outbound call analysis
SELECT 
    'Outbound' AS call_type,
    vlog.call_date,
    vl.lead_id,
    vl.first_name,
    vl.last_name,
    vl.phone_number,
    vli.list_name,
    vc.campaign_name,
    vlog.status AS call_status,
    vlog.length_in_sec AS call_duration,
    vu.full_name AS agent_name,
    val.talk_sec,
    val.wait_sec,
    val.dispo_sec,
    rec.location AS recording_location,
    vcn.call_notes
FROM 
    vicidial_log vlog
JOIN 
    vicidial_list vl ON vlog.lead_id = vl.lead_id
JOIN 
    vicidial_lists vli ON vl.list_id = vli.list_id
JOIN 
    vicidial_campaigns vc ON vlog.campaign_id = vc.campaign_id
LEFT JOIN 
    vicidial_users vu ON vlog.user = vu.user
LEFT JOIN 
    vicidial_agent_log val ON vlog.uniqueid = val.uniqueid
LEFT JOIN 
    recording_log rec ON vlog.uniqueid = rec.vicidial_id
LEFT JOIN
    vicidial_call_notes vcn ON vlog.uniqueid = vcn.vicidial_id
WHERE 
    vlog.call_date BETWEEN '2025-01-01' AND '2025-01-31'
    AND vlog.length_in_sec > 0
ORDER BY 
    vlog.call_date DESC
LIMIT 100;

-- Inbound call analysis
SELECT 
    'Inbound' AS call_type,
    vcl.call_date,
    vl.lead_id,
    vl.first_name,
    vl.last_name,
    vl.phone_number,
    vli.list_name,
    vcl.campaign_id AS inbound_group,
    vcl.status AS call_status,
    vcl.length_in_sec AS call_duration,
    vcl.queue_seconds,
    vu.full_name AS agent_name,
    val.talk_sec,
    val.wait_sec,
    val.dispo_sec,
    rec.location AS recording_location,
    vcn.call_notes
FROM 
    vicidial_closer_log vcl
JOIN 
    vicidial_list vl ON vcl.lead_id = vl.lead_id
JOIN 
    vicidial_lists vli ON vl.list_id = vli.list_id
LEFT JOIN 
    vicidial_users vu ON vcl.user = vu.user
LEFT JOIN 
    vicidial_agent_log val ON vcl.uniqueid = val.uniqueid
LEFT JOIN 
    recording_log rec ON vcl.uniqueid = rec.vicidial_id
LEFT JOIN
    vicidial_call_notes vcn ON vcl.uniqueid = vcn.vicidial_id
WHERE 
    vcl.call_date BETWEEN '2025-01-01' AND '2025-01-31'
    AND vcl.length_in_sec > 0
ORDER BY 
    vcl.call_date DESC
LIMIT 100;
```

### 2. Combined Call History for a Lead

```sql
SELECT 
    CASE 
        WHEN vlog.uniqueid IS NOT NULL THEN 'Outbound' 
        ELSE 'Inbound' 
    END AS call_type,
    COALESCE(vlog.call_date, vcl.call_date) AS call_date,
    vl.lead_id,
    vl.first_name,
    vl.last_name,
    vl.phone_number,
    COALESCE(vlog.status, vcl.status) AS status,
    COALESCE(vlog.length_in_sec, vcl.length_in_sec) AS length_in_sec,
    COALESCE(vlog.user, vcl.user) AS agent,
    CASE 
        WHEN rec.recording_id IS NOT NULL THEN 'Yes' 
        ELSE 'No' 
    END AS has_recording,
    CASE 
        WHEN vcn.notesid IS NOT NULL THEN 'Yes' 
        ELSE 'No' 
    END AS has_notes
FROM 
    vicidial_list vl
LEFT JOIN 
    vicidial_log vlog ON vl.lead_id = vlog.lead_id
LEFT JOIN 
    vicidial_closer_log vcl ON vl.lead_id = vcl.lead_id AND vlog.uniqueid IS NULL
LEFT JOIN 
    recording_log rec ON COALESCE(vlog.uniqueid, vcl.uniqueid) = rec.vicidial_id
LEFT JOIN 
    vicidial_call_notes vcn ON COALESCE(vlog.uniqueid, vcl.uniqueid) = vcn.vicidial_id
WHERE 
    vl.lead_id = 12345  -- Replace with actual lead_id
    AND (vlog.uniqueid IS NOT NULL OR vcl.uniqueid IS NOT NULL)
ORDER BY 
    call_date DESC;
```

## Notes on Query Usage

1. **Date Ranges**: Replace the date ranges (`'2025-01-01'` and `'2025-01-31'`) with the actual date ranges you want to analyze.

2. **Status Codes**: The status codes used in these queries (e.g., `'SALE'`, `'XFER'`, `'ANSWER'`) should be replaced with the actual status codes used in your system.

3. **Performance Considerations**: Some of these queries involve multiple joins and aggregations, which can be resource-intensive on large databases. Consider adding appropriate indexes and testing on a non-production environment first.

4. **Customization**: These queries can be customized to fit specific reporting needs by modifying the columns, joins, filters, and grouping.

5. **Pagination**: For queries that might return a large number of rows, consider adding `LIMIT` and `OFFSET` clauses for pagination.
