# Debugging (Web) Accessibility

## What this talk is about

A 30-minute debugging session for web developers who already ship UI. The screen can look finished while the accessibility tree, the keyboard, or a screen reader is stuck: a control with no name, a dialog that leaks focus into the page behind it, or a "Saved" message that never gets announced. The debugger is the keyboard, plus the Chrome panels that read that tree: the Accessibility pane, the full-page accessibility tree, the Rendering tab, Lighthouse accessibility, and the Issues panel. A screen reader confirms the announcement when the room audio cooperates. A browser agent reads the same tree, so a missing role or name makes it miss the control too. The live pages are Tap2Play CodeSandboxes from the open-source [Awesome A11Y](https://github.com/Ditectrev/Awesome-A11Y-Book-Course-Web-Accessibility-A11Y) course (Daniel Danielecki, Ditectrev). The personal talk repo [Debugging-Accessibility](https://github.com/danieldanielecki/Debugging-Accessibility) is only a README title as of 21 Oct 2026, so the deck uses the course sandboxes that already run.

## Minute budget

- 0:00–1:30 — Title and the claim
- 1:30–5:00 — Where the bug shows up, and the DevTools you open
- 5:00–6:30 — Scanners, shortened, then demo setup
- 6:30–9:00 — How an agent reads the same tree
- 9:00–15:00 — Demo: dark mode switch (name, role, value)
- 15:00–21:00 — Demo: delete-account dialog (focus)
- 21:00–27:00 — Demo: status and alert (announcements)
- 27:00–30:00 — The bug report, the repos, close
- The button sandbox is off the clock. If a demo runs long, skip that sentence. Do not shorten the three demos. The form-error sandbox is the spare, not a fourth demo.

## Slides

### Debugging (Web) Accessibility

- Daniel Danielecki
- 30 minutes
- Three bugs that look fine on screen
- Tools you already have: keyboard, Chrome's accessibility tree, one screen reader

### The page looks finished

- The button is styled, the dialog opens, "Settings saved" appears
- The task still fails for someone reaching it by keyboard, or hearing it
- We debug that gap

### Where the bug shows up

- Accessibility tree: name, role, value, state
- Keyboard: Tab, Shift+Tab, Enter, Space, Escape, arrows
- Screen reader: what is announced, and when
- Chrome's accessibility reference draws the same line: DevTools checks whether the page is marked up for assistive tech. Walking it with the keyboard is still on you
- Open the Accessibility pane on the node you clicked: Elements panel, select that node, Accessibility tab. The tab can sit behind the More Tabs button. Computed Properties is where you read the name, the role, and the other computed properties. ARIA attributes are in the same pane
- Full-page tree: on that tab, toggle Show accessibility tree. It replaces the DOM tree in Elements. Select a node and Computed Properties stays in the pane. A generic `div` or `span` is announced in that tree as GenericContainer
- Rendering tab, in the drawer, when the bug is visual preference rather than a missing role. Open it from the Command Menu entry Show Rendering, or open the drawer and choose More Tools, then Rendering. The controls this talk uses are Emulate vision deficiencies, Emulate CSS media feature `prefers-reduced-motion` (`reduce`), and Emulate CSS media feature `forced-colors` (`active`)
- Leave Rendering closed on the three live breaks. None of those bugs is a motion, contrast, or forced-colors bug

### Scanners are a first pass

- Lighthouse panel, Accessibility category: missing names, contrast, bad roles. axe DevTools and WAVE overlap it. Run Lighthouse only if you need the list. Do not spend this slide on a report
- From the course section Testing and Evaluation: a scanner misses focus-trap logic, and it cannot tell you whether a live region was heard. Read the specific audit. The score is a hint
- Issues panel: the Open Issues button next to Settings, or Issues from the More tools menu. It groups problems the browser already detected. The accessibility item in the current docs is low-contrast, and that reporting is still a preview (Settings, Experiments, enable automatic contrast issue reporting via the Issues panel). It will not flag a missing name or a silent status message

### Before the demos

- Chrome, Accessibility tab open, Show accessibility tree on, full page zoom at 100%
- Rendering, Lighthouse, and Issues stay closed until a question is actually about contrast, motion, or forced colors
- One screen reader if the room can hear it: NVDA on Windows, or VoiceOver on macOS (VoiceOver on, Command-F5)
- Editor tab and preview tab for each sandbox, already loaded
- The published sandboxes show the working pattern. The demo is removing one real attribute, watching it fail, and putting it back

### How an agent reads the same tree

- Practical case for this room: a browser agent, and Playwright, locate controls from the accessibility tree. Accessible name, role, and state. Not from the stylesheet
- Playwright's concrete API: `getByRole`, and the aria snapshot (`locator.ariaSnapshot()`, or `toMatchAriaSnapshot`). The snapshot is YAML of that tree. A node is `role "name"`, plus state such as `[checked]` when that state is on. Playwright's docs point you at Chrome's Accessibility tab to inspect the same tree
- `getByRole('switch', { name: 'Dark mode' })` matches the accessible name. The name match is a substring and case-insensitive unless you pass `exact`. It does not match the pixels
- There is a separate `mode: "ai"` snapshot. It is still this tree, with element refs added for an agent. It is not a screenshot
- A broken tree makes that agent miss the control the same way a screen reader does
- A screenshot-only agent has the painted page and still misses focus order and whether a message was announced. That is a different miss from a tree-based agent
- WCAG 2.5.3 Label in Name is already in the course. The accessible name has to contain the visible label. A visible "Dark mode" whose accessible name is some other string trips a speech user and an agent matching that visible text
- The next demo is the switch. Say what the snapshot contains before you delete anything

### Demo 1 — Dark mode switch (6 minutes)

- Preview: [cykvdg.csb.app](https://cykvdg.csb.app/)
- Editor: [CodeSandbox 055, name / role / value](https://codesandbox.io/p/sandbox/055-exposing-name-role-value-to-assistive-tech-custom-control-requirements-cykvdg)
- Course section: Name, role, value (WCAG 4.1.2), custom controls
- Open the Accessibility pane on the control, with the full-page tree toggled on. Leave Rendering, Lighthouse, and Issues closed
- Two controls do the same job. The heading in the page says the native one needs no script
- Native: `<input type="checkbox" role="switch">` with `<label for="dark-mode-native">Dark mode</label>`
- Custom: a `<div id="dark-mode-custom">` with `role="switch"`, `tabindex="0"`, `aria-checked="false"`, `aria-labelledby="dark-mode-custom-label"`, plus click and keydown listeners
- Accessibility pane on the native switch: name "Dark mode", role switch, checked state
- Same pane on the custom div: role switch, checked from `aria-checked`, name from the label id
- That custom switch, while off, is what a default aria snapshot contains: a `switch` named "Dark mode", and no `[checked]` flag. Playwright prints `[checked]` when the checked state is true, and omits it when false. Turn the switch on and the same line gains `[checked]`. `getByRole('switch', { name: 'Dark mode' })` finds it either way
- Break it in the editor, one piece at a time, and refresh the preview:
  - Remove `tabindex="0"`: Tab skips it. The role and the name are still in the tree, so an agent matching the switch can still hit it
  - Remove the keydown handler for Space and Enter: it can take focus and still does nothing from the keyboard. A click on the role can still succeed
  - Remove `role="switch"`: the pane no longer exposes a switch. Chrome's full-page tree shows a generic container, not a switch. The default aria snapshot drops that switch, and `getByRole('switch', { name: 'Dark mode' })` misses it. The words can still be painted on screen
  - Remove `aria-labelledby`: the role survives, the name does not. The snapshot still has a switch, without the name "Dark mode". The same `getByRole` call misses it. A screen reader no longer hears "Dark mode"
- Put the attributes back. Tree, keyboard, and the snapshot agree again
- Say this out loud: the native checkbox switch did not need that script. Its visible label and its accessible name are the same string, which is also what 2.5.3 is protecting

### What you file after demo 1

- User impact: keyboard and screen reader users cannot toggle the setting. An agent matching role and name cannot toggle it either, once the role or the name is gone
- Map: WCAG 4.1.2 Name, Role, Value, and 2.1.1 Keyboard if it cannot be operated. 2.5.3 if the visible label and the accessible name disagree
- Evidence: the accessibility pane, the attribute you removed, and the snapshot line if you captured one

### Demo 2 — Delete account dialog (6 minutes)

- Preview: [jgcpws.csb.app](https://jgcpws.csb.app/)
- Editor: [CodeSandbox 031, modal tab cycle and inert](https://codesandbox.io/p/sandbox/031-tab-cycle-within-modal-first-and-last-focusable-inert-jgcpws)
- Course section: Focus trap in modals (WCAG 2.1.2 No Keyboard Trap)
- Open the Accessibility pane on the dialog, and the full-page tree. Leave Rendering, Lighthouse, and Issues closed. None of them catch a focus trap
- On the page: button "Delete account", dialog titled "Confirm account deletion", buttons Cancel and Delete
- Open path in the example: save nothing yet, set `appRoot.inert = true`, `dialog.show()`, move focus to the heading (`tabindex="-1"`)
- Pane on the dialog: read its role and its name. The full-page tree is how you see that dialog in the page, not how you prove the trap
- Show the working case: Tab and Shift+Tab stay inside the dialog. The page behind it is inert. Escape and the close button leave
- Break it: in the open handler, leave `appRoot.inert` false. Open the dialog and Tab. Focus walks into the page behind the dialog. Look at the full-page tree again: the background is still exposed
- A focus trap inside an open dialog is the requirement. Focus that escapes into the background is the bug. A trap with no Escape and no reachable close button is the other bug
- An agent that only looks at a screenshot of the open dialog cannot see that Tab has left it. The keyboard is the check
- If someone asks where focus goes on close: [CodeSandbox 032](https://jq8s7l.csb.app/) ([editor](https://codesandbox.io/p/sandbox/032-returning-focus-on-close-tabindex-0-vs-tabindex-1-jq8s7l)) returns focus to the opener, and uses `tabindex="-1"` on the heading you focus when the dialog opens, so it does not join the tab order

### Demo 3 — A message nobody hears (6 minutes)

- Preview: [kl8pd2.csb.app](https://kl8pd2.csb.app/)
- Editor: [CodeSandbox 056, aria-live, status, and alert](https://codesandbox.io/p/sandbox/056-aria-live-role-status-and-role-alert-kl8pd2)
- Course section: Status messages (WCAG 4.1.3)
- Open the Accessibility pane on the message node. After you remove the role, glance at the full-page tree. Leave Rendering, Lighthouse, and Issues closed. A scanner cannot tell you the message was heard
- The page updates text without moving focus. Examples in the sandbox include "15 results loaded", "Settings saved.", "Payment failed. Card was declined.", and "Submission failed. Please fix highlighted fields."
- `role="status"` is the polite channel. `role="alert"` interrupts
- Accessibility pane on those nodes shows the role. When the accessible name is the text inside the node, an aria snapshot of it is that `status` or `alert` with that name
- Break it: remove `role="status"` or `role="alert"`, click the same button. The text still paints. The live region is gone, so a screen reader has nothing new to announce. The full-page tree no longer has that status or alert role
- A screenshot-only agent sees the new sentence and still does not know whether it was announced, or where focus is. A tree-based agent sees the role disappear
- With NVDA or VoiceOver on: status waits, alert interrupts. Course testing notes: browse mode for headings and landmarks, focus mode when you are in a form
- Spare, if this one misfires: [CodeSandbox 053, form errors](https://kzgxcj.csb.app/) ([editor](https://codesandbox.io/p/sandbox/053-aria-describedby-aria-invalid-error-summary-kzgxcj)). Invalid email and postal code, `aria-invalid="true"`, `aria-describedby` pointing at hint and error, an error summary with links, focus moved to that summary. Accessibility pane: the invalid state and the description. Remove `aria-invalid` and the describedby ids: the red text can stay on screen while the field no longer exposes the error

### If you are ahead

- One sentence, then stop. [CodeSandbox 030, buttons and links](https://4zxpnc.csb.app/) ([editor](https://codesandbox.io/p/sandbox/030-all-functionality-via-keyboard-space-vs-enter-for-buttons-and-links-4zxpnc)): a native button activates on Space and Enter, a link on Enter, and the custom `<span role="button" tabindex="0">` only does that because of a keydown listener
- Not a timed block. Skip it the moment a demo ran long

### The report you file on Monday

- Who is blocked, and whether a workaround exists. Include an agent that has to complete the same task, if that task is one you automate
- Steps: keyboard or screen reader, expected, actual
- One WCAG criterion you can defend: 2.1.1, 2.1.2, 2.5.3, 4.1.2, or 4.1.3
- DOM snippet, the accessibility pane, and the aria snapshot line when the bug is name or role
- Severity from task impact (login, pay, submit, destroy), not from the order axe printed

### Where the code lives

- Course, with the 61 sandboxes: [Ditectrev/Awesome-A11Y-Book-Course-Web-Accessibility-A11Y](https://github.com/Ditectrev/Awesome-A11Y-Book-Course-Web-Accessibility-A11Y)
- Personal talk repo, stub today (README is the title only): [danieldanielecki/Debugging-Accessibility](https://github.com/danieldanielecki/Debugging-Accessibility)
- Same home as the other talk repos under [github.com/danieldanielecki](https://github.com/danieldanielecki), once the before-and-after snippets for these three bugs are published there

### Close

- The pixels can be right while the tree is wrong
- Debug name, role, and value, then the keyboard, then the announcement
- An agent that reads the tree fails with you. A screenshot of the finished UI will not show it
- Bookmark the three previews: [switch](https://cykvdg.csb.app/), [dialog](https://jgcpws.csb.app/), [status and alert](https://kl8pd2.csb.app/)
