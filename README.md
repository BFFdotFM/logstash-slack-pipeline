# Logstash/Beats Slack Alerts

We're using Elastic's `heartbeat` with Logstash to monitor BFF.fm service status.
We visualise this in Kibana's uptime board, while this simple logstash pipeline also
sends state-change alerts to Slack.

# Usage

Use the files in `/src` in your beats logstash pipeline.

Set the following keystore variables:

* `SLACK_ALERTS_WEBHOOK`: The secret part of your Slack webhook URL (after `services/…`)
* `KIBANA_UPTIME_URL`: We set this to our Kibana dashboard URL for viewing the uptime history.

# Basic concept

* We ingest heartbeat beats events, and if the event is `down`, we throttle it to require two `down` events within a 3 minute period. We monitor our pages every 60s, you might need to adjust the window if you're monitoring more/less frequently. We're looking for 2/3 before we declare the site down.
* Not throttling gave us a lot of down/up noise due to occasional read failures in heartbeat.
* We store the state of the current monitor event in `memcached`, so we can compare state in subsequent events.
* If the event isn't throttled and the monitor state between events has changed, we send a Slack alert.

# Future work

* Add the capability to send repeat alerts for `down` state after some time.
