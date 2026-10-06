---
name: app-store-expedite-review
description: Request or preview expedited App Store review for a submitted app using Codex's signed-in in-app browser. Use when the user asks to expedite Apple review; do not submit an expedited request as part of an ordinary release.
---

# Expedited App Store review

Use `mcp__cua_repl` and Codex's in-app browser for Apple's form:
https://developer.apple.com/contact/app-store/?topic=expedite

Reuse an existing tab at that URL, otherwise open one in the in-app browser. Follow the browser tool's current documentation. Do not launch Chrome, export cookies, or require an App Store Connect API key. If Apple requires login, let the user sign in in this same browser and continue afterward. Wait for the form or login UI before concluding that the session expired.

Determine the requested app and developer team from the conversation, repository's Fastlane Appfile, or the visible app picker. A submitted app is required; check its review status through available App Store Connect access when practical. Never submit a new build or cancel a submission just to expedite it. Ask for the app or team only when it remains ambiguous. Do not request a reason or reason file: the inspected form has no reason field. If Apple adds one, follow the visible form and ask for genuine missing details.

## Form workflow

Inspect the live UI; these are observed selectors, not a substitute for verifying the current form:

- Topic: `#form_topic`, label “request an expedited app review”. The URL normally preselects it.
- Organization: `#contact_team_name`. Choose the exact intended team.
- App name: `#expedite_app_name`. Type the app's name, then select the exact matching entry inside `#list-of-apps-container`. Typing alone does not select the app.
- Platform: `#expedite_app_platform`, e.g. “iOS”.
- Send: `#contactForm_saveButton`.

Team/app loading is asynchronous. Wait for populated fields rather than treating a transient “no apps” message as definitive. Reinspect after selecting the team and app. Confirm the selected app, platform, and enabled Send button.

For a preview or test, fill the form and stop before Send. Save and show a screenshot, and state that nothing was sent. For an explicit request to expedite the named app, click Send once and verify Apple's acknowledgement in the resulting UI. The request to build or test this skill does not authorize sending a live request.

After Send, report success only when Apple visibly acknowledges receipt. If acknowledgement is ambiguous or the browser times out, do not click Send again; inspect the resulting page and explain that receipt is unconfirmed. Do not imply that requesting expedition guarantees Apple will grant it.

Keep the in-app browser session intact: do not sign out or close it as cleanup. Save any screenshots outside tracked repositories and never include authentication data in the skill or source control. Mark a preview/confirmation tab as a deliverable or handoff as appropriate.

Apple's requirements: https://developer.apple.com/help/app-review/after-submitting-for-review/request-expedited-review/
