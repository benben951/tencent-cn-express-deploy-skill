# Troubleshooting

## 1. Overseas hosts are blocked or unstable in China

Symptom:

- `vercel.app` opens on desktop but not on mobile.
- Different carriers behave differently.

Action:

- Move final delivery to Tencent Cloud.

## 2. Tencent deploy succeeds but function URL is broken

Symptom:

- Root returns `443` with cloud error JSON.

Common root causes:

- Backend entrypoint is wrong.
- Tencent executed browser `app.js` instead of Express server.
- Backend still uses unsupported syntax like optional chaining.

Fix:

- Keep root backend entry at `app.js`.
- Move frontend bundle to `client.js`.
- Remove optional chaining from backend code if using `Nodejs16.13`.

## 3. Health endpoint works but AI or access code is wrong

Symptom:

- `deepseek_ready` is false.
- `model` or other values appear as literal `${env.X}`.
- Unlock code is always rejected.

Fix:

- Tencent `http` deployments may not substitute `${env.*}` reliably.
- Write concrete values into `serverless.yml` if needed for final deployment.

## 4. Trigger missing after redeploy

Symptom:

- `ListTriggers` returns empty array.

Fix:

- Recreate the `http` function URL trigger after deploy.
- Read the `ExtranetUrl` from `TriggerDesc`.

## 5. Fastest verification order

1. `GET /api/health`
2. `GET /`
3. `POST /api/unlock` if access protection exists
4. `POST /api/ai/insights`

Do not assume the site is good just because deployment reports success.
