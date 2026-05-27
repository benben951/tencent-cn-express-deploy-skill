---
name: tencent-cn-express-deploy
description: Use when deploying a Node/Express web app for reliable mainland China access on Tencent Cloud Serverless. Best for projects with static frontend files plus backend API routes, especially when Vercel or other overseas hosts are slow or blocked in China.
---

# Tencent CN Express Deploy

Deploy an Express-based website to Tencent Cloud `http` component when the user needs a mainland-China-friendly public URL.

Use this skill when:

- The project has `server.js` or another Express entrypoint.
- The user needs mobile access from mainland China.
- The project includes static frontend files plus API endpoints.
- Existing overseas links like `vercel.app` are unstable in China.

Do not use this skill for:

- Pure static sites that can live on simpler static hosting.
- Projects that already have a domestic deployment pipeline the user wants to preserve.

## Workflow

1. Confirm the app structure.
2. Separate frontend and backend entrypoints.
3. Prepare `serverless.yml` for Tencent Cloud `http` component.
4. Configure Tencent credentials with `scf credentials set`.
5. Deploy with `scf deploy --force`.
6. Create or verify the function URL trigger.
7. Validate root path, health endpoint, and AI endpoint.

## Structure Rules

- Keep backend entrypoint at root as `app.js` because Tencent's Express web-function path may execute `node app.js` directly.
- If the project also has a browser-side `app.js`, rename that frontend file to `client.js`.
- Static assets should live under `public/`.
- Express server should export `app` and only call `listen(...)` when run directly.

Example backend root entry:

```js
const app = require('./server');

if (require.main === module) {
  const port = Number(process.env.PORT || 9000);
  app.listen(port, () => {
    console.log(`READY http://0.0.0.0:${port}`);
  });
}

module.exports = app;
```

## Required Tencent Files

- `serverless.yml`
- `app.js`
- `server.js`
- `public/`

See the template in [templates/serverless.yml](templates/serverless.yml).

## Credential Setup

Use:

```bash
npx scf credentials set -i <SecretId> -k <SecretKey> -o
```

Then deploy:

```bash
npx scf deploy --force
```

If the project needs environment variables, prefer writing explicit values into `serverless.yml` for Tencent `http` deployments when `${env.*}` substitution is unreliable in practice.

## Post-Deploy Trigger

After deploy, verify the function URL trigger exists:

```js
ListTriggers({
  FunctionName: 'your-function-name',
  Namespace: 'default',
  Offset: 0,
  Limit: 20
})
```

If missing, create one with:

- `Type: "http"`
- `AuthType: "NONE"`
- `EnableExtranet: true`

The returned `TriggerDesc` contains:

- `ExtranetUrl`
- `ExtranetHTTPUrl`

These are the public links to share.

## Validation Checklist

- Root path returns `200` or `302` as expected.
- `/unlock.html` works if access protection is enabled.
- `/api/health` returns expected JSON.
- AI endpoints return real results, not placeholder env strings.

## Common Failures

- `runtime参数错误`
  Use a Tencent-supported runtime. Prefer `Nodejs16.13` for compatibility.

- `Unexpected token '.'`
  Tencent runtime may not support optional chaining in server code. Remove `?.` from backend files.

- `Cannot find module '/var/user/app.js'`
  Tencent is launching root `app.js`. Ensure root `app.js` exists and starts the server when run directly.

- Root page opens but AI is unavailable
  Environment variables were not injected correctly. Recheck the `environments` block.

- Trigger exists but URL fails
  Recheck backend entrypoint behavior and function startup.

Detailed notes: [references/troubleshooting.md](references/troubleshooting.md)
