# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** Muhammad Salman Manzar
- **Reference:** MATACS-TH-2026-02 (Associate Software Developer)
- **Time actually spent:** Approximately 3 hours

## Files

- [WRITEUP.md](WRITEUP.md): answers to the five questions, with the result on the sample set
- [diagram/process-flow.png](diagram/process-flow.png): flow from upload to ranked output (blue = rules/code, orange = AI, green = human checkpoint)
- [PROMPTS.md](PROMPTS.md): the injection-check and extraction prompts, exactly as sent
- [ranking.json](ranking.json): expected ranking of proposals A-E

No prototype is included. Proposals are referred to by letter (A-E) only.

## How I used AI tools

I used Claude to help me think through the design and to draft the files in this repository (the flow diagram, write-up, prompts and ranking). I then read everything, checked the scoring numbers by hand, and edited the files so I could explain every choice myself.

The AI first scored D's missing completion time as 0 and still ranked D third. I disagreed because the project must finish within 16 weeks and D gave no commitment, so I changed D to flagged, asked for clarification instead of guessing, and updated the ranking, write-up and diagram to match.
