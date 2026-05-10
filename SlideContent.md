
### Slide 1 - Title

**M365 FAQ Agent**

An AI-powered FAQ and escalation assistant for Microsoft 365 support.

Presented for students and beginners.

Speaker notes:

Today we will look at a practical Microsoft 365 support chatbot built with Copilot Studio, SharePoint, Power Automate, Teams, and Outlook.

### Slide 2 - Problem

**The support challenge**

- Users ask repeated Microsoft 365 questions.
- IT teams answer the same questions many times.
- Beginners may not know where to find official help.
- Some questions need human support.

Speaker notes:

The problem is simple: many support questions are repeated. A chatbot can answer common questions quickly and send difficult questions to the right team.

### Slide 3 - Solution Overview

**What the M365 FAQ Agent does**

- Greets the user.
- Lets the user choose a product.
- Searches a product-specific FAQ source.
- Answers from curated SharePoint content.
- Escalates unanswered questions to IT operations.
- Sends notifications and logs the request.

Speaker notes:

The agent is not just a chatbot. It is connected to business systems, so it can search knowledge and create escalation records.

### Slide 4 - Audience

**Who this helps**

- Students learning Microsoft 365.
- Beginners learning Power Platform.
- Employees who need quick help.
- IT teams handling repeated questions.

Speaker notes:

This is a good learning project because it uses real Microsoft 365 services in a simple support scenario.

### Slide 5 - Products Covered

**FAQ categories**

- Microsoft Teams
- SharePoint
- Power Apps
- Power Automate
- Power Platform
- Other non-M365 tools

Speaker notes:

The solution separates knowledge by product. This makes the FAQ easier to maintain and makes answers more focused.

### Slide 6 - How the Chat Works

**Basic conversation flow**

1. User starts chat.
2. Agent asks for product.
3. User selects product.
4. Agent asks for question.
5. Agent searches FAQ.
6. Agent answers or escalates.

Speaker notes:

This flow is beginner-friendly because the user does not need to know where information is stored. The agent guides them step by step.

### Slide 7 - Solution Architecture

**Main building blocks**

- Copilot Studio agent for chat.
- SharePoint lists for FAQ knowledge.
- Power Automate for escalation.
- Microsoft Teams for ops notification.
- Outlook for email confirmation.
- SharePoint list for escalation logging.

Speaker notes:

Each Microsoft service has a clear job. Copilot Studio talks to the user, SharePoint stores knowledge, and Power Automate connects the process together.

### Slide 8 - Knowledge Source Design

**Why SharePoint is used**

- Easy for teams to maintain FAQ content.
- Supports product-specific lists.
- Keeps answers inside the organization.
- Can be reviewed and updated by IT owners.

Speaker notes:

Instead of asking the agent to guess, the solution uses curated SharePoint FAQ lists. This makes the answers more controlled and reliable.

### Slide 9 - Escalation Flow

**When the bot cannot answer**

- Creates a ticket reference.
- Posts an adaptive card to Teams.
- Sends an email to the operations team.
- Sends a confirmation email to the user.
- Saves the escalation in SharePoint.

Speaker notes:

Escalation is important because the user should not get stuck. If the agent cannot answer, the support team is notified.

### Slide 10 - Example User Journey

**Example**

- User selects Microsoft Teams.
- User asks: "How do I add a guest to a team?"
- Agent searches the Teams FAQ list.
- If answer exists, it replies with steps.
- If no confident answer exists, it creates an escalation ticket.

Speaker notes:

This example shows the difference between self-service and assisted support. The same chatbot can handle both.

### Slide 11 - Benefits

**Why this solution is useful**

- Faster answers for users.
- Less repeated work for IT.
- Clear escalation path.
- FAQ gaps become visible.
- Easy to extend with more topics.

Speaker notes:

The biggest benefit is that support becomes more organized. Easy questions are answered automatically and difficult questions are tracked.

### Slide 12 - What Beginners Can Learn

**Learning outcomes**

- How a Copilot Studio agent is structured.
- How topics control conversation flow.
- How SharePoint can be a knowledge source.
- How Power Automate connects services.
- How Teams and email can support escalation.

Speaker notes:

This project is a strong beginner example because it shows an end-to-end business process without requiring custom code.

### Slide 13 - Current Limitations

**Possible next steps**

- Add analytics dashboard for FAQ usage.
- Add approval process for new FAQ content.
- Add multilingual answers.
- Add user satisfaction reporting.
- Add adaptive card actions for support staff response.

Speaker notes:

Once the basic agent works, the next step is improving operations: reporting, content governance, and better support workflows.

### Slide 15 - Demo

1. Open the M365 FAQ Agent.
2. Show the product selection menu.
3. Select Microsoft Teams.
4. Ask a known Teams FAQ question.
5. Show the answer from the knowledge source.
6. Choose "Escalate to IT team".
7. Show the tracking number response.
8. Show the Teams adaptive card.
9. Show the email notification.
10. Show the SharePoint escalation log item.


### Slide 16 - Summary

**Key takeaway**

M365 FAQ Agent shows how Microsoft 365 and Power Platform can be combined to create a practical AI support assistant.

It helps users get answers faster, helps IT teams reduce repeated work, and gives beginners a clear example of a real-world low-code solution.

Speaker notes:

The solution is valuable because it is simple to understand but covers real business needs: knowledge search, automation, escalation, and tracking.

