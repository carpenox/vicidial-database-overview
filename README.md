# ViciDB Database Documentation

This documentation provides a comprehensive overview of the MySQL database structure, table purposes, and relationships. The goal is to understand what data is stored in each table and how tables can be joined to produce meaningful results.

## Documentation Files

| File | Description |
|------|-------------|
| [Table Schemas](table_schemas.md) | Detailed information about each table's structure, fields, and purpose |
| [Table Relationships](table_relationships.md) | Documentation of relationships between tables and how they can be joined |
| [Common Fields](common_fields.md) | Lists of common fields across tables that can be used for joins |
| [Table Categories](table_categories.md) | Organization of tables by their functional categories |
| [Database Diagram](database_diagram.md) | Visual representation of the database structure and relationships |
| [Sample Queries](sample_queries.md) | Practical SQL query examples for common tasks |

## Database Overview

The database is for the ViciDial call center system, with tables organized into several functional categories:

### Call Management
Tables for tracking outbound and inbound calls, including call logs, recordings, and call outcomes.

### Lead Management
Tables for storing contact information, lead status, and lead history.

### Agent Management
Tables for tracking agent activity, performance metrics, and time utilization.

### Campaign Management
Tables for configuring and monitoring outbound calling campaigns.

### System Configuration
Tables for system-wide settings, server configuration, and user management.

## Key Relationships

The core relationships in the database revolve around:

1. **Leads** (`vicidial_list`) - Contains contact information and status
2. **Calls** (`vicidial_log`, `vicidial_closer_log`) - Records of outbound and inbound calls
3. **Agents** (`vicidial_agent_log`, `vicidial_users`) - Agent activity and user accounts
4. **Campaigns** (`vicidial_campaigns`) - Campaign configuration and settings

These tables are linked through common fields like `lead_id`, `uniqueid`, `campaign_id`, and `user`.

## Common Join Fields

The most frequently used join fields include:

- `lead_id` - Links leads to calls and agent activity
- `uniqueid` - Unique identifier for calls, links call logs to agent activity
- `campaign_id` - Links campaigns to calls and agent activity
- `user` - Links users/agents to calls and agent activity
- `phone_number` - Can be used to match leads with calls

## Using This Documentation

This documentation is designed to help understand the ViciDial database structure and relationships. Use it to:

1. Understand the purpose and structure of each table
2. Identify relationships between tables for joining in queries
3. Find common fields that can be used for data analysis
4. Create effective queries for reporting and analysis

For practical examples, refer to the [Sample Queries](sample_queries.md) document.
