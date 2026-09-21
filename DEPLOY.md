# Staging deployment runbook

## Trigger and target

A successful push to `main` runs the test job, publishes the application image to GHCR, and triggers the Render Web Service through its deploy API. The Render service uses a disposable Render PostgreSQL database. Its `DATABASE_URL` environment variable must contain the database's internal connection string.

GitHub stores `RENDER_API_KEY` and `RENDER_SERVICE_ID` as repository secrets. `STAGING_URL` is a repository variable without a trailing slash. No credential value belongs in this repository.

## Verification

The deploy job retries both checks for a bounded period and fails unless both return HTTP 200:

```sh
curl -f "$STAGING_URL/health"
curl -f "$STAGING_URL/items"
```

`/health` proves process liveness. `/items` also proves that the application can query PostgreSQL.

## Rollback

Every published image has an immutable commit tag. To roll back, select the last known-good commit SHA, configure the Render service to deploy `ghcr.io/Panaki-GILLOT/hbtn-devops-pipeline-lab:<commit-sha>`, trigger a deploy, then repeat both verification requests.

## Cleanup

When the lab is finished, remove the disposable Render Web Service and PostgreSQL database. Revoke the temporary Render API key, remove the GitHub repository secrets and variable, and remove or privatize the GHCR package according to the chosen cleanup policy.
