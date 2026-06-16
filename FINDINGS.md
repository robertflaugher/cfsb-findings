# CFSB Field Findings

**Maintainer:** Independent CFSB developer and consultant
**Platform:** Claude for Small Business (CFSB), launched May 13, 2026
**Environment:** Claude Desktop + Claude Cowork + Small Business plugin, Mac, Pro account
**Last updated:** 2026-06-13
**Status:** Active — updated after each documented skill run

---

## Purpose

This file documents real platform behavior observed during systematic testing of Claude for Small Business. It is not a review. It is a field log.

Two audiences are intended.

**SMB owners and evaluators** — This log gives you an honest picture of what the platform does well, where it stumbles, and what you need to set up before your first skill run produces useful output.

**Anthropic and CFSB developers** — This log surfaces edge cases, failure modes, and UX friction points encountered during real-world testing. It is written as structured field notes, not a complaint. Every finding is tied to a specific observed behavior. Corrections and clarifications are welcome via GitHub issues.

Findings are organized by skill run or platform behavior category. Within each section, findings are numbered sequentially starting at F-1. Status tags indicate current standing: **Confirmed**, **Unconfirmed**, or **Retracted**.

---

## Testing Methodology

All findings were produced under the following conditions unless noted otherwise.

- Claude Desktop on macOS, Claude Pro account
- Claude Cowork with the Small Business plugin installed
- Connectors tested: QuickBooks Online (production trial account), HubSpot (developer test account, Enterprise tier, free), Gmail, Google Calendar, Google Drive
- Location: Bangkok, Thailand
- VPN: ExpressVPN (Los Angeles endpoint), tested both on and off where noted
- Screen recording via Loom for all live runs
- Timing: wall-clock time from skill invocation to completion

**Finding status definitions**

**Confirmed** — observed in at least one documented run and consistent with other evidence.
**Unconfirmed** — observed once but not yet replicated under controlled conditions.
**Retracted** — initially logged but subsequently disproved by controlled testing.

**Investigation types**

Findings originate from two sources. Tool Tests (TT) are findings produced during planned, documented skill runs. Community-Triggered investigations (CT) are added to the queue because of signals from outside my own testing — posts, articles, or claims made by other CFSB developers or community members.

---

## Skill Run: /monday-brief

**Run TT-1a** — First run. Gmail connector added mid-execution. Runtime: 36 minutes.
**Run TT-1b** — Second run. All connectors pre-authorized. Runtime: 3 minutes 25 seconds.
**Run TT-1c** — VPN comparison. ExpressVPN (Los Angeles) vs VPN off, both from Bangkok. Runtimes: 1:50 and 1:57.

---

**F-1 — Slash command invocation is unreliable**
**Status:** Confirmed

`/monday-brief` invoked successfully via slash command in TT-1a. `/call-list` returned "Unknown skill: call-list" in the same environment. Slash command reliability is inconsistent across skills and sessions.

---

**F-2 — Plain English invocation works where slash commands fail**
**Status:** Confirmed

Plain English skill description successfully triggered /monday-brief in the same session where slash command invocation had previously failed. Plain English is the more reliable invocation method.

**Recommendation:** Do not rely on slash command invocation. Use plain English descriptions as your default invocation method until slash command reliability is confirmed.

---

**F-3 — Mid-run connector authorization is permitted but costly**
**Status:** Confirmed

Cowork allowed a mid-run Gmail connector authorization when /monday-brief detected Gmail was missing. The skill did not abort. However, mid-run connector addition contributed to a 36-minute runtime for TT-1a.

---

**F-4 — Mid-run connector authorization does not fully complete the handshake**
**Status:** Confirmed

Despite being authorized mid-run in TT-1a, Gmail showed as needing reconnection in the /monday-brief output at the end of that same run. Mid-run authorization is insufficient for reliable connector state.

---

**F-5 — Runtime drops 10x when all connectors are pre-authorized**
**Status:** Confirmed

TT-1a (mid-run Gmail addition): 36 minutes. TT-1b (all connectors pre-authorized): 3 minutes 25 seconds. The difference is entirely attributable to connector state at invocation time.

---

**F-6 — Pre-authorization requirement is not communicated by the platform**
**Status:** Confirmed

Pre-authorizing all connectors before invoking any skill is a prerequisite for normal runtime. The platform communicates this nowhere — not during onboarding, not in skill documentation, not at invocation time.

**Recommendation:** Before running any skill, verify all required connectors are authorized and active. Do not add connectors mid-run.

---

**F-7 — Skills stall silently on permission prompts with no alert mechanism**
**Status:** Confirmed

Skills that trigger permission prompts mid-run require the user to actively monitor execution and respond. There is no notification or alert when a prompt appears. A skill will stall indefinitely and silently until the user responds. This is a significant UX problem for SMB owners who expect low-supervision behavior from an AI assistant.

---

**F-8 — "Always allow" permission prompt scope is ambiguous**
**Status:** Confirmed

The "Always allow" permission prompt on QBO Balance Sheet tool calls reappeared in TT-1b despite QBO having been previously authorized. The button label "Always allow" does not communicate what scope the grant covers, whether it persists across sessions, or whether it applies to the connector globally or only to the specific tool call type.

---

**F-9 — QBO permission prompts recur across sessions**
**Status:** Confirmed

QuickBooks permission prompts reappeared during TT-1b despite QBO having been previously authorized in TT-1a. Whether permanent per-connector authorization is achievable through any available mechanism is an open question.

---

**F-10 — Gmail connector produced accurate live output**
**Status:** Confirmed

/monday-brief pulled live Gmail watch-list items accurately in TT-1b. This is the first confirmed evidence of the Gmail connector producing meaningful output in a skill run.

---

**F-11 — Google Drive save silently skipped when connector is absent**
**Status:** Confirmed

In TT-1a, /monday-brief attempted to save the brief as a markdown file to Google Drive, but Google Drive was not connected at that time. The skill proceeded without error and did not warn the user that the save would fail.

---

**F-12 — Google Drive save silently skipped even when connector is active**
**Status:** Confirmed

In TT-1b and TT-1c, /monday-brief failed to save the brief to Google Drive despite the Google Drive connector being active and authorized. The Progress tab showed no Google Drive activity during either run. The skill is not attempting the save at all — it is silently ignoring the connected connector.

**Impact:** Users who expect output saved to Google Drive will receive no file and no error. The failure is invisible in both directions: no save attempt, no warning.

---

**F-13 — VPN does not cause skill stalls**
**Status:** Retracted

ExpressVPN initially appeared to cause skill stalls. Controlled testing did not confirm this. The apparent stall was caused by session state corruption from hitting Escape to abort a prior run, not VPN interference.

---

**F-14 — VPN routing to a US endpoint does not materially affect runtime**
**Status:** Confirmed

TT-1c controlled comparison — ExpressVPN set to Los Angeles versus VPN off, both from Bangkok — produced runtimes of 1:50 and 1:57 respectively. The difference is within normal variance. VPN routing does not materially affect /monday-brief runtime under clean session conditions.

---

## Skill Run: /call-list

**Run TT-1 (partial)** — Run from Bangkok with VPN on. Gmail and Google Calendar not connected at time of run.

---

**F-1 — Runtime of 2:26 from Bangkok with VPN on**
**Status:** Unconfirmed

/call-list completed in 2 minutes 26 seconds when run from Bangkok with VPN on. Whether this reflects geography, VPN routing, session state, or normal variance is unconfirmed. A clean controlled retest with all connectors pre-authorized and VPN off is pending (TT-5).

---

**F-2 — Graceful degradation when connectors are missing**
**Status:** Confirmed

/call-list generated a usable call list despite Gmail and Google Calendar not being connected. It explicitly told the user what additional value those connectors would add. This is well-designed behavior.

---

**F-3 — Contact selection logic is opaque**
**Status:** Confirmed

/call-list offered to write tone-matched follow-up emails for two specific contacts without explaining why those contacts were selected over others. The ranking and selection logic is not surfaced to the user.

---

**F-4 — Output quality cannot be validated without purpose-built test data**
**Status:** Confirmed

/call-list output cannot be validated against known ground truth without deliberately seeding CRM data with contacts of known relative priority. Output quality verification requires synthetic test data with a known correct answer.

**Recommendation:** Before deploying /call-list with a real client, seed HubSpot with test contacts of known priority ordering and confirm the skill ranks them correctly.

---

## Platform Behavior: Connector Setup and Authentication

Findings in this section were produced during connector setup across multiple sessions, primarily during onboarding and the first skill runs.

---

**F-1 — No data security explanation during QuickBooks connection**
**Status:** Confirmed

CFSB onboarding does not explain what Claude can see or do with QuickBooks data at any point during the connection flow. A real SMB client connecting their financial data receives no disclosure.

---

**F-2 — OAuth error message uses engineering language**
**Status:** Confirmed

When the QBO connection fails, the error message uses the term "OAuth flow." No small business owner understands what OAuth means. This is engineering language in a consumer product.

---

**F-3 — Failed OAuth leaves user with no clear recovery path**
**Status:** Unconfirmed (detail)

When the OAuth connection fails, the error message appears to instruct users to run a specific command to authenticate. The exact command text was not captured and has not been confirmed. The core behavior — a failed OAuth state with no clear recovery path for a non-technical user — is consistent with the surrounding findings. Exact error text will be captured on next occurrence.

---

**F-4 — QuickBooks connector shows as connected before authentication completes**
**Status:** Confirmed

The QuickBooks connector appears connected in the Connectors directory before actual authentication has completed. The failure is only revealed when a skill attempts to use it. This silent failure would mislead any user who did not immediately run a skill after connecting.

---

**F-5 — Connector permissions screen uses accounting jargon**
**Status:** Confirmed

The connector permissions screen lists tool names such as "A/P Aging Detail" and "A/R Aging Summary." Most small business owners will not understand these terms. This screen was written for accountants, not SMB owners.

---

**F-6 — Industry Benchmarking tool is not mentioned in onboarding**
**Status:** Confirmed

The Industry Benchmarking tool is not mentioned anywhere in the onboarding flow despite being a potentially compelling value proposition for SMB owners evaluating the platform.

---

**F-7 — QuickBooks connection succeeds visually but fails silently at runtime**
**Status:** Confirmed

The QBO connection appears to succeed in the connector setup UI but fails silently when a skill actually attempts to use it. The system only surfaces the authentication problem when real work is attempted. Related to F-4.

---

**F-8 — QBO connector routes to production onboarding instead of recognizing existing Intuit account**
**Status:** Confirmed

When attempting to complete authentication, CFSB redirected to a page prompting activation of a free QuickBooks Online trial. The connector is routing to QBO production onboarding rather than recognizing an existing Intuit developer account.

---

**F-9 — QBO sandbox accounts are not supported**
**Status:** Confirmed

CFSB's QuickBooks connector does not support Intuit developer sandbox accounts. It routes all connections through QBO production onboarding, making free sandbox testing impossible through the standard connector flow.

**Impact:** Any developer, consultant, or evaluator attempting to test CFSB with a free Intuit developer sandbox account will hit an unexplained wall. A paid or trial QBO subscription is a required cost of CFSB development and evaluation. The platform communicates this nowhere.

---

**F-10 — Onboarding does not distinguish Intuit developer accounts from QBO user accounts**
**Status:** Confirmed

CFSB onboarding does not explain the difference between an Intuit developer account and a QuickBooks Online user account. A developer setting up a testing environment can easily get stuck authenticating with the wrong account type.

---

**F-11 — QBO connection threw an OAuth warning before succeeding**
**Status:** Unconfirmed

Even with a production QBO account, the connector threw an OAuth warning before completing successfully. The warning scrolled past too quickly to capture. The connection handshake appears to have a recoverable error state that would alarm a non-technical user. Exact warning text not captured.

---

**F-12 — HubSpot OAuth modal presents conflicting instructions**
**Status:** Confirmed

The HubSpot OAuth flow presents a "Grant access" modal at the critical authorization step while simultaneously displaying a message saying "Complete the sign-in steps in the new browser tab." These two instructions appear to conflict.

---

**F-13 — HubSpot OAuth modal has no actionable button**
**Status:** Confirmed

The "Grant access to HubSpot" modal contains no actionable button and no explicit instruction to return to the browser tab where OAuth is completing. The connection succeeded only because the user correctly inferred the right action. A less technical user would likely abandon at this step.

---

## Platform Behavior: Slash Command Invocation

See also Skill Run: /monday-brief F-1 and F-2, and Skill Run: /call-list F-1.

---

**F-1 — Slash command failures produce no explanation**
**Status:** Confirmed

Slash command invocation fails intermittently across multiple sessions and multiple skills. When it fails, the user receives an "Unknown skill" error with no explanation and no fallback guidance. SMB owners who learn slash commands from documentation will hit silent failures with no path forward.

---

## Platform Behavior: Cowork UX

---

**F-1 — Permission prompt text was not captured**
**Status:** Confirmed (detail unconfirmed)

Claude in Cowork requested explicit user permission for two actions before completing an invoice query. The exact text of those prompts was not captured. Capturing exact prompt text is a methodology requirement for all future runs.

---

**F-2 — Transient status messages are unreadable**
**Status:** Confirmed

Cowork provides no way to pause, replay, or review transient status messages that appear during skill execution. These messages scroll past at speed and are unreadable under normal operating conditions.

---

## Platform Behavior: Environment Constraints

---

**F-1 — MacInCloud Managed Plan is incompatible with Cowork**
**Status:** Confirmed

Claude Cowork's workspace initialization requires admin-level access to Apple's Virtualization framework. MacInCloud's Managed Server plan runs with standard user permissions and cannot satisfy this requirement. US-based latency testing via MacInCloud on the standard plan is not feasible.

---

## External Observations

Findings in this section originate from external sources — community posts, published articles, or claims made by other CFSB developers. They are included because they affect how the platform is understood or evaluated, and in some cases because they are being actively investigated.

---

**F-1 — Forbes article significantly understates platform capability**
**Status:** Confirmed

A May 2026 Forbes article on CFSB describes the platform as offering "15 workflow actions." The Small Business plugin contains 30 skills across five categories. Inaccurate capability descriptions in mainstream coverage create mismatched expectations for evaluators arriving through press coverage.

---

## Open Investigations

The following questions are raised by findings above and are under active investigation.

**OI-1** (from Skill Run: /monday-brief F-8, F-9) — What is the full scope of the "Always allow" permission grant? Does it persist across sessions? Does it apply to the connector globally or only the specific tool call type?

**OI-2** (from Skill Run: /monday-brief F-12) — Is the Google Drive save failure in /monday-brief a connector issue, an incomplete feature, or a configuration dependency? Is there any condition under which it works?

**OI-3** (from Skill Run: /call-list F-1) — What is the clean /call-list runtime with all connectors pre-authorized and VPN off? Current timing data is from an uncontrolled run. TT-5 will establish the baseline.

**OI-4** (from Skill Run: /call-list F-3) — Does adding a CLAUDE.md business context file to Cowork improve /call-list contact selection quality and make the selection logic more transparent?

**OI-5** (CT) — Does CFSB run the reasoning agent in a sandbox truly separated from connector actions, as claimed by at least one community developer? What evidence supports or contradicts this?

**OI-6** (CT) — Can permission prompts be permanently suppressed per connector? If so, what is the correct procedure and what scope does the suppression cover?

---

## Recommended Practitioner Checklist

Based on findings to date, the following steps are recommended before running any CFSB skill in a client or demo context.

1. **Use a production QBO account.** Sandbox accounts are not supported. Plan for this cost before committing to a client engagement.
2. **Pre-authorize all connectors** before invoking any skill. Do not add connectors mid-run. Expect a 10x runtime penalty if you don't.
3. **Use plain English invocation.** Do not rely on slash commands — they fail intermittently with no explanation.
4. **Stay present during execution.** Permission prompts stall a run silently. There is no alert. Plan to watch the screen.
5. **Do not expect Google Drive saves.** /monday-brief does not currently attempt the save even when Google Drive is connected and authorized.
6. **Seed CRM data deliberately** before running /call-list if you need to validate output quality. There is no other way to verify the ranking is correct.
7. **Time every run.** Runtime varies significantly based on connector state. Establish a baseline before quoting turnaround to a client.

---

## About This Log

This findings log is maintained as part of an independent consulting practice focused on deploying CFSB for small businesses. The goal is to build in public, document honestly, and share findings that help SMB owners make informed decisions.

All findings are based on direct observation during documented skill runs. Where a finding is uncertain, it is marked Unconfirmed. Where a finding was disproved, it is marked Retracted and the correction is explained. Nothing here is estimated or fabricated.

This log is updated at the end of each documented session. New sections are added at the top of the Findings by Category area so the most recent work appears first.

If you are a developer working on the CFSB platform and have found this file: the findings above are offered as field notes from real-world testing, in the hope that they are useful.

---

*Maintained by an independent CFSB developer and consultant. Not affiliated with Anthropic.*
