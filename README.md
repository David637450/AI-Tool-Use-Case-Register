# AI Use Case Governance Triage

Triage proposed AI use cases against the **EU AI Act**, **NIST AI RMF** and **ISO/IEC 42001**, route them proportionately, and keep an auditable AI register. Staff can submit through a web form, or through an AI assistant such as Claude via MCP.

Everything runs in **your own** n8n account. Your use cases, register and emails stay in your accounts.

## How it works

Everything is in **one workflow**, with three sections on the canvas:

| Section | What it does |
|---|---|
| **Ways in** | A hosted **web form**, and an optional **MCP tool** that lets Claude and other AI assistants submit use cases in conversation. |
| **Assessment** | Classifies each use case, decides the route in code, logs it to your register, and answers immediately. All your settings live here. |
| **Human review** | Runs in the background for cases that need a person. Emails your reviewer, waits for their decision, updates the register row and emails the requester the outcome. |

**Routing**

- **Prohibited** (EU AI Act Article 5): rejected automatically.
- **High risk**, or any NIST trustworthiness concern rated High: the reviewer approves or rejects by email.
- **Limited or minimal risk:** approved automatically, with the required controls listed.
- **Low confidence**, or **a suspected attempt to manipulate the assessment:** the reviewer classifies the case and records a decision through an email form.

The model advises and code decides. Routing rules are deterministic and readable in the **Build case record** node. The prompt treats submitted descriptions as data, not instructions, and flags any attempt to steer the classification for human review.

## You will need

- An n8n account (Cloud or self-hosted)
- An **Anthropic API key**. The model sub-node can be swapped for OpenAI, Gemini or Mistral.
- A **Google account** (personal or Google Workspace) for Gmail and Sheets. A dedicated account rather than a personal one is recommended, because n8n's Gmail access covers the whole mailbox. **Create your register in this same account.**
- *Using Microsoft 365?* See [Using Microsoft 365](#using-microsoft-365).

## Setup (about 20 minutes)

### 1. Create your register
Signed into the **same Google account** you'll connect to n8n, create a Google Sheet and paste this header row into cell A1. Then use **Data → Split text to columns** so each header sits in its own column.

```
case_id,submitted_at,requester,email,department,use_case,description,vendor,data_involved,affected,human_review,risk_tier,overall_risk_rating,confidence,rationale,eu_ai_act_reference,nist_ratings,nist_high_concerns,nist_govern,nist_map,nist_measure,nist_manage,iso42001_controls,impact_assessment_required,key_risks,route,escalation_reason,final_risk_tier,reviewer_notes,decision,decided_by,decided_at,next_step
```

### 2. Import the workflow
In n8n, create a new workflow, open the **⋯** menu, choose **Import from file**, and select `ai-use-case-governance-triage.json`. Save it.

### 3. Fill in Settings
Open the **Settings** node and set:
- `reviewer_email`: who approves high-risk and uncertain cases
- `register_spreadsheet_url`: the full URL of your sheet
- `register_sheet_name`: the tab name (usually `Sheet1`)
- `confidence_threshold`: cases below this go to human triage (default `0.7`)

### 4. Connect your credentials
- **Anthropic Chat Model:** your Anthropic API credential
- **Log to AI register** and **Update AI register:** your Google Sheets credential. In **Update AI register**, confirm the column to match on is `case_id`.
- **Request reviewer approval**, **Request triage decision** and **Notify requester:** your Gmail credential

### 5. Publish and test
Save, then publish (activate) the workflow. Publishing stays greyed out until every node has its credential. After any later change, **save and publish again**: the live version only updates when you republish.

Then open the form's **production URL** from the **AI Use Case Intake Form** node, and submit a clearly high-risk case, for example an AI tool that ranks job applicants and sends automated rejections. You should see:
1. A *Pending human review* row appear in your register immediately.
2. An approval email arrive at your reviewer address.
3. After you approve, the same row update to the final decision, and an outcome email go to the requester.

Share the form's production URL with your staff. Consider enabling form authentication if the link will travel beyond your organisation.

## Optional: submit use cases through Claude (MCP)

You need a Claude account on any plan. Free plans allow one custom connector.

1. Open **MCP Server Trigger** and create a **Bearer Auth** credential with a long random token. Treat it like a password: anyone with it can submit cases to your register.
2. Make sure the workflow is published, then copy the trigger's **production URL**.
3. In Claude, go to **Settings → Connectors → Add custom connector**, paste the URL, and select **Continue anyway** when Claude says it couldn't determine how the server signs in. Choose **No sign in**, and add a request header `Authorization` with the value `Bearer YOUR_TOKEN`.
4. Start a chat and describe a use case. Claude gathers the details, calls the tool, and reports the case ID, risk tier and next step.

Human review always runs in the background, so Claude answers in seconds even when a case needs approval.

**Recommended: install the skill.** `SKILL.md` tells Claude how to use the tool well: gather all the details in one go, use the exact allowed values, ask rather than guess, report the result consistently, and never override the tool's decision. Without it, Claude still calls the tool, but less reliably. To install it:
1. Put `SKILL.md` in a folder named `ai-use-case-triage`, and zip the folder.
2. In Claude, go to **Customize → Skills**, click **+**, choose **Create skill → Upload a skill**, and upload the zip.
3. Skills need code execution enabled: on Free, Pro and Max plans this is under **Settings → Capabilities**.

On Claude Team and Enterprise plans, an organisation owner can provision the skill for every member.

## Customising

- **Your AI policy:** edit the prompt in **Classify risk tier** to reflect your risk appetite, approved vendors and internal rules.
- **Routing rules:** edit **Build case record**.
- **A different form tool:** replace the **AI Use Case Intake Form** trigger with Microsoft Forms, Typeform or Google Forms, and map its answers in **Normalise form answers**.
- **Approvals in Slack or Teams:** n8n's Slack and Microsoft Teams nodes support the same send-and-wait-for-response operation as Gmail.

## Using Microsoft 365

- **Email:** replace the three Gmail nodes with **Microsoft Outlook** nodes, using *Send and wait for response* for the two review nodes and *Send a message* for **Notify requester**. Copy across the recipient, subject and message expressions.
- **Register:** replace the two Google Sheets nodes with **Microsoft Excel 365** nodes (append for logging; update matched on `case_id` for the review).
- Many organisations need IT admin approval before a third-party app like n8n can connect to Microsoft 365 or Google Workspace. Allow time for this.

## Troubleshooting

| Problem | Likely cause and fix |
|---|---|
| **Publish is greyed out** | A node is missing its credential. Look for warning triangles, especially on the three email nodes, and save before publishing. |
| *"missing credential: gmailOAuth2"* | Open the named node, set Authentication to OAuth2, select your Gmail credential, save and publish again. |
| **Log to AI register:** *"The caller does not have permission"* | The register isn't owned by or shared with the Google account in your credential, or the URL in **Settings** points to the wrong sheet. |
| **Log to AI register:** *"not found"* | The spreadsheet URL or tab name in **Settings** is wrong, or still a placeholder. |
| **Rows appear but some columns stay empty** | The headers aren't in separate columns, or don't match exactly. Re-paste the header row and split it into columns. |
| **Claude: "Couldn't connect to the server"** | The workflow isn't published, or you used the test URL. Open the production URL in a browser: *"webhook not registered"* means it isn't published. |
| **Claude connects but calls fail with an authorisation error** | The header must be exactly `Authorization` with the value `Bearer ` plus your token. Check for stray spaces. |
| **"Conflicting URL path" when publishing** | Another workflow in your n8n account uses the same trigger path, for example a second copy of this template. Give the trigger in one of them a unique **Path**, or deactivate the other copy. |
| **Changes don't take effect** | Save and publish again. The live workflow runs the last published version. |

## Costs and security

- **Anthropic API usage** is billed to whoever installs the workflow. Each assessment costs a small amount. Set a monthly spend limit in the Anthropic Console.
- **The MCP token** lets anyone who holds it submit cases. It cannot read the register or access your accounts. Share it only with authorised staff, through a password manager, and rotate it when someone leaves.
- **Requester identity isn't verified.** The register records the name and email provided. If you need verified identities, put the form behind company sign-in.

## Important

This workflow provides **decision support, not legal advice**. AI-generated framework mappings can be incomplete or out of date. Have a qualified person validate classifications, and review the routing rules against your own obligations before relying on them.

## About

Built by **Foundation Cyber**, an AI governance and cybersecurity consultancy. For help tailoring this to your organisation, including your AI policy, curated ISO 42001 control mappings and calibration against your real use cases, get in touch: [ADD YOUR WEBSITE / EMAIL]
