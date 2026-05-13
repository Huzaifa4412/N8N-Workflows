# AI Recruitment Screening and Retell Calling Workflow

An n8n workflow that automates CV screening for a Senior Sales Professional / FMCG Sales Manager role. It reads unread Gmail messages with PDF CV attachments, extracts candidate information, screens candidates with Google Gemini, sends shortlisted profiles to Slack, calls qualified candidates through Retell AI, and posts call results back to Slack.

## Overview

This workflow helps automate the first stage of recruitment by combining email intake, AI CV analysis, candidate shortlisting, Slack notifications, and AI voice calling.

It is designed for recruitment teams, HR operations, staffing agencies, and businesses that receive CVs by email and want to quickly identify suitable candidates before manual review.

## What This Workflow Does

- Watches Gmail for unread emails with attachments
- Downloads PDF CV attachments
- Extracts text from candidate CVs
- Sends CV text to an AI recruitment screening agent
- Evaluates candidates against a defined FMCG sales role
- Produces structured candidate screening output
- Checks whether the candidate should be shortlisted
- Sends shortlisted candidate details to Slack
- Cleans and validates the candidate phone number
- Calls the candidate using Retell AI
- Receives Retell AI call analysis through a webhook
- Sends call summary, sentiment, status, and transcript-related details to Slack

## Workflow Architecture

```text
Gmail Trigger
  -> Get Message With Attachments
  -> Prepare PDF Attachments
  -> Extract CV Text
  -> Prepare AI Input
  -> AI Recruitment Screening Agent
  -> Normalize Screening Result
  -> Candidate Qualifies?
      -> Send Candidate to Slack
      -> Clean Phone For Retell
      -> Has Valid Phone?
          -> Call Candidate With Retell AI

Retell Webhook
  -> Is Call Analyzed?
  -> Normalize Retell Result
  -> Send Call Result to Slack
  -> Respond to Retell
```

## Main Use Case

The workflow is built for screening candidates for this role:

**Job Title:** Senior Sales Professional / FMCG Sales Manager

The AI agent checks the CV against requirements such as:

- 8 to 15+ years of FMCG sales and distribution experience
- Territory management
- Market expansion
- Sales target and KPI achievement
- Distributor and dealer network management
- Sales team leadership
- Key account management
- Sales forecasting
- Credit control and receivables management
- Route planning and market coverage
- Relevant business, commerce, marketing, or related education

## Key Integrations

### Gmail

Used to detect unread emails and download CV attachments.

Required credential:

- Gmail OAuth2 credential

### Google Gemini

Used as the AI model for CV analysis and structured candidate screening.

Required credential:

- Google Gemini / PaLM API credential

### Slack

Used to notify the team when:

- A candidate is shortlisted
- A Retell AI call is completed and analyzed

Required credential:

- Slack OAuth2 credential

### Retell AI

Used to place an outbound AI phone call to shortlisted candidates.

Required setup:

- Retell AI API key
- Retell phone number
- Retell agent ID
- Retell webhook URL connected to this n8n workflow

## Required Credentials

Before using this workflow, configure these credentials inside n8n:

| Service | Purpose |
| --- | --- |
| Gmail OAuth2 | Read emails and download CV attachments |
| Google Gemini / PaLM API | Analyze CV content with AI |
| Slack OAuth2 | Send candidate and call result notifications |
| Retell AI API Key | Create outbound AI phone calls |

Do not hardcode secrets directly in production workflows. Store API keys and credentials securely using n8n credentials or environment variables.

## Input

The workflow expects unread Gmail messages that include PDF CV attachments.

Supported attachment type:

- `.pdf`

The workflow extracts:

- Email ID
- Thread ID
- Email subject
- Sender email
- PDF file name
- CV text

## AI Screening Output

The AI agent returns structured JSON with fields such as:

- `job_title`
- `decision`
- `match_score`
- `summary`
- `matched_skills`
- `missing_skills`
- `matched_requirements`
- `missing_requirements`
- `years_of_experience`
- `industry_experience`
- `education`
- `recommended`
- `rejection_reason`
- `candidate_details`

Allowed screening decisions:

- `GOOD MATCH`
- `PARTIAL MATCH`
- `NOT A MATCH`

Candidates are sent to Slack only when:

- `recommended` is `true`
- `decision` is `GOOD MATCH` or `PARTIAL MATCH`

## Phone Number Handling

The workflow includes a phone cleaning step before calling the candidate.

It normalizes common phone formats by:

- Removing spaces, brackets, dots, and dashes
- Converting numbers starting with `00` to international format
- Adding `+` when the number starts with country code `92`
- Converting local Pakistan numbers starting with `0` to `+92`

The candidate is called only if the final number matches a valid international phone format.

## Retell AI Call Flow

When a candidate qualifies and has a valid phone number, the workflow sends a request to Retell AI to create an outbound phone call.

Dynamic variables passed to Retell AI include:

- Candidate name
- Job title
- Match score
- Candidate summary
- Matched skills
- Missing skills

Metadata passed to Retell AI includes:

- Candidate name
- Candidate email
- Candidate phone
- Source CV file
- Email subject

## Retell Webhook

The workflow exposes a webhook endpoint for Retell AI call results:

```text
POST /webhook/retell-call-result
```

The webhook listens for Retell events and processes only:

```text
call_analyzed
```

After the call is analyzed, the workflow sends a Slack message containing:

- Candidate name
- Phone number
- Email
- Call status
- Call success result
- User sentiment
- Voicemail status
- Disconnection reason
- Call summary
- Source file
- Call ID

## Slack Notifications

### Shortlisted Candidate Notification

The first Slack message includes:

- Job title
- Decision
- Match score
- Candidate name
- Candidate email
- Candidate phone
- Candidate location
- AI-generated summary
- Source file

### Call Result Notification

The second Slack message includes:

- Candidate details
- Call status
- Call success
- Sentiment
- Voicemail status
- Disconnection reason
- Call summary
- Source file
- Retell call ID

## Setup Instructions

1. Import the workflow JSON into n8n.
2. Connect your Gmail OAuth2 credential.
3. Connect your Google Gemini / PaLM API credential.
4. Connect your Slack OAuth2 credential.
5. Add your Retell AI API key securely.
6. Update the Retell `from_number`.
7. Update the Retell `override_agent_id`.
8. Update the Slack channel where candidate alerts should be sent.
9. Configure the Retell webhook URL in your Retell AI dashboard.
10. Test the workflow with a sample email containing a PDF CV.
11. Activate the workflow after successful testing.

## Recommended Environment Variables

For production use, store sensitive values as environment variables or n8n credentials:

```text
RETELL_API_KEY=
RETELL_FROM_NUMBER=
RETELL_AGENT_ID=
SLACK_CHANNEL_ID=
```

## Security Notes

- Never commit live API keys, OAuth credentials, phone numbers, or private candidate data to a public repository.
- Rotate any API key that has been exposed in exported workflow JSON.
- Candidate CVs may contain personal data, so keep workflow exports private unless all sensitive data is removed.
- Limit access to the Slack channel receiving candidate and call details.
- Review local privacy and consent requirements before placing automated candidate calls.

## Customization Ideas

- Add Google Sheets or Airtable to store candidate screening results
- Add a database for recruitment pipeline tracking
- Add email replies for rejected candidates
- Add different job roles with dynamic screening criteria
- Add score-based routing for HR review
- Add error handling and failure notifications
- Add duplicate candidate detection
- Add manual approval before Retell AI calls

## Suggested Folder Structure

```text
recruitment-ai-retell-workflow/
├── workflows/
│   └── ai-recruitment-screening-retell.json
├── docs/
│   └── setup-notes.md
├── README.md
└── .gitignore
```

## Production Checklist

- [ ] Credentials are stored securely
- [ ] Retell API key is not hardcoded
- [ ] Slack channel ID is correct
- [ ] Gmail trigger is connected to the correct inbox
- [ ] PDF extraction works with sample CVs
- [ ] AI output parser returns valid JSON
- [ ] Candidate qualification condition works correctly
- [ ] Phone number cleaning works for target regions
- [ ] Retell call is tested with a safe phone number
- [ ] Retell webhook returns `{ "ok": true }`
- [ ] Slack receives both screening and call result messages
- [ ] Error handling is added for production use

## License

Use this workflow for personal, internal, or client automation projects. Add a license file if you plan to publish this repository as open source.
