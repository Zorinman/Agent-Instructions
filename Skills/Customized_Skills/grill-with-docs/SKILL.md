---
name: grill-with-docs
description: A relentless interview to sharpen a plan or design, which creates an ADR and glossary updates as decisions emerge.
disable-model-invocation: true
---

Run a `/grilling` session, using the `/domain-modeling` skill.

When the user confirms a shared understanding and the session reaches an ADR-worthy architectural decision, create the ADR through `/domain-modeling`. Write it to the feature's local document group:

- Reuse the matching feature number and name when the feature already has a Spec or Ticker group.
- Otherwise allocate the next feature number by scanning the project's `workflow/docs/adr`, `workflow/docs/specs`, `workflow/docs/tickers`, and `workflow/docs/archives` paths.
- Write the ADR as `workflow/docs/adr/<NNNN>_adr_<feature-name>.md`.

This skill creates the ADR and glossary updates only. `/To Spec` creates the matching Spec and `/To Tickets` creates the matching multi-file Ticker group. Do not begin implementation until the user directs it.
