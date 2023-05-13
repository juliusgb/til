# Print GitHub Webhook events and payloads

Many times, I want to know what the Json payload looks like when an event occurs on GitHub that triggers a [GitHub Actions workflow](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows).

In the past, I used to setup a Webhook that sends the json payload to my local server.
Now, I use GitHub Actions to print the whole json payload.

The [delete event](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#delete) sends a [delete webhook payload](https://docs.github.com/en/webhooks-and-events/webhooks/webhook-events-and-payloads#delete).

To print out the payload, I use the workflow below:

```yaml
name: Dump GH events and payloads

on:
  delete

jobs:
  test:
    name: debug
    runs-on: [ubuntu]

    steps:
      - name: print whole context
        env:
          GITHUB_CONTEXT: ${{ toJson(github) }}
        run: |
          echo "$GITHUB_CONTEXT"

      - name: print delete webhook payload
        run: |
          echo ${{ GITHUB.event }}
```


