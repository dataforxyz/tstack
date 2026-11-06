---
name: post-merge-double-check
description: Schedule and perform post-merge checks to confirm a merged change reached its target and works as expected. Use before merging during full-send, or when explicitly asked for post-merge verification.
---

# Post-merge Double Check

Ensure a change is merged and its expected behavior is observable after the
merge. This skill is a required pre-merge gate when used by `full-send`.

## Before the merge

1. Identify the PR, target branch, expected behavior, and the most direct way to
   verify it. Choose a behavior check time that allows for the real deployment,
   propagation, and observation delay; do not assume CI passing proves runtime
   behavior.
2. Before the merge action, create at least two durable, one-time follow-ups:
   - A merge check a few minutes after the planned merge. It verifies GitHub
     reports the PR merged and the merge commit is on the target branch.
   - A behavior check after the expected deploy and observation window. It
     verifies the deployed change using the direct signal identified above.
3. Put the repository, PR URL, target branch, expected behavior, verification
   instructions, and rescheduling rule in each follow-up. Record their IDs and
   scheduled times. Confirm the scheduler accepted both tasks before merging.

Use the harness's durable scheduler, such as `schedule_task` in T3 Code. In T3
Code, create separate one-time tasks bound to this thread so each check wakes
it. A sleep, an in-memory timer, or a reminder that depends on the current turn
staying active is not a scheduled follow-up. If the harness cannot reliably run
the checks later, do not merge; leave the PR ready and ask the user how to
proceed.

## At each scheduled check

- For the merge check, confirm both the merged state and that the merge commit
  is present on the target branch. If the PR is still open or the commit has
  not landed, schedule another check for a reasonable later time before
  finishing.
- For the behavior check, first confirm the change has reached the relevant
  environment. Then run the direct verification and inspect its result.
- If a check runs too soon for its signal to be available—for example, the
  deploy is still progressing or the expected observation window has not
  elapsed—schedule another one-time check for after that signal should be
  available. Do not mark the check complete or leave it without a next timer.
- Stop rescheduling when the check is verified or when a concrete blocker
  requires the user's input. Report failures and evidence plainly; a scheduled
  check by itself is not proof that the change works.

## Report

Include the PR and merge commit, each check's scheduled time and result, the
evidence for the behavior check, and any remaining blocker. If a check could
not be completed, say what prevented it and whether another follow-up is set.
