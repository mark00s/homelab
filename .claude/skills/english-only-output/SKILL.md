---
name: english-only-output
description: Forces all content that lands in this repository — code, comments, docstrings, commit messages, PR titles/descriptions, README and other docs, resource and variable names — to be written in English, regardless of what language the user writes in during the conversation. Always consult this before writing or editing any file, commit message, or PR text in this repo. Does not affect conversational replies to the user in chat — only what actually gets written into the repo.
---

# English-Only Repo Output

This repo is public. Everything that lands in it stays in English, even though the conversation about it may happen in any other language.

## Applies to

- Code: variable, function, class, resource, and file names
- Code comments and docstrings
- Commit messages and PR titles/descriptions
- README, architecture docs, changelogs, inline config comments
- Error messages or log strings authored as part of the code (not output produced by third-party tools/libraries, which isn't yours to translate)

## Does not apply to

- Conversational replies to the user in chat — keep responding in whatever language they use there
- Content the user explicitly asks to be in given language (e.g. a Polish-language section of the README aimed at local readers) — only that explicitly-flagged part stays in given language

## How to apply it

- When the user describes what they want in non-english language, translate the _intent_ into English when writing it to a file — don't transliterate, and don't leave a non-english name or comment in "temporarily" to fix later
- Watch for accidental leakage: a word copy-pasted from non-english conversation into a comment, a commit message drafted in the same language as the chat instruction that requested it, a TODO left in non-english language
- A resource name that's naturally a proper noun with no English equivalent (e.g. a real device model like `RB5009UG+S+IN`) stays as-is — this rule is about language, not renaming products or brands
- When in doubt about a specific string, default to English
