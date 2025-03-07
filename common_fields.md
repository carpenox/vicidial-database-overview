# Common Fields Across Tables

This document identifies common fields that appear across multiple tables in the ViciDial database. These fields can be used for joining tables, even when not explicitly defined as foreign keys.

## Identifier Fields

| Field Name | Tables | Description | Potential Use |
|------------|--------|-------------|--------------|
| `lead_id` | vicidial_list, vicidial_log, vicidial_agent_log | Unique identifier for leads/contacts | Join lead information with call logs and agent activity |
| `uniqueid` | vicidial_log, vicidial_agent_log | Unique identifier for calls | Link call logs with agent activity for the same call |
| `campaign_id` | vicidial_campaigns, vicidial_log, vicidial_agent_log | Campaign identifier | Connect campaign settings with call logs and agent activity |
| `user` | vicidial_log, vicidial_agent_log, vicidial_list | Agent/user identifier | Track agent activity across calls and lead modifications |
| `list_id` | vicidial_list, vicidial_log | List identifier | Group leads by list and track call outcomes by list |

## Contact Information Fields

| Field Name | Tables | Description | Potential Use |
|------------|--------|-------------|--------------|
| `phone_number` | vicidial_list, vicidial_log | Contact phone number | Match calls to leads even without lead_id |
| `phone_code` | vicidial_list, vicidial_log | Country/area code | Additional matching criteria for phone numbers |
| `alt_phone` | vicidial_list | Alternative phone number | May match with phone_number in vicidial_log |

## Status and Outcome Fields

| Field Name | Tables | Description | Potential Use |
|------------|--------|-------------|--------------|
| `status` | vicidial_list, vicidial_log, vicidial_agent_log | Status/disposition code | Track status changes across the system |
| `called_count` | vicidial_list, vicidial_log | Number of call attempts | Compare call attempts between list and log |

## Timestamp Fields

| Field Name | Tables | Description | Potential Use |
|------------|--------|-------------|--------------|
| `call_date` | vicidial_log | Date/time of call | Time-based correlation with other tables |
| `event_time` | vicidial_agent_log | Time of agent event | Time-based correlation with call logs |
| `last_local_call_time` | vicidial_list | Time of last call | Time-based correlation with call logs |
| `modify_date` | vicidial_list | Last modification date | Track when lead information was updated |

## Grouping and Categorization Fields

| Field Name | Tables | Description | Potential Use |
|------------|--------|-------------|--------------|
| `user_group` | vicidial_log, vicidial_agent_log | User/agent group | Group and filter by agent teams |
| `comments` | vicidial_list, vicidial_log, vicidial_agent_log | Free-text comments | Text analysis across different contexts |

## Time Measurement Fields

| Field Name | Tables | Description | Potential Use |
|------------|--------|-------------|--------------|
| `length_in_sec` | vicidial_log | Call duration in seconds | Compare with agent talk time |
| `talk_sec` | vicidial_agent_log | Agent talk time in seconds | Compare with call duration |
| `wait_sec` | vicidial_agent_log | Agent wait time in seconds | Analyze agent efficiency |
| `dispo_sec` | vicidial_agent_log | Disposition time in seconds | Analyze agent wrap-up time |

## Join Strategies Using Common Fields

### Direct Identifier Joins

These are the most reliable joins, using primary key to foreign key relationships:

```sql
-- Join leads with call logs
FROM vicidial_list vl
JOIN vicidial_log vlog ON vl.lead_id = vlog.lead_id

-- Join call logs with agent activity
FROM vicidial_log vlog
JOIN vicidial_agent_log val ON vlog.uniqueid = val.uniqueid
```

### Contact Information Joins

Useful when direct identifiers are not available:

```sql
-- Join leads with call logs by phone number
FROM vicidial_list vl
JOIN vicidial_log vlog ON vl.phone_number = vlog.phone_number AND vl.phone_code = vlog.phone_code

-- Join leads with call logs by alternative phone
FROM vicidial_list vl
JOIN vicidial_log vlog ON vl.alt_phone = vlog.phone_number
```

### Time-Based Joins

Useful for correlating events that happened around the same time:

```sql
-- Join call logs with agent activity by approximate time
FROM vicidial_log vlog
JOIN vicidial_agent_log val 
  ON val.event_time BETWEEN 
     vlog.call_date AND DATE_ADD(vlog.call_date, INTERVAL vlog.length_in_sec SECOND)
  AND vlog.user = val.user
```

### Multi-Criteria Joins

Combining multiple fields for more precise matching:

```sql
-- Join leads with call logs using multiple criteria
FROM vicidial_list vl
JOIN vicidial_log vlog 
  ON vl.phone_number = vlog.phone_number 
  AND vl.last_local_call_time = vlog.call_date
  AND vl.status = vlog.status
```

## Best Practices for Joining Tables

1. **Use direct identifiers when available**: Always prefer joins on `lead_id`, `uniqueid`, and other primary key/foreign key relationships.

2. **Consider data quality**: Phone numbers and other contact information may have formatting inconsistencies that need to be addressed before joining.

3. **Time-based joins require care**: When joining on timestamps, consider appropriate time windows and time zone differences.

4. **Multi-table joins can be complex**: When joining more than two tables, carefully consider the join order and conditions to avoid cartesian products.

5. **Test with EXPLAIN**: Use the EXPLAIN statement to analyze query performance and optimize join strategies.
