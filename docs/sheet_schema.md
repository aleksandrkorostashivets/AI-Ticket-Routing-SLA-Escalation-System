# Ticket Database Schema (Google Sheets)

This document describes the structure of the centralized Google Sheet used as the ticket database for the AI Ticket Routing & SLA Escalation System.

Create a sheet named **`Tickets`** with the following columns (row 1 = headers):

| Column | Field | Type | Description |
|---|---|---|---|
| A | `ticket_id` | string | Unique identifier (e.g. `TCK-00001`), generated on creation |
| B | `received_at` | datetime (ISO 8601) | Timestamp the email was received in Gmail |
| C | `sender_email` | string | Customer's email address |
| D | `sender_name` | string | Customer's display name, if available |
| E | `subject` | string | Original email subject line |
| F | `body_summary` | string | AI-generated short summary of the email body |
| G | `category` | string | `delivery` \| `payment` \| `quality` \| `technical` \| `other` |
| H | `priority` | string | `P1` \| `P2` \| `P3` \| `P4` |
| I | `sentiment` | string | `negative` \| `neutral` \| `positive` |
| J | `assigned_owner` | string | Agent assigned to handle the ticket |
| K | `team_lead` | string | Team lead responsible if escalated to Level 1 |
| L | `director_email` | string | Director email used for Level 2 escalation |
| M | `sla_deadline` | datetime (ISO 8601) | Calculated deadline based on priority |
| N | `status` | string | `open` \| `in_progress` \| `resolved` \| `closed` |
| O | `escalation_level` | integer | `0` (none) \| `1` (team lead) \| `2` (director) |
| P | `last_notified_at` | datetime (ISO 8601) | Timestamp of the last Slack/email notification sent |
| Q | `resolved_at` | datetime (ISO 8601) | Timestamp the ticket was marked resolved/closed |
| R | `gmail_message_id` | string | Original Gmail message ID, used to avoid duplicate processing |

## Notes

- **`ticket_id`** should be generated once on row creation and never change — use it as the lookup key for the SLA Monitor workflow.
- **`sla_deadline`** is calculated by the classification workflow as `received_at + SLA window for priority`. Suggested default windows:
  - P1 → 30 minutes
  - P2 → 2 hours
  - P3 → 8 hours
  - P4 → 24 hours
  Adjust these to match your actual support commitments.
- **`escalation_level`** should only ever increase, never decrease, while a ticket is open. The SLA Monitor workflow reads this column before sending a new notification, so a ticket already at Level 1 doesn't get re-sent a Level 0 reminder.
- **`status`** should be updated to `resolved` or `closed` by agents (manually, or via a separate "close ticket" workflow/form) — this is what removes a ticket from the SLA Monitor's active check.
- Keep `gmail_message_id` indexed/checked on the classification workflow's first step to prevent the same email being turned into duplicate tickets if Gmail re-delivers a message.

## Suggested additional sheet: `Escalation_Log`

For auditing, you can log every escalation event separately:

| Column | Field | Description |
|---|---|---|
| A | `ticket_id` | Reference to the ticket |
| B | `escalation_level` | Level reached (0/1/2) |
| C | `triggered_at` | Timestamp of the escalation |
| D | `notified_channel` | Slack channel/user or email address notified |
| E | `minutes_overdue` | How many minutes past SLA deadline at trigger time |

This keeps the main `Tickets` sheet clean while preserving full escalation history for reporting.
