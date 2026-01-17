# Custom GPT: City Requests Action Guide

This guide helps you build a Custom GPT that can file and look up city service requests ("City311") using GPT Actions. It includes a sample OpenAPI spec, example prompts, and implementation notes for a lightweight backend.

## 1) Decide the action scope

Typical actions for a City311 GPT:

- **Create a service request** (e.g., report a pothole, noise complaint, graffiti).
- **Check request status** by request ID.
- **Search requests** by address, category, or time range.
- **List available request types** (so the GPT can guide the user).

## 2) Build a small API that the GPT Action will call

You can implement a simple REST API (Node/Express, Python/FastAPI, etc.) that talks to your city’s 311 service. If you don’t have a direct city API, you can:

- Use the city’s open 311 endpoint (if provided).
- Integrate with an internal ticketing system.
- Use a spreadsheet/DB to store requests during a pilot.

### Example endpoint contract

- `POST /requests` → create a new request
- `GET /requests/{request_id}` → fetch status
- `GET /request-types` → list supported request types
- `GET /requests` → search by `type`, `address`, `from`, `to`

## 3) Example OpenAPI spec for GPT Actions

> Use this in the **Actions** section of the Custom GPT builder.

```yaml
openapi: 3.0.1
info:
  title: City311 Action API
  version: "1.0.0"
servers:
  - url: https://YOUR_DOMAIN.example.com
paths:
  /request-types:
    get:
      operationId: listRequestTypes
      summary: List supported city request types
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    id:
                      type: string
                    name:
                      type: string
  /requests:
    get:
      operationId: searchRequests
      summary: Search requests by filters
      parameters:
        - name: type
          in: query
          schema:
            type: string
        - name: address
          in: query
          schema:
            type: string
        - name: from
          in: query
          schema:
            type: string
            format: date-time
        - name: to
          in: query
          schema:
            type: string
            format: date-time
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    request_id:
                      type: string
                    type:
                      type: string
                    address:
                      type: string
                    status:
                      type: string
                    created_at:
                      type: string
                      format: date-time
    post:
      operationId: createRequest
      summary: Create a new city service request
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [type, address, description]
              properties:
                type:
                  type: string
                address:
                  type: string
                description:
                  type: string
                contact_name:
                  type: string
                contact_phone:
                  type: string
                contact_email:
                  type: string
                location:
                  type: object
                  properties:
                    lat:
                      type: number
                    lon:
                      type: number
      responses:
        "201":
          description: Created
          content:
            application/json:
              schema:
                type: object
                properties:
                  request_id:
                    type: string
                  status:
                    type: string
  /requests/{request_id}:
    get:
      operationId: getRequestStatus
      summary: Get status for a specific request
      parameters:
        - name: request_id
          in: path
          required: true
          schema:
            type: string
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object
                properties:
                  request_id:
                    type: string
                  status:
                    type: string
                  updated_at:
                    type: string
                    format: date-time
```

## 4) Configure the Custom GPT

In the GPT builder:

1. **Name**: “City Requests Assistant” (example)
2. **Instructions** (system prompt):
   - Ask clarifying questions for missing fields (e.g., address, request type).
   - Summarize what will be submitted and ask for confirmation.
   - Be explicit about what data will be shared with the city.
   - For status checks, request the `request_id` if not provided.
3. **Actions**: paste the OpenAPI spec, configure auth (API key or OAuth) if needed.

## 5) Example conversation behavior

- User: “There’s a pothole on my street.”
- GPT: “I can file that. What’s the exact address or nearest intersection?”
- User: “123 Main St.”
- GPT: “Thanks. Please describe the pothole size or any hazards.”
- User: “About 2 feet wide.”
- GPT: “I’m ready to file a Pothole request at 123 Main St with description ‘About 2 feet wide.’ Shall I submit?”
- (After confirmation) GPT calls `POST /requests`.

## 6) Implementation tips

- Log all action requests and responses for auditability.
- Validate addresses and normalize them (geocoding helps).
- Rate-limit to prevent abuse.
- Return clear error messages so the GPT can recover.

## 7) Next steps

- Connect your API to the official city 311 backend if available.
- Add a dashboard for request tracking.
- Expand the supported request types.
