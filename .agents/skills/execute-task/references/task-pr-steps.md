# Open Task PR (PR_INTEGRATION=true only)

Skip this step entirely when `PR_INTEGRATION=false`.

1. Push the task branch:
   ```bash
   git push -u origin plan/{plan-slug}/{task-path}
   ```
2. Open a PR targeting the integration branch:
   ```bash
   gh pr create \
     --title "plan({plan-slug}/{task-path}): {task title}" \
     --body "## Task: {task title}

   {objective from task.md}" \
     --base plan/{plan-slug} \
     --head plan/{plan-slug}/{task-path}
   ```
3. Record the PR URL in `status.md` under `**PR**`.
