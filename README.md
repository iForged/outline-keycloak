# OutlineWebhook
This project is a fork of [WirtsLegs/outline-keycloak](https://github.com/WirtsLegs/outline-keycloak), which itself is modified from [Frando's Outline Webhook example](https://gist.github.com/Frando/aa561ca7e6c72ab64b5d17df911c0b1f). It provides a simple webhook server built with Deno that syncs Outline groups from Keycloak on user sign-in.

## Prerequisites
- A Keycloak client configured in your realm with service account permissions to access Keycloak user groups
- A webhook configured in Outline (note that the container expects the webhook to come to `/webhook`)

## Running the Application

### Using Docker directly
1. Build the Docker image:
   ```bash
   docker build -t outline-webhook .
   ```
2. Run the Docker container:
   ```bash
   docker run -p 8000:8000 --env-file .env outline-webhook
   ```
3. The server will be accessible on port `8000`.

### Using Docker Compose
1. Copy `.env.example` to `.env` and fill in your values.
2. Start the service:
   ```bash
   docker compose up -d --build
   ```
   Compose automatically reads `.env` from the same directory and substitutes the `${VAR}` placeholders in `docker-compose.yml`. You can verify the resolved config without starting anything via:
   ```bash
   docker compose config
   ```

## Environment Variables
- `DENO_ENV`: Set to `production` by default in the Dockerfile.
- `WEBHOOK_SECRET`: Your Outline webhook secret.
- `OUTLINE_ENDPOINT`: The API endpoint of your Outline instance (e.g., `https://your-outline-instance.com/api`).
- `OUTLINE_API_TOKEN`: Your Outline API token.
- `KEYCLOAK_ENDPOINT`: The endpoint of your Keycloak instance (e.g., `https://your-keycloak-instance.com`).
- `KEYCLOAK_REALM`: Your Keycloak realm.
- `KEYCLOAK_CLIENT_ID`: Your Keycloak client ID.
- `KEYCLOAK_CLIENT_SECRET`: Your Keycloak client secret.
- `GROUP_SYNC_PREFIX`: (optional) Comma-separated list of group name prefixes to restrict syncing to, e.g. `wiki_,docs_`. Only Keycloak groups matching one of these prefixes are created/joined in Outline, and only matching Outline groups are considered when removing stale memberships. Leave empty to sync all groups (default behavior).

## Group Sync Behavior
On each `users.signin` webhook event, the server:
1. Looks up the signed-in user in Keycloak by email.
2. Fetches the user's Keycloak groups, filtered by `GROUP_SYNC_PREFIX` if set.
3. Compares them against the user's current Outline groups (also filtered by prefix) and:
   - Creates any missing groups in Outline.
   - Adds the user to groups they're missing.
   - Removes the user from prefixed Outline groups no longer present in Keycloak.

Groups outside the configured prefix(es), in either Keycloak or Outline, are never touched.

## Credits
This project is a fork of [WirtsLegs/outline-keycloak](https://github.com/WirtsLegs/outline-keycloak), which is inspired by and based on [Frando's Outline Webhook example](https://gist.github.com/Frando/aa561ca7e6c72ab64b5d17df911c0b1f). Special thanks to WirtsLegs and Frando for the original implementations.
