# Open questions

This folder holds the questions that research in economy-information-rand has not answered yet. Each question is one Markdown file. Together, the files are the repository's list of open questions. The list is kept up to date.

## What it is for

Research finds gaps as well as answers. A record may be missing, two sources may disagree, or a question may need a test, a lawyer or a decision to answer it. Writing the gap down keeps it from being lost. Then the next researcher, a lawyer or NTARI's board can work on it. A clear gap is a useful result.

The Janus Facing Architecture (JFA) asks every JFA project to keep a list like this one, and to keep it up to date. See the [concept triage](https://github.com/NTARI-RAND/Janus/blob/main/jfa-concept-triage-2026-08-24.md) and [Janus's own list](https://github.com/NTARI-RAND/Janus/blob/main/OPEN-QUESTIONS.md). A question leaves the list only when it is resolved, or when the people who live with the outcome make an explicit decision about it. If this list is out of date, the repository no longer tells the truth about what is known. Check this folder whenever you send a pull request.

A question about the JFA standard itself goes in [Janus's list](https://github.com/NTARI-RAND/Janus/blob/main/OPEN-QUESTIONS.md). A question about one repository's code goes in that repository's own list, if it has one. Other questions about research on this layer go here.

## Adding a question

1. Check that the question is not already listed. Look in this folder, in the open-questions list of the repository it is about (if it has one), and in [Janus's list](https://github.com/NTARI-RAND/Janus/blob/main/OPEN-QUESTIONS.md). If it is in this folder, add what you found to that file, under **What was checked**, with your name and the date. If it is listed somewhere else, add what you found there instead.
2. Create a new file in this folder. Start the file name with the tier; if the question spans tiers, use the lowest one. Then add a few words from the question. Use only lowercase letters a to z, digits and hyphens. For example: `tier-1-exchange-protocol.md`.
3. Copy the template below into the file, and fill in every heading.
4. Send the file in a pull request with a DCO sign-off. Use the same steps as for other research (see [How we research](../README.md#how-we-research)), but put the file in this folder, not in a tier folder.

Research often ends with questions it could not answer. List each one here as an open question, in its own pull request, so it is on the list while your research is still under review. Link to it from your research.

Write in plain English. This repository is public, and deleting a file later does not remove it from the history. Never include passwords, API keys or other secrets. Apart from your own name, never include anything that identifies a person: a member's name, username or account ID, the IP address or location of someone's machine, home addresses, personal phone numbers or emails, handwritten signatures, dates of birth, personal ID numbers (such as passport numbers), bank details or donor names. Use made-up test data instead.

## Template

```markdown
# The question, in one sentence

**Status:** open

**Asked by:** your name, YYYY-MM-DD

**Tier:** 1 (Platform protocols), 2 (Running on the substrate) or 3 (Platform frontends). If it spans tiers, name each one.

## Why it matters

One or two sentences: what depends on the answer.

## What was checked

Each source you checked and what it showed, with its link. For a file on
GitHub, link to a fixed commit, not to main, so the link still shows what
you saw. Also say what you looked for and did not find.

## Constraints inherited

The rules any answer must respect, such as bylaws sections, governance
registry entries, the JFA's Lines That Cannot Be Crossed, or laws.
Write "None known" if there are none.

## Who might answer

For example: the people who maintain the platform it is about (such as Agrinet or fruitful), a lawyer, #rand-economy-information on Slack, or NTARI's board.

## Resolution

Leave this empty while the question is open.
```

## When a question is answered

Update its file in a pull request. Set **Status** to `resolved (YYYY-MM-DD)`, and write the answer under **Resolution**, with its evidence and links. Keep the file. The list keeps its history.

A question can also leave the list without an answer. This needs an explicit decision by the people who live with the outcome, such as NTARI's board or a vote in #federation-economy-information. The person who asked the question cannot close it alone. Set **Status** to `closed (YYYY-MM-DD)`. Under **Resolution**, write who decided, what they decided and why, with a link to where the decision is recorded, such as a governance registry entry.

## Opening a question again

No answer is closed for good. Under bylaws §2.3, any member may bring a decided matter back. To reopen a question, send a pull request that sets **Status** back to `open`. Move the old **Resolution** text under **What was checked**, with its date, so the history stays. Then add the new facts, with your name and the date.
