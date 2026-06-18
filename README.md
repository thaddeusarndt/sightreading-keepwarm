# sightreading-keepwarm

A tiny scheduled GitHub Action that pings
[the Sight-Reading Generator](https://sightreading-generator.onrender.com) every
~10 minutes so Render's free tier never cold-starts. No secrets, no code — just a
`curl` to the public health endpoint.

The embedded tool on [thaddeusarndt.com/sight-reading](https://thaddeusarndt.com/sight-reading)
stays instant because of this.

To pause it: disable the **keep-warm** workflow in the Actions tab.

Note: GitHub disables scheduled workflows after 60 days with no repo activity —
push any commit (or run the workflow manually) to re-enable if that happens.
