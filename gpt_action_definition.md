# GPT Action Definition: get_311_service_requests

## Description
A governed, read-only GPT Action for NYC311 staff to retrieve aggregated service request metrics from an internal API wrapper over the public NYC 311 Service Requests dataset. This action supports analytics and reporting use cases for analysts and supervisors, with strict controls to prevent data mutation, SQL/database access, or PII exposure.

## Action Name
`get_311_service_requests`

## Purpose
Retrieve aggregated NYC311 service request metrics filtered by borough, complaint type, status, and date range.

## Input JSON Schema
```json
{
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "borough": {
      "type": "string",
      "enum": ["Bronx", "Brooklyn", "Manhattan", "Queens", "Staten Island"],
      "description": "NYC borough to filter results."
    },
    "complaint_type": {
      "type": "string",
      "description": "Normalized complaint category (e.g., 'Noise - Residential', 'Street Condition')."
    },
    "status": {
      "type": "string",
      "enum": ["Open", "Closed", "All"],
      "description": "Status filter for requests."
    },
    "start_date": {
      "type": "string",
      "format": "date",
      "description": "Start date (ISO 8601, YYYY-MM-DD)."
    },
    "end_date": {
      "type": "string",
      "format": "date",
      "description": "End date (ISO 8601, YYYY-MM-DD)."
    },
    "aggregation_level": {
      "type": "string",
      "enum": ["daily", "weekly", "monthly"],
      "description": "Time aggregation for trend summaries."
    }
  },
  "required": ["borough", "complaint_type", "status", "start_date", "end_date", "aggregation_level"]
}
```

## Output JSON Schema
```json
{
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "total_requests": {
      "type": "number",
      "description": "Total number of requests matching the filters."
    },
    "requests_by_agency": {
      "type": "object",
      "description": "Counts grouped by responsible agency.",
      "additionalProperties": {"type": "number"}
    },
    "requests_by_borough": {
      "type": "object",
      "description": "Counts grouped by borough.",
      "additionalProperties": {"type": "number"}
    },
    "trend_summary": {
      "type": "string",
      "description": "Human-readable summary of trends for the selected aggregation level."
    },
    "data_last_updated": {
      "type": "string",
      "format": "date-time",
      "description": "Timestamp of the most recent data refresh."
    }
  },
  "required": [
    "total_requests",
    "requests_by_agency",
    "requests_by_borough",
    "trend_summary",
    "data_last_updated"
  ]
}
```

## Example User Prompt
"Show me weekly counts of open residential noise complaints in Brooklyn from 2024-01-01 to 2024-03-31, and summarize trends."

## Example API Response
```json
{
  "total_requests": 1842,
  "requests_by_agency": {
    "NYPD": 1430,
    "DEP": 312,
    "DOB": 100
  },
  "requests_by_borough": {
    "Brooklyn": 1842
  },
  "trend_summary": "Weekly totals were stable in January, rose ~18% in February, and leveled off through March.",
  "data_last_updated": "2024-04-01T09:30:00Z"
}
```

## Testing & Validation
Because this is a GPT Action definition, the primary validation is schema conformance and a controlled API smoke test. Recommended steps:

1. **Schema check**: Validate a sample request against the input schema and a sample response against the output schema using a JSON Schema validator in your internal tooling.
2. **Smoke test (staging only)**: Call the internal governed API wrapper with a known, low-volume query and verify:
   - Response fields match the output schema.
   - Only aggregate data is returned (no request-level records or PII).
   - `data_last_updated` is present and current.
3. **Rate-limit/audit verification**: Confirm the request is logged and that rate limits are enforced for repeated calls.
4. **Human-in-the-loop check**: Ensure the action only runs when explicitly invoked by a user request in ChatGPT Enterprise.

## Governance & Public-Sector Safety Notes
- **Read-only access**: The action only retrieves aggregated metrics and does not allow writes, updates, or deletions.
- **No database/SQL access**: The action is backed by a governed internal API wrapper, not direct database access or SQL execution.
- **No PII exposure**: The action returns aggregate counts and summaries only; no request-level or personal data is exposed.
- **Rate-limited**: Requests are subject to organizational rate limits to protect service availability.
- **Auditable**: All calls are logged for compliance and oversight.
- **Human-in-the-loop**: The action is invoked only in response to explicit user requests; no autonomous execution or chaining.
