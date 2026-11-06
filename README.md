# commit-heartbeat

A small **reusable GitHub Actions workflow** that keeps a contribution streak alive. Built as a learning project.

## What it does

Each time it runs, it:

1. Gets today's date in **UTC** (GitHub counts contribution days in UTC).
2. Validates the numeric settings, then counts commits marked `Heartbeat-Bot: true` in today's UTC window. If the daily target (default **20**) is reached, normal runs skip writing.
3. Checks your streak: asks the GitHub API whether you have any contributions today. If not, this run's commits start the streak; if you do, the bot still makes its own commits. Your own commits never reduce the bot's count.
4. Makes up to `per_run` bot commits (default **5**). Four normal runs can reach 20; the last scheduled run is a backup that fills the remainder. Each commit adds a UTC timestamp and run ID to `updates.txt`, using a random subject from `.heartbeat/messages.txt`.
5. If the API check fails, it commits anyway.
6. If the repo has had no commit for 50 days and the normal plan is empty, it makes one keepalive commit. This cannot revive a schedule that GitHub already disabled.
7. Writes an Actions summary with your contribution count, the planned and created commits, confirmed pushes, and the decision reason. Contribution reporting stays enabled, including in dry runs.

**Be clear about what this means:** these are automated commits, not real work. They make your contribution graph look more active than it is. Anyone who opens the repo can see that. Don't use it to mislead employers, schools or anyone else.

## Setup

1. Click **Use this template → Create a new repository** (public). Don't fork: commits in forks do not count toward your contribution graph.
2. In the new repo, copy `examples/streak.yml` to `.github/workflows/streak.yml`. Keep the example and `.github/workflows/heartbeat.yml` too: the caller uses this local reusable workflow from the same revision.
3. Copy `examples/messages.txt` to `.heartbeat/messages.txt` and edit the messages.
4. For an existing repository, copy **both** workflow files together. The published `v3` workflow does not support these new inputs. A local call avoids depending on an unpublished release; update your local copy when you adopt future changes.
5. Add repository secrets (**Settings → Secrets and variables → Actions**):
   - `GIT_USER_NAME`: your name
   - `GIT_USER_EMAIL`: your GitHub no-reply email (`ID+username@users.noreply.github.com`)
   - `GH_USERNAME`: your GitHub username
   - `CONTRIB_TOKEN` (recommended): a fine-grained token with **Public repositories (read-only)** access and no extra permissions
6. **Settings → Actions → General → Workflow permissions → Read and write permissions**.
7. Test from **Actions → Streak guard → Run workflow**. Manual runs default to `dry_run: true`: preview the plan and contribution count without changing the repository. To create exactly one real test commit, tick `force` and untick `dry_run`.

The disadvantage is that future changes to `commit-heartbeat` must be copied into `heartbeat-log`. Keeping your original central setup would instead require publishing the upgrade under a new version tag and pointing the caller there.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `force` | `false` | Exactly 1 test commit, even if the target is reached; overrides normal and final counts |
| `dry_run` | `false` | Calculate and report the plan without committing or pushing; the example's manual-run form defaults to `true` |
| `target` | `20` | Bot commits per UTC day, on top of yours; integer 0–100 (`0` leaves only keepalive/force) |
| `per_run` | `5` | Max commits per normal run; integer 1–100 |
| `final_run` | `false` | Top up to `target`; before noon UTC, apply the normal cap instead |
| `keepalive_days` | `50` | Commit if the repo was idle this long and nothing else is planned; integer 1–59 |
| `messages_file` | `.heartbeat/messages.txt` | Commit message list |

`dry_run` takes precedence over every writing mode, including `force` and keepalive. Scheduled runs do not have dispatch inputs and continue writing normally. Invalid numeric settings fail before checkout or commits.

## Commit messages

The example contains 30 subjects mixing `docs:` for activity records and `chore:` for routine log maintenance. Add one subject per line. Blank lines and comments (including indented comments) are ignored, and Windows line endings are supported. A missing or empty list uses a default log-update subject.

Each commit retains the `Heartbeat-Bot: true` trailer so daily counting continues to work. Message subjects should describe the diff. Reserve `fix:` and `feat:` for actual bug fixes and new functionality; adding a timestamp does neither.

## Push recovery and reporting

The workflow retries failed pushes up to three times total, waiting 5 then 10 seconds between attempts. Temporary failures are retried even when the upstream branch has not changed or a fetch fails. If another writer advances the default branch, the workflow fetches and rebases its commits before retrying. It never force-pushes. A content conflict aborts the rebase and fails the run, preserving the remote changes. Persistent permission or branch-protection failures still require fixing the repository settings.

The summary distinguishes planned commits, local commits, and commits whose push succeeded. If a final run completes below the target, it emits a warning. A run that never starts cannot produce a summary or warning; check Actions history for missing runs.

## Development checks

CI runs actionlint on the workflows and example caller, including its ShellCheck integration on the Ubuntu runner. To run the same lint check locally with Go and ShellCheck installed:

```sh
go install github.com/rhysd/actionlint/cmd/actionlint@v1.7.7
actionlint -color .github/workflows/*.yml examples/streak.yml
```

## Limits

- Scheduled runs can be delayed or skipped; the daily target is best effort. The final-run guard treats a run before noon UTC as a normal run. This is a time-window heuristic, not a way to identify every delayed event, and a delay beyond that window cannot be detected reliably this way.
- If UTC midnight arrives during a commit batch, the batch stops and the next run recalculates. Already-created commits keep their explicit UTC timestamps. Push recovery also stops if the UTC day changes before a rebase.
- The contribution graph can take a while to update, so the count may lag.
- Public repos with no activity for 60 days get their schedules disabled.
- A successful push is not proof that a profile widget has counted it. GitHub may take up to 24 hours to show contributions, and the commit email and repository must meet its [contribution requirements](https://docs.github.com/en/account-and-profile/how-tos/contribution-settings/troubleshooting-missing-contributions). Missing the target of 20 is different from having no qualifying contribution that day.
- No workflow can guarantee an uninterrupted streak or immunity from account restrictions. GitHub documents that [scheduled jobs can be delayed or dropped](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule). Its [Acceptable Use Policies](https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies#4-spam-and-inauthentic-activity-on-github) prohibit automated inauthentic activity; changing commit-message prefixes does not change the nature of the activity. Use meaningful project updates rather than filler commits if avoiding that risk is the priority.

## Disclaimer

This is for **education only**. It may conflict with GitHub's [Terms of Service](https://docs.github.com/en/site-policy/github-terms/github-terms-of-service) and [Acceptable Use Policies](https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies), which forbid things like inauthentic activity and putting excess load on Actions. GitHub may disable workflows or take action on accounts. **You use it at your own risk.** The authors are not responsible for misuse or any outcome of using it.

## Alternatives

This is deliberately tiny. If you want something more complete, there are more powerful tools in the [GitHub Marketplace](https://github.com/marketplace?type=actions), and you can build your own with scheduled workflows, the GraphQL API, or a GitHub App.

## License

MIT
