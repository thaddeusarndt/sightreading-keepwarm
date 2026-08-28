# sightreading-keepwarm

A tiny scheduled GitHub Action that pings
[Notespawn](https://notespawn.com) (Render service `sightreading-generator`) so
the free tier never cold-starts. No secrets, no code — just `curl` to the public
health endpoint.

The site, the iOS app, and the embed on
[thaddeusarndt.com/sight-reading](https://thaddeusarndt.com/sight-reading) all
hit the same Render service, so one ping keeps all three instant.

## Why it loops

GitHub treats the `*/10` cron as a hint, not a promise. Measured delivery on
this repo is roughly **one run per hour**, and on 2026-08-06 there was a
six-hour hole. Render's free tier sleeps after **15 minutes** idle — so the
original "one curl per run" version left the app asleep for most of the day.

Each run now pings every 4 minutes for ~55 minutes, which bridges the gap
between whatever runs GitHub actually hands out. Overlapping runs are harmless;
this is a public repo, so the minutes are free.

A run fails only after **three consecutive** bad responses (~8 minutes), which
means a "Run failed" e-mail is a real outage rather than a blip. The previous
version swallowed every curl error and always reported success.

## Why it only runs during the day

Render's free tier gives the whole workspace **750 instance hours a month** and
bills time the service is *awake*, not traffic. Keeping it warm around the clock
costs ~744 hours in a 31-day month — which is under the cap, but by six hours,
with no room for a redeploy or a second service. On 2026-08-27 that produced a
"637 of 750" warning e-mail and a real risk of suspension.

So the schedule runs 6am–midnight Pacific (`13:00–07:00` UTC) instead of 24/7:
roughly 19 awake hours a day once the last run's tail is counted, or **~589
hours a month**. The only cost is one cold start for the first visitor after a
quiet night. Cron is fixed UTC, so the window slides to 5am–11pm once DST ends.

## Notes

- Occasional `The job was not acquired by Runner of type hosted` failures are
  GitHub capacity problems, not ours. Nothing to fix; the next run recovers.
- The permanent fix is Render's **Starter** plan (~$7/mo, always on), which
  would make this repo unnecessary. See `plan: free` in `render.yaml` over in
  the `sightreading` repo.
- To pause: disable the **keep-warm** workflow in the Actions tab.
- GitHub disables scheduled workflows after 60 days with no repo activity —
  push any commit (or run the workflow manually) to re-enable.
