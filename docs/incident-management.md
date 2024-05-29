# Incident Management for InstructLab

## Attribution

Before we start, I would like to mention that this document and process is inspired
by and influenced heavily from [here][gist] originally authored by [coderanger][ranger].

## Problem Statement

With the multiple systems that come together for the open source project InstructLab
to work, we need a catalogue of major issues and problems so we can track and find
critical bugs in the open. Building a formalized incident management process is
the first step in building this knowledge database.

## Proposed Solution

Create a light weight but formalized Incident Management process. Whomever calls
for the Incident is the initial "Incident Commander" [IC] and becomes the hub of
the information, starts the conversation in the shared Slack channel
and has a shared timeline (google doc or the like).
After "a reasonable" amount of time they can hand off the IC
to someone else, and so on till the Incident is resolved.

### What counts as an incident?

Anything with community visible negative consequences. In most cases this
will be an outage or downtime event but some non-outage incidents include severe
performance degradations and security events.

## When should I file an incident report?

Preferably after the incident is resolved. Or at least don't consider it even remotely
a priority to do the paperwork in the middle of an ongoing issue. But if possible do
think about recording timeline information that isn't in Slack or the Mailing list which
might otherwise be lost, so that you will have them for the report later.

## Incident Template

## [Incident Date and Title]

Incident Commander: [Name of IC]

## Summary

[A quick one or two sentence description of the issue from a high-level view.]

## Duration

[The amount of time of the user impact. This does not include any time spent after restoration of service but before the formal end of the incident.]

## User Impact

[Describe the impact to end users, such as which features or services were unavailable.]

## Timeline

[A timeline of events from the start of the incident until the Incident Commander declares it over. All times should be in UTC.]

* 01:23 - Incident start.
* 23:34 - Incident concluded.

## Proximate Trigger

[The most direct trigger of incident. Often times this will be a human error such as a code bug that was missed in review or an operational mistake. Our process is blameless and we document our mistakes so that we can learn from them. But try to not turn this into a personal callout, even of yourself.]

## Root Cause

[Root causes are the underlying deeper problem that lead to the incident. For example, if the proximate trigger was a bug missed in review then a root cause might be missing static analysis tooling in CI that could have caught it, or missing code review guidelines. Root causes should never be human error, they are systemic issues that create the conditions for human error to become a problem. Also root causes are sometimes slippery, you can trace the chain of events back infinitely far if you try hard enough but putting "Root Cause: 13 billion years ago a singularity expanded into the Big Bang" is not productive. Look for root causes that help guide our future path rather than documenting every contributing factor for its own sake.]

## Detection

[How was this problem noticed? User reports, automated alerts, etc.]

## Resolution

[What steps were taken to resolve the incident. Try to be specific, such as linking to a commit/PR for code fixes or listing the commands used for an interactive fix, as these can help guide future improvements.]

## What Went Well?

[Any notes about things that went well in our process during the handling of the incident.]

## What Went Poorly?

[Like the above but things that went poorly. This is only related to the process and handling, the incident itself is probably something that went poorly too but that is discussed above.]

## Where We Got Lucky?

[Any places where things went well but more due to happenstance, such as one bug cancelling out another or an issue being noticed early before automated detection warned us.]

## Action Items

[Tasks we should take away from this incident to prevent it from recurring or to improve our handling of similar incidents in the future. Action items can be divided into 5 categories listed below. It's possible not all categories will have an action item, they are only to help guide your thinking.]

### Detect

[Ways to improve the detection of problems.]

### Respond

[Ways to get eyeballs on the incident faster.]

### Contain

[Ways to limit the damage of incidents.]

### Prevent

[Ways to reduce the chances of the proximate triggers of this incident.]

### Eliminate

[Ways to solve the root causes of this incident so as to make it structurally impossible.]

_All times in UTC._

## Next steps

1. Create a incident@instructlab.ai mailing list
1. Create a incidents directory in the community repository
1. Create a #incident channel in Slack


[gist]: https://gist.github.com/coderanger/cbf7e80b76a7b2ff284ab592d798de8e
[ranger]: https://github.com/coderanger
