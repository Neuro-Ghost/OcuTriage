# OcuTriage - Ophthalmology Triage AI Agent

Capstone project for the Agentic AI Bootcamp. This version fixes the two review findings:

| Review finding | Fix in v2 |
|---|---|
| Agent had a model and memory but no tools | Four tools are connected to the agent's **Tool** input: `Patient_Record_Lookup`, `Clinic_Knowledge_Base`, `Check_Clinic_Slots`, `Trigger_STAT_Pager` |
| Technical document missing | `Technical_Document_Ophthalmology_Triage_Agent.pdf` (architecture, tools, knowledge sources, guardrails, evaluation, limitations) |

## Files

- `Ophthalmology_Triage_AI_Agent_v2.json` - n8n workflow (import this)
- `index.html` - chat UI that calls the workflow webhook
- `Technical_Document_Ophthalmology_Triage_Agent.pdf` - required technical document
- `README.md` - this file

## Setup (about 5 minutes)

1. In n8n choose **Import from file** and select `Ophthalmology_Triage_AI_Agent_v2.json`.
2. Open the **Google Gemini Chat Model** node and select your Gemini credential (the previous one is kept in the file, but re-select it if n8n shows a warning). Pick a current Gemini Flash or Pro model that supports tool calling.
3. Open the **Ophthalmology Triage AI Agent** node and confirm the four tool nodes are attached under **Tools** (they appear below the agent on the canvas).
4. Test inside n8n first: click **Execute workflow**, then send a POST request to the test URL (or use the UI with "Production webhook" unticked).
5. Publish / activate the workflow so the production URL `http://localhost:5678/webhook/ophthalmology-triage` works.
6. Open `index.html` in a browser (double-click is fine). Click a test scenario.

No credentials other than Gemini are needed: the EHR, calendar, pager and knowledge base are self-contained mock tools.

## Test request without the UI

```bash
curl -X POST http://localhost:5678/webhook/ophthalmology-triage \
  -H "Content-Type: application/json" \
  -d '{"chatInput":"Patient PT-89421 calls with sudden severe right eye pain and severe vision loss.","session_id":"demo-1"}'
```

## Mock patients

| ID | Purpose |
|---|---|
| PT-89421 | Cataract surgery 16 days ago (post-op window) |
| PT-33018 | Intravitreal injection 9 days ago |
| PT-55107 | Monocular patient, glaucoma |
| PT-10233 | Contact lens wearer |
| PT-70654 | No history |

## Evidence to capture for the resubmission

1. Screenshot of the canvas showing the agent with model, memory and **four tools** attached.
2. Screenshot of an execution where the agent called tools (open the agent node, "Intermediate steps").
3. Screenshots of the UI for scenarios A, B, C and E.
4. Fill in the **Result** column of the evaluation table in the PDF (section 6) from your real runs.

## Swapping the simulated pager for real SMS

`Trigger_STAT_Pager` (and the `Fallback STAT Pager` node) are simulated. To send real SMS, delete the `Trigger_STAT_Pager` Code Tool and add a **Twilio Tool** node with the same name connected to the agent's Tool input, and replace the fallback Code node with a Twilio node.

## Troubleshooting

- **Agent never calls tools:** use a Gemini model that supports function calling; keep temperature at 0; check the tool descriptions were not edited.
- **UI shows "Request failed":** workflow not active (use the test URL and click Execute workflow first), wrong base URL, or n8n not running.
- **"trace unavailable" in the log:** enable **Return Intermediate Steps** in the agent options (it is on in the imported file).
- **Tool trace shows an error for a Code Tool:** open the tool node and confirm it is set to JavaScript.

Decision-support prototype using synthetic data. Not a medical device.
