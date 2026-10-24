---
name: browse
description: Open a Tabductor cloud browser at a URL and report what is there, keeping the browser open for follow-up questions and actions.
argument-hint: "<url> [what to look for]"
disable-model-invocation: true
---

Open a Tabductor cloud browser and look at: $ARGUMENTS

Follow the `tabductor` skill. Concretely:

1. If a browser from this conversation is still live (check with `browser_get`), reuse it; otherwise
   call `browser_start` and wait until `ready` is true.
2. `browser_navigate` to the URL (the first argument). If there is no URL, ask for one.
3. Read the page with `browser_evaluate` (`document.title`, `location.href`, and
   `document.body.innerText.slice(0, 20000)`), and take a `browser_screenshot` if layout matters.
4. Answer the question in the remaining arguments, or summarize the page if there is none. Quote the
   final URL and title. Mention the `liveViewUrl` once so the user can watch.
5. Keep the browser open for follow-ups. Say that it stops by itself after the idle timeout, and
   offer `browser_stop` when the user is done.
