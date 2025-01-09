# Architecture Decision Records

The ADR is a lightweight record format intended to capture individual architecturally important decisions. They are meant to be easy to write - 10 minutes or less. They should be stored in the codebase they affect, go through peer review, and have a commit history.

This simple format, which is described below, has a surprising number of functions:

* **Decision making process**: by going through peer review, it includes the entire team and gives all perspectives a chance to be heard. There is a clear decision making process with a clear lifecycle - once an ADR meets whatever approval criteria the team chooses, it is merged and the decision is done. If new information comes to light that causes the team to reconsider the decision, then that is simply a new ADR.
* **Institutional knowledge and transparency**: Not everyone will comment on every ADR, but the transparency of the mechanism should serve to keep everyone informed and encode tribal knowledge into writing. This also builds resilience - there should ideally never be decision making that is blocked by someone being sick or on vacation. The team should always be able to make significant decisions.
* **Distribute design authority**: As a team becomes familiar and comfortable with the ADR mechanism, every team member has an equal tool to bring design decisions to the team. This encourages autonomy, accountability, and ownership.
* **Onboarding and training material**: A natural consequence of it being easy to write an ADR and getting into the habit of doing so is that new team members can simply read the record of ADRs to onboard.
* **Knowledge sharing**: The peer review phase allows sharing of expertise between team members.
* **Fewer meetings**: As decision making becomes asynchronous and as the team forms its social norms around the process, there should be less time required in meetings.

## When to write an ADR

* A decision is being made that required discussion between two or more people.
* A decision is being made that required significant investigation.
* A decision is being proposed for feedback / discussion.
* A decision is being proposed that affects multiple teams.

## Template

[Here](template.md).

## Related Reading

* [Suggestions for writing good ADRs](https://github.com/joelparkerhenderson/architecture-decision-record?tab=readme-ov-file#suggestions-for-writing-good-adrs)
* [ADRs at RedHat](https://www.redhat.com/architect/architecture-decision-records)
* [ADRs at Amazon](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/adr-process.html)
* [ADRs at GitHub](https://adr.github.io/)
* [ADRs at Google](https://cloud.google.com/architecture/architecture-decision-records)