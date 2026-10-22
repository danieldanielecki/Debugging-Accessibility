# Debugging (Web) Accessibility

## What this talk is about

A 30-minute debugging session for web developers who already ship UI. The screen can look finished while the accessibility tree, the keyboard, or a screen reader is stuck: a control with no name, a dialog that leaks focus into the page behind it, or a "Saved" message that never gets announced. Chrome's accessibility pane and the keyboard are the debugger. A screen reader confirms the announcement when the room audio cooperates. The live pages are Tap2Play CodeSandboxes from the open-source [Awesome A11Y](https://github.com/Ditectrev/Awesome-A11Y-Book-Course-Web-Accessibility-A11Y) course (Daniel Danielecki, Ditectrev). The personal talk repo [Debugging-Accessibility](https://github.com/danieldanielecki/Debugging-Accessibility) is only a README title as of 21 Oct 2026, so the deck uses the course sandboxes that already run.

## Minute budget

- 0:00–2:00 — Title and the claim
- 2:00–5:00 — Where the bug shows up
- 5:00–8:00 — What a scanner will and will not tell you
- 8:00–14:00 — Demo: dark mode switch (name, role, value)
- 14:00–20:00 — Demo: delete-account dialog (focus)
- 20:00–26:00 — Demo: status and alert (announcements)
- 26:00–30:00 — The bug report, the repos, close
- If a demo runs long, shorten demo 3. The form-error sandbox is the spare, not a fourth demo.

## Slides

### Debugging (Web) Accessibility

- Daniel Danielecki
- 30 minutes
- Three bugs that look fine on screen
- Tools you already have: keyboard, Chrome accessibility pane, one screen reader

### The page looks finished

- The button is styled, the dialog opens, "Settings saved" appears
- The task still fails for someone reaching it by keyboard, or hearing it
- We debug that gap

### Where the bug shows up

- Accessibility tree: name, role, value, state
- Keyboard: Tab, Shift+Tab, Enter, Space, Escape, arrows
- Screen reader: what is announced, and when
- First stop is Chrome DevTools → Accessibility, with the node you clicked selected

### Scanners are a first pass

- axe DevTools, WAVE, and Lighthouse catch a lot of missing names, contrast failures, and bad roles
- From the course section Testing and Evaluation: they miss focus-trap logic, and they cannot tell you whether a live region was actually heard
- Read the specific audit. A Lighthouse number is a hint
- Reproduce the task yourself before you file it

### Before the demos

- Chrome, accessibility pane open, full page zoom at 100%
- One screen reader if the room can hear it: NVDA on Windows, or VoiceOver on macOS (VoiceOver on, Command-F5)
- Editor tab and preview tab for each sandbox, already loaded
- The published sandboxes show the working pattern. The demo is removing one real attribute, watching it fail, and putting it back

### Demo 1 — Dark mode switch (6 minutes)

- Preview: [cykvdg.csb.app](https://cykvdg.csb.app/)
- Editor: [CodeSandbox 055, name / role / value](https://codesandbox.io/p/sandbox/055-exposing-name-role-value-to-assistive-tech-custom-control-requirements-cykvdg)
- Course section: Name, role, value (WCAG 4.1.2), custom controls
- Two controls do the same job. The heading in the page says the native one needs no script
- Native: `<input type="checkbox" role="switch">` with `<label for="dark-mode-native">Dark mode</label>`
- Custom: a `<div id="dark-mode-custom">` with `role="switch"`, `tabindex="0"`, `aria-checked="false"`, `aria-labelledby="dark-mode-custom-label"`, plus click and keydown listeners
- Accessibility pane on the native switch: name "Dark mode", role switch, checked state
- Same pane on the custom div: role switch, checked from `aria-checked`, name from the label id
- Break it in the editor, one piece at a time, and refresh the preview:
  - Remove `tabindex="0"`: Tab skips it
  - Remove `role="switch"`: the tree no longer exposes a switch
  - Remove the keydown handler for Space and Enter: it can take focus and still does nothing from the keyboard
  - Remove `aria-labelledby`: the role survives, the name does not
- Put the attributes back. Tree and keyboard agree again
- Say this out loud: the native checkbox switch did not need that script

### What you file after demo 1

- User impact: keyboard and screen reader users cannot toggle the setting
- Map: WCAG 4.1.2 Name, Role, Value, and 2.1.1 Keyboard if it cannot be operated
- Evidence: the accessibility pane, plus the attribute you removed

### Demo 2 — Delete account dialog (6 minutes)

- Preview: [jgcpws.csb.app](https://jgcpws.csb.app/)
- Editor: [CodeSandbox 031, modal tab cycle and inert](https://codesandbox.io/p/sandbox/031-tab-cycle-within-modal-first-and-last-focusable-inert-jgcpws)
- Course section: Focus trap in modals (WCAG 2.1.2 No Keyboard Trap)
- On the page: button "Delete account", dialog titled "Confirm account deletion", buttons Cancel and Delete
- Open path in the example: save nothing yet, set `appRoot.inert = true`, `dialog.show()`, move focus to the heading (`tabindex="-1"`)
- Show the working case: Tab and Shift+Tab stay inside the dialog. The page behind it is inert. Escape and the close button leave
- Break it: in the open handler, leave `appRoot.inert` false. Open the dialog and Tab. Focus walks into the page behind the dialog
- A focus trap inside an open dialog is the requirement. Focus that escapes into the background is the bug. A trap with no Escape and no reachable close button is the other bug
- If someone asks where focus goes on close: [CodeSandbox 032](https://jq8s7l.csb.app/) ([editor](https://codesandbox.io/p/sandbox/032-returning-focus-on-close-tabindex-0-vs-tabindex-1-jq8s7l)) returns focus to the opener, and uses `tabindex="-1"` on the heading you focus when the dialog opens, so it does not join the tab order

### Demo 3 — A message nobody hears (6 minutes)

- Preview: [kl8pd2.csb.app](https://kl8pd2.csb.app/)
- Editor: [CodeSandbox 056, aria-live, status, and alert](https://codesandbox.io/p/sandbox/056-aria-live-role-status-and-role-alert-kl8pd2)
- Course section: Status messages (WCAG 4.1.3)
- The page updates text without moving focus. Examples in the sandbox include "15 results loaded", "Settings saved.", "Payment failed. Card was declined.", and "Submission failed. Please fix highlighted fields."
- `role="status"` is the polite channel. `role="alert"` interrupts
- Accessibility pane on those nodes shows the role
- Break it: remove `role="status"` or `role="alert"`, click the same button. The text still paints. The live region is gone, so a screen reader has nothing new to announce
- With NVDA or VoiceOver on: status waits, alert interrupts. Course testing notes: browse mode for headings and landmarks, focus mode when you are in a form
- Spare, if this one misfires: [CodeSandbox 053, form errors](https://kzgxcj.csb.app/) ([editor](https://codesandbox.io/p/sandbox/053-aria-describedby-aria-invalid-error-summary-kzgxcj)). Invalid email and postal code, `aria-invalid="true"`, `aria-describedby` pointing at hint and error, an error summary with links, focus moved to that summary. Remove `aria-invalid` and the describedby ids: the red text can stay on screen while the field no longer exposes the error

### Optional 90 seconds, only if you are ahead

- [CodeSandbox 030, buttons and links](https://4zxpnc.csb.app/) ([editor](https://codesandbox.io/p/sandbox/030-all-functionality-via-keyboard-space-vs-enter-for-buttons-and-links-4zxpnc))
- Native `<button>` activates on Space and Enter. A link activates on Enter
- The custom `<span role="button" tabindex="0">` only does that because of a keydown listener. Delete the listener and the click still works with a mouse

### The report you file on Monday

- Who is blocked, and whether a workaround exists
- Steps: keyboard or screen reader, expected, actual
- One WCAG criterion you can defend: 2.1.1, 2.1.2, 4.1.2, or 4.1.3
- DOM snippet and the accessibility pane
- Severity from task impact (login, pay, submit, destroy), not from the order axe printed

### Where the code lives

- Course, with the 61 sandboxes: [Ditectrev/Awesome-A11Y-Book-Course-Web-Accessibility-A11Y](https://github.com/Ditectrev/Awesome-A11Y-Book-Course-Web-Accessibility-A11Y)
- Personal talk repo, stub today (README is the title only): [danieldanielecki/Debugging-Accessibility](https://github.com/danieldanielecki/Debugging-Accessibility)
- Same home as the other talk repos under [github.com/danieldanielecki](https://github.com/danieldanielecki), once the before-and-after snippets for these three bugs are published there

### Close

- The pixels can be right while the tree is wrong
- Debug name, role, and value, then the keyboard, then the announcement
- Bookmark the three previews: [switch](https://cykvdg.csb.app/), [dialog](https://jgcpws.csb.app/), [status and alert](https://kl8pd2.csb.app/)
