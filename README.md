# Interior Design Lead Management System

## Overview
This system automates the entire process of managing interior design leads through integrated n8n workflows. It handles lead acquisition, call scheduling, tracking, follow-up, and status management. The system leverages n8n for workflow automation, Airtable for data storage, and the Vapi.ai API for automated calling.

## System Architecture

### Data Storage (Airtable)
- **Interior Design Leads Base** containing:
  - **Leads Table**: Stores contact information and tracking status
  - **Call Records Table**: Logs detailed call data and transcripts

### Call Management
- Automated outbound calls using Vapi.ai API
- Call transcription and analysis
- Automatic retry logic for unanswered calls

### Workflow Automation (n8n)
- Schedule-triggered processes for initial outreach
- Webhook-triggered processes for call follow-up
- Status tracking and updates
- Conditional logic for call handling

## Lead Status Tracking

The system tracks leads through the following statuses:

| Status | Description |
|--------|-------------|
| TBC | Initial lead received, awaiting first contact |
| In-Progress | Call is currently active |
| Called | Call completed successfully |
| Failed | Unable to reach after attempts or other failure |

## Workflows

### 1. Automated Lead Management Workflow

This workflow runs on a schedule to identify and process new leads:

#### Components
1. **Schedule Trigger**
   - Runs every 30 minutes
   - Initiates the workflow on a regular schedule

2. **Airtable Query**
   - Searches for records with Status = "TBC"
   - Identifies new leads that need processing

3. **Edit Fields**
   - Prepares data for API call
   - Captures: id, First Name, Mobile

4. **HTTP Request to Vapi.ai**
   - POST to https://api.vapi.ai/call
   - Initiates an automated phone call to the lead
   - Payload:
     ```json
     {
       "assistantId": "8e05fc17-bd5e-4211-9f93-4766ebe5f928",
       "customer": {
         "name": "{{First Name}}",
         "number": "{{Mobile}}"
       },
       "phoneNumberId": "ddeebb3b-0c1c-4fa0-9932-a7524e7c0c0d"
     }
     ```

5. **Airtable Update**
   - Updates lead status to "In-Progress"

6. **Wait**
   - Minimal pause for orderly processing

7. **Follow-up HTTP Request**
   - Triggers webhook for additional processing
   - Extended timeout (200,000 ms)

### 2. Call Attempt and Follow-up Workflow

This webhook-triggered workflow handles call outcomes and retry logic:

#### Components
1. **Webhook**
   - Endpoint: `ad4d22d0-c1a4-461f-acf1-e80381ae41cc`
   - Receives call event data

2. **Event Type Check**
   - Determines if it's an end-of-call report

3. **Raw Data Storage**
   - Records call details in Call Records table

4. **Call Status Analysis**
   - Checks if call was answered (not voicemail)
   - Verifies no technical errors occurred

5. **Lead Update**
   - If successful: Updates as "Called" with summary
   - If unsuccessful: Processes retry logic

6. **Retry Logic**
   - Wait 1 minute
   - Find lead by phone number
   - Update status to "In-Progress" and attempt count
   - Initiate second call via Vapi.ai API

## Call Attempt Logic

1. **Initial Call**: System attempts to reach the lead
2. **Unanswered Call Handling**:
   - If the call isn't answered (voicemail, no answer), the system waits 1 minute
   - The system automatically attempts a second call
   - Lead is temporarily marked as "In-Progress" with attempt count incremented
3. **Final Status**:
   - Successful call: Status updated to "Called" with call summary
   - After 2 failed attempts: Status updated to "Failed" with reason "Unreachable"

## Data Flow

1. **Lead Identification**: Schedule trigger finds TBC leads
2. **Call Initiation**: System triggers outbound call to lead
3. **Call Completion**: System receives end-of-call report via webhook
4. **Data Processing**:
   - Call data stored in Call Records table
   - Lead status updated based on call outcome
5. **Conditional Logic**:
   - Success path: Update lead as "Called" with summary
   - Failure path: Retry or mark as "Failed" based on attempt count

## Technical Components

### Endpoints
- Schedule trigger: Runs every 30 minutes
- Primary webhook for receiving call reports: `ad4d22d0-c1a4-461f-acf1-e80381ae41cc`
- Follow-up webhook: `e67d5a73-152e-45bc-b171-71c3553e6810`

### API Integration
- Vapi.ai API for call management
- Authorization via API tokens

### Data Mapping
- Call metadata stored with each record
- Timestamps for call start/end
- Cost tracking
- Transcript storage

## Implementation Details

### Airtable Schema

#### Leads Table
- First Name, Last Name
- Mobile, Email
- Status (TBC, In-Progress, Called, Failed)
- Attempt (1, 2, 3, Success)
- Date Time (timestamp)
- Summary (call outcome)
- Assignee

#### Call Records Table
- Call provider ID
- Phone number ID
- Customer number
- Status
- Type (outbound)
- Started/Ended timestamps
- Duration (milliseconds)
- Cost
- End reason
- Transcript

## Setup Requirements

### Credentials
- Airtable Personal Access Token
- Vapi.ai API token

### External Services
- Airtable database with appropriate structure
- Vapi.ai account with configured assistant
- n8n instance with webhook capability

## Maintenance and Troubleshooting

### Common Issues
- Webhook connectivity problems
- API rate limits
- Call status tracking discrepancies

### Troubleshooting Steps
- If calls are not being initiated, verify the Vapi.ai credentials and assistant configuration
- If lead status is not updating, check Airtable access permissions
- For webhook failures, verify the endpoint URL and n8n instance connectivity

### Monitoring
- Regularly monitor the workflow execution
- Check n8n execution logs for workflow failures
- Review Airtable records for consistency
- Monitor call costs and usage metrics

## Security Considerations
- API tokens should be securely stored
- Personal data handling follows privacy regulations
- Access to call transcripts should be restricted

## Notes
- The workflow is designed to process multiple leads in sequence
- The webhook trigger allows for extensibility and integration with other systems
- The system automatically handles retry logic for unanswered calls

---

*This documentation covers the automated lead management system as implemented in the n8n workflows. For technical support or modifications, please contact the system administrator.*
