---
name: ai-use-case-triage
description: Submit a proposed AI use case to the organisation's AI governance triage tool. Use whenever someone wants to adopt a new AI tool, use an existing AI tool in a new way, or asks whether an AI use case is allowed, risky or compliant.
---

# AI use case triage

The organisation assesses every proposed AI use case through a governance triage tool, `triage_ai_use_case`. It classifies the use case against the EU AI Act, NIST AI RMF and ISO/IEC 42001, logs it to the organisation's AI register, and decides what happens next. Your job is to gather accurate details, submit them, and explain the result. The tool makes the decision, not you.

## When to use this

Use it when someone:
- wants to start using an AI tool, or use one they already have for something new;
- asks whether an AI use case is allowed, compliant or risky;
- describes a plan that involves AI making or informing decisions, processing personal data, or interacting with customers.

If it is unclear whether the person wants to register a use case or is just asking a general question, ask before submitting. Every submission creates a register entry and may email a reviewer.

## 1. Gather the details

You need all nine inputs. Take what you can from the conversation and ask for the rest in a single message, not one question at a time.

| Input | What to collect |
|---|---|
| `requester_name` | The person's name |
| `requester_email` | Their work email. The decision is sent here. |
| `department` | Exactly one of: `Operations`, `Finance`, `HR`, `Sales & Marketing`, `Customer Service`, `Technology`, `Legal & Compliance`, `Other` |
| `use_case_title` | A short, specific title |
| `description` | What the AI does, who uses it, what data it uses, and what outputs or decisions it produces |
| `vendor` | The tool, vendor or model, e.g. Microsoft Copilot, ChatGPT Enterprise, an in-house model |
| `data_involved` | Exactly one of: `Public data only`, `Internal business data`, `Personal data`, `Special category or biometric data` |
| `affected` | Exactly one of: `Internal staff productivity only`, `Customers or the public`, `Decisions about individuals (hiring, credit, access to services)` |
| `human_review` | Exactly one of: `Yes - always`, `Sometimes`, `No - fully automated` |

Rules:
- **Use the allowed values exactly**, including capitals and punctuation. If the person's answer doesn't map clearly to one value, ask them to choose.
- **Ask rather than guess.** A vague submission gets escalated to manual triage and slows the person down.
- **Choose the more cautious value when evidence points that way.** If customer data includes names or contact details, it is `Personal data`. If outputs affect whether someone gets a job, loan or service, it is `Decisions about individuals`.
- **Write the description in plain, factual terms.** Include anything that raises risk: automated decisions, personal or sensitive data, customer-facing outputs, or limited human oversight. Don't soften or omit details to make approval more likely.

Before submitting, briefly confirm the details with the person if any were inferred rather than stated.

## 2. Submit

Call `triage_ai_use_case` with all nine inputs. Submit each use case once. If the call fails, tell the person and don't retry more than once.

## 3. Report the result

Tell the person, in this order:
1. **Case ID**, so they can refer to it later.
2. **Decision**, in plain words: approved, rejected, or sent for review.
3. **Next step**, using the tool's `next_step` field.
4. **Required controls**, if any, as a short list.
5. **Why**, in one or two sentences, from the rationale and EU AI Act reference.

Then follow the guidance for the route:

| Route | What to tell them |
|---|---|
| `Low` (auto-approved) | They can proceed once the listed controls are in place. Mention if an impact assessment is required before go-live. |
| `Review` | A governance reviewer will decide. They must not start using the tool until it's approved. They'll be emailed the decision. |
| `Needs triage` | The governance team will review it manually and may ask questions. They'll be emailed the outcome. |
| `Prohibited` | The use case falls under a practice prohibited by the EU AI Act and cannot go ahead. Offer to help them find a lower-risk alternative that meets the same goal. |

## Rules

- **Never override, soften or second-guess the tool's decision.** Don't add your own risk rating, and don't suggest the decision is wrong. If the person disagrees, tell them they can raise it with the governance reviewer, quoting the case ID.
- **Never encourage someone to reword a use case to get a different result.** You can help them redesign the use case itself, for example by adding human review or removing personal data, and submit the redesigned version as a new case.
- **This is decision support, not legal advice.** If the person needs a legal view, direct them to their legal or compliance team.
