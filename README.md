# commit-heartbeat

A small **reusable GitHub Actions workflow** that keeps a contribution streak alive. Built as a learning project.

## What it does

Each time it runs, it:

1. Gets today's date in **UTC** (GitHub counts contribution days in UTC).
2. Counts how many commits the bot already made today. If it already made the target (default **15**), it stops.
3. Asks the GitHub API how many contributions you have today.
4. If you have fewer than the target, it makes enough commits to reach it. Each commit adds a UTC timestamp to `updates.txt` with a random message from `.heartbeat/messages.txt`.
5. If the API check fails, it commits anyway (fail-safe).
6. If the repo has had no commit for 50 days, it makes one commit so GitHub doesn't disable the schedule.

**Be clear about what this means:** these are automated commits, not real work. They make your contribution graph look more active than it is. Anyone who opens the repo can see that. Don't use it to mislead employers, schools or anyone else.

## Setup

1. Click **Use this template → Create a new repository** (public). Don't fork: commits in forks do not count toward your contribution graph.
2. In the new repo, move `examples/streak.yml` to `.github/workflows/streak.yml` and replace `OWNER` with the account that hosts this tool.
3. Copy `examples/messages.txt` to `.heartbeat/messages.txt` and edit the messages.
4. Delete the copied `.github/workflows/heartbeat.yml` from your new repo if you call the upstream `@v1` version (optional).
5. Add repository secrets (**Settings → Secrets and variables → Actions**):
   - `GIT_USER_NAME`: your name
   - `GIT_USER_EMAIL`: your GitHub no-reply email (`ID+username@users.noreply.github.com`)
   - `GH_USERNAME`: your GitHub username
   - `CONTRIB_TOKEN` (recommended): a fine-grained token with **Public repositories (read-only)** access and no extra permissions
6. **Settings → Actions → General → Workflow permissions → Read and write permissions**.
7. Test it from the **Actions** tab with **Run workflow** and `force` ticked.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `force` | `false` | Make 1 commit even if nothing is needed |
| `target` | `15` | Contributions per UTC day |
| `keepalive_days` | `50` | Commit if the repo was idle this long |
| `messages_file` | `.heartbeat/messages.txt` | Commit message list |

## Limits

- Scheduled runs can be late or skipped. That's why there are several runs a day.
- The contribution graph can take a while to update, so the count may lag.
- Public repos with no activity for 60 days get their schedules disabled.

## Disclaimer

This is for **education only**. It may conflict with GitHub's [Terms of Service](https://docs.github.com/en/site-policy/github-terms/github-terms-of-service) and [Acceptable Use Policies](https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies), which forbid things like inauthentic activity and putting excess load on Actions. GitHub may disable workflows or take action on accounts. **You use it at your own risk.** The authors are not responsible for misuse or any outcome of using it.

## Alternatives

This is deliberately tiny. If you want something more complete, there are more powerful tools in the [GitHub Marketplace](https://github.com/marketplace?type=actions), and you can build your own with scheduled workflows, the GraphQL API, or a GitHub App.

## License

MIT
