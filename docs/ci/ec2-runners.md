# CI/CD with EC2 Runners

## Problem

Projects run E2E tests on EC2 runners for small, medium, and large jobs. (Some
projects may have other names for such jobs, like `smoke` in `training`).

These runners are used to get access to accelerated hardware (e.g., GPUs) to
run compute intensive processes.

Access to instances with such hardware is sometimes limited and depends on the
current demand among all EC2 users in a particular zone. This means that
sometimes requested instance types are not available, which makes jobs that
rely on these instances fail.

## Solution

Availability depends on a particular zone. If a zone is busy, we can try
another zone.

For this, a new
[`launch-ec2-runner-with-fallback`](https://github.com/instructlab/ci-actions/tree/main/actions/launch-ec2-runner-with-fallback)
action was implemented in `ci-actions` repository. If adopted, this action will
walk through AZs and try to request an instance in each AZ until it finds one.

All projects that rely on AWS EC2 runners should adopt the
`launch-ec2-runner-with-fallback` action in all of the jobs to avoid fluke test
failures.
