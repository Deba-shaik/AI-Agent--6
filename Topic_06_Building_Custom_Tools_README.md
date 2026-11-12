# Topic 06 — Building Custom Tools via Workflow, Code, and API Nodes

## Objective
Extend an n8n AI Agent with a custom JavaScript age-calculation tool and an HTTP Request tool for mock customer profile retrieval; demonstrate independent and combined tool execution.

## Workflow architecture
When chat message received → AI Agent
- Chat Model: OpenAI Chat Model (custom-qwen-agent, temperature 0.0; Responses API disabled during troubleshooting)
- Memory: Simple Memory
- Tool 1: fetch_profile_tool — HTTP GET to JSONPlaceholder /users/{customer_id}, with customer_id provided by the AI Agent
- Tool 2: calculate_age_tool — JavaScript Code Tool, accepts a YYYY-MM-DD birthdate string

## Tool implementation
### fetch_profile_tool
Description: Retrieve a customer's name, email, and company using their customer ID.
Method: GET. Authentication: None.
URL expression:
`{{ 'https://jsonplaceholder.typicode.com/users/' + $fromAI('customer_id', 'Numeric customer ID to look up', 'number') }}`

### calculate_age_tool
Description: Calculate a user's age from a YYYY-MM-DD birthdate.
The tool receives the date as a plain text `query` value (not a JSON object). The final JavaScript validates the YYYY-MM-DD date, checks that it is a real non-future date, calculates the year difference, and adjusts if the birthday has not yet occurred.

```javascript
const birthdate = query.trim();
if (!/^\d{4}-\d{2}-\d{2}$/.test(birthdate)) {
  return "Error: Birthdate must be in YYYY-MM-DD format.";
}
const birthDate = new Date(birthdate + "T00:00:00Z");
const today = new Date();
if (isNaN(birthDate.getTime()) ||
    birthDate.toISOString().slice(0, 10) !== birthdate ||
    birthDate > today) {
  return "Error: Invalid birthdate.";
}
let age = today.getUTCFullYear() - birthDate.getUTCFullYear();
const monthDifference = today.getUTCMonth() - birthDate.getUTCMonth();
if (monthDifference < 0 ||
    (monthDifference === 0 && today.getUTCDate() < birthDate.getUTCDate())) age--;
return `Age calculated successfully: ${age} years old.`;
```

## System instructions
The AI Agent is instructed to retrieve profiles when given a customer ID, call the age tool when given a birthdate and an age question, combine tool results, avoid invented customer details, and report tool failures.

## Tests and evidence
1. Profile-only test: Customer ID 2 → Ervin Howell, Shanna@melissa.tv, Deckow-Crist. HTTP tool and AI Agent completed successfully.
2. Age-only test: Birthdate 1991-04-12 → 35 years old on 12 November 2026. Code Tool and AI Agent completed successfully.
3. Combined test: Customer ID 2 and birthdate 1991-04-12 in one prompt → both tools executed (green checkmarks), merged response includes Ervin Howell, Deckow-Crist, and age 35.

## Troubleshooting performed
- Initial model call returned an empty error; disabling Use Responses API resolved the execution issue in this setup.
- Initial age tool attempted `JSON.parse(query)` on a raw date string, resulting in a JSON syntax error. Updated the code to use `query.trim()` and validate the date directly.

## Outcome
All three demonstrated scenarios completed successfully. The HTTP endpoint is a public mock data service, not a production CRM. The evidence does not establish deployment/publishing or broader negative-test coverage.

## Supporting artifacts
See the Evidence PDF for screenshots of the actual n8n executions and this package's Video Script for a 5–6 minute recording plan.
