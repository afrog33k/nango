import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Sync notifications

Nango uses webhooks to notify your app in real-time when Sync jobs finish.

## Webhook configuration

To receive job completion notifications, specify a webhook URL using the `NANGO_WEBHOOK_URL` environment variable (in the `.env` file at the root of the nango folder).

Nango will send a `POST` request to this URL on job completions.

## Job success notifications

When a Sync job successfully completes, you might want to quickly use the recently imported data.

The body payload of the webhook you will receive is:
```json
{
    type: 'job_success',
    job: {...},              // Detailed information about the completed job (all fields: https://github.com/NangoHQ/nango/blob/main/packages/worker/lib/models/job.model.ts)
    sync: {...},             // The configuration of your Sync (all fields: https://github.com/NangoHQ/nango/blob/main/packages/core/lib/sync.model.ts)
    info: {
        added: [...],        // Ids of the added records (same Ids are used for the raw & mapped data)
        updated: [...],      // Ids of the updated records
        deleted: [...]       // Ids of the deleted records
    }
}
```

## Job failure notifications

When a Sync job fails, you might want to send an alert to quickly investigate the issue. 

The body payload of the webhook you will receive is:
```json
{
    type: 'job_failure',
    job: {...},              // Detailed information about the completed job, including the error description, stacktrace, etc. (all fields: https://github.com/NangoHQ/nango/blob/main/packages/worker/lib/models/job.model.ts)
    sync: {...}             // The configuration of your Sync (all fields: https://github.com/NangoHQ/nango/blob/main/packages/core/lib/sync.model.ts)
}
```
