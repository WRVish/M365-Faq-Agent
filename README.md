# 🤖 M365 FAQ Agent (Copilot Studio + Power Platform)

> A Microsoft 365 support agent built with Copilot Studio, SharePoint, Power Automate, and Teams. Users chat to get instant answers to M365 questions — Teams, SharePoint, Power Apps, Power Automate, and more — without opening a support ticket.

---

> [!WARNING]
> **This is a learning project, not a production-ready solution.**
> It is built and shared for educational purposes — to demonstrate how Copilot Studio, SharePoint, Power Automate, and Teams work together. It has not been security-reviewed, load-tested, or hardened for enterprise deployment. Do not use this in a live environment without your own thorough testing, security review, and adaptation to your organisation's policies and standards.

---

## What This Does

Most support teams answer the same M365 questions over and over. This agent fixes that by letting users self-serve first, and escalating only what the agent can't confidently answer.

**M365 FAQ Agent handles three things:**

- 💬 **Product-specific FAQ search** — Answers questions about Microsoft Teams, SharePoint, Power Apps, Power Automate, Power Platform, and other tools. Uses Copilot Studio generative answers backed by curated SharePoint FAQ lists.
- 📋 **Intelligent escalation** — When the agent can't find a confident answer, it escalates to IT operations, creates a ticket reference, posts to Teams, sends emails, and logs everything in SharePoint.
- 🎯 **Knowledge base integration** — Separates FAQ content by product. One SharePoint list per product area keeps content organized and searchable.

**Stack:** Copilot Studio · SharePoint · Power Automate · Microsoft Teams · Outlook

---

## Full Documentation

Everything you need to build and deploy this is available in multiple places:


**📚 [Step-by-Step Setup Guide](./docs/step-by-step-guide.md)** — Detailed walkthrough with screenshots for each deployment step.

**🌐 [Blog Post: Building an M365 FAQ Agent](<placeholder>)** — Complete implementation guide with explanations of concepts and architecture decisions (external link).



## Quick Start

### 1. SharePoint Setup
- Create an M365 FAQ Hub site in SharePoint.
- **Create lists:** Run the PowerShell scripts in `ListCreation-Sampledata/` to create all 6 FAQ lists and the escalation log list.
  - Or manually create lists using the column definitions in `list-column-definitions.json`.
- **Load sample data:** Import the CSV files from `ListDataLoad-Sampledata/` into each corresponding SharePoint list.
  - In SharePoint, go to each list → Import → Select CSV → Map columns → Import.

### 2. Power Automate Setup
- Import `power-automate/EscalateToTeams-flow.zip` into your Power Platform environment.
- Update connection references to your SharePoint site URL.
- Update email addresses for ops team and user notifications.
- Turn the flow On.

### 3. Copilot Studio Setup
- Create a new Copilot Studio agent named `M365 FAQ Agent`.
- Connect each SharePoint FAQ list as a federated structured search source.
- Create topics for Conversation Start, product selection (Teams, SharePoint, Power Apps, etc.), and escalation.
- Register the Power Automate flow via the Tools tab.
- Enable generative actions and model knowledge in agent settings.

### 4. Teams Deployment
- Publish the agent to Teams.
- Request admin approval in Teams Admin Centre.
- (Optional) Set up an App Setup Policy to pin the agent for all users.

Full step-by-step in the [Solution Design Document](./docs/solution-design-document.md).

---

## Architecture Overview

```
User / Employee
       ↓
M365 FAQ Agent (Copilot Studio)
       ↓
Conversation Topics
       ├→ SharePoint FAQ Knowledge Sources
       │  (Teams, SharePoint, Power Apps, Power Automate, Power Platform, Other)
       │
       └→ Power Automate Flow (EscalateToTeams)
          ├→ Microsoft Teams (adaptive card to ops channel)
          ├→ Outlook (ops email + user confirmation)
          └→ SharePoint (escalation log)
```

---

## Key Escalation Flow Inputs

When mapping topic variables to the escalation flow in Copilot Studio, use these Power Fx formulas in the ⋯ → Formula tab:

| Flow Input | Formula | Type |
|---|---|---|
| `userQuestion` | `Text(Topic.VarUserQuestion)` | String |
| `userEmail` | `Text(System.User.Email)` | String |
| `product` | `Text(Topic.VarProductSelected)` | String |
| `conversationId` | `Text(conversation.Id)` | String |

Green checkmark = working correctly.

---

## FAQ Lists Recommended Fields

Each SharePoint FAQ list should have:

| Field | Type | Purpose |
|---|---|---|
| Title | Text | Short FAQ question / topic title |
| Answer | Rich HTML | Full answer shown to users |
| Product | Choice | Product category (Teams, SharePoint, etc.) |
| Keywords | Text | Search terms to improve matching |
| LastReviewedDate | Date | When the FAQ was last checked |
| Owner | Person | Person/team responsible |
| IsActive | Yes/No | Whether the FAQ is currently used |

---

## Deployment Checklist

Before production:

- [ ] All 6 SharePoint FAQ lists created and populated with content
- [ ] Escalation log SharePoint list created
- [ ] Power Automate flow imported and connection references updated
- [ ] Flow variables point to correct SharePoint site
- [ ] Email addresses updated (ops team, user notifications)
- [ ] Teams ops channel ID configured in flow
- [ ] Copilot Studio agent created with all topics
- [ ] Knowledge sources connected to SharePoint lists
- [ ] Escalation flow registered in agent Tools
- [ ] Generative actions enabled in agent settings
- [ ] Agent tested with known FAQ questions
- [ ] Agent tested with unknown questions (escalation path)
- [ ] Teams adaptive card posting verified
- [ ] Escalation emails received correctly
- [ ] SharePoint log entries created
- [ ] Agent published to Teams
- [ ] Admin approval obtained in Teams Admin Centre

---

## Common Mistakes & Fixes

| Issue | Cause | Fix |
|---|---|---|
| No answers returned | SharePoint lists empty or not connected | Check list content; verify knowledge sources in agent settings |
| Escalation not posting to Teams | Wrong Teams group/channel ID | Update group and channel IDs in flow actions |
| Emails not sent | Shared mailbox not configured | Update To/From addresses; verify Outlook connector permissions |
| Agent not recognizing questions | Knowledge source not selected for topic | Verify topic references correct knowledge source |
| Users seeing duplicate escalations | Flow triggered multiple times | Add deduplication logic or check trigger conditions |

---

## Testing Scenarios

| Test | Expected Result |
|---|---|
| Start conversation | Agent greets and shows product choices |
| Select Microsoft Teams | Agent asks for a Teams question |
| Ask known FAQ | Agent returns answer from FAQ_Teams list |
| Ask unknown question | Agent escalates, creates ticket reference |
| Check Teams channel | Adaptive card posted with escalation details |
| Check user inbox | Confirmation email received |
| Check SharePoint log | Escalation recorded with ticket reference |
| Select different product | Agent switches to correct FAQ source |
| Say goodbye | Agent ends conversation politely |

---

## About Me

I'm **Vish** — a Microsoft Solutions Architect. I build and share working M365 and Copilot Studio solutions — things I've actually built and debugged, not just polished demos.

- 🌐 Website: [wrvishnu.com](https://www.wrvishnu.com)
- 💼 LinkedIn: [linkedin.com/in/vishnuwr](https://linkedin.com/in/vishnuwr)
- 🐙 GitHub: [github.com/vishpowerlabs](https://github.com/vishpowerlabs)

---

## If This Helped You

If this saved you time or got you unstuck, a ⭐ on the repo goes a long way — it helps others find it.

```bash
git clone https://github.com/WRVish/M365-Faq-Agent.git
```

---

## Contributing

This is an active project and I welcome contributions — whether that's:
- Fixing something that doesn't work in your environment
- Adding new product topics
- Improving flow logic or error handling
- Adding request tracking or analytics
- Improving documentation

If you want to collaborate, reach out:

- 📧 [info@wrvishnu.com](mailto:info@wrvishnu.com)
- 💼 [LinkedIn](https://linkedin.com/in/vishnuwr)

**Please open a PR against a feature branch, not main.**

---

## Feature Requests & Questions

Use **[GitHub Discussions](https://github.com/vishpowerlabs/M365-Faq-Agent/discussions/)** for:

- 💡 Feature requests or extension ideas
- ❓ Questions about specific setup steps
- 🗣️ Sharing how you've adapted this for your organisation

---

## A Note on Learning & Sharing

This repo is part of my commitment to sharing real, working M365 and Copilot Studio builds with the community — not polished demos that never see production, but things that have been built, tested, and documented properly.

**This is shared as a learning resource.** The intent is to give you a working reference you can study, adapt, and extend — not a plug-and-play solution you drop into production unchanged. Treat it as a starting point, not a finished product.

Adapt it to your organization's needs, security standards, and business processes.

---

## License

MIT — free to use, adapt, and build on. Attribution appreciated but not required.

---

*Built by [Vish](https://wrvishnu.com) - May 2026*
