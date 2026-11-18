# Deployment notes

This file intentionally contains no credentials or provider-specific secret values.

## Before deployment

1. Configure the environment variables listed in `.env.example` in your hosting provider's secret/environment settings.
2. Provision MongoDB and configure `MONGODB_URI`.
3. Configure IBM Watson Speech to Text and watsonx credentials for the features being deployed.
4. Configure Cloudinary / Agora only if the corresponding application features are enabled.
5. Review the custom Socket.IO server in `server.js`; some serverless platforms do not support long-lived custom WebSocket servers in the same way as a persistent Node.js host.
6. Run `npm run build` and test meeting, transcription, and summary flows in the target environment.

## Local production check

```bash
npm ci
npm run build
npm start
```

## Secret handling

Never put API keys, database passwords, private service instance identifiers, or deployment environment dumps in this repository. Use your hosting provider's secret manager. If a credential has ever been exposed, revoke or rotate it before deploying.
