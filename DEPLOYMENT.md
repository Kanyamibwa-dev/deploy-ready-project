# Deployment Guide

## Cloud provider and service

**Render.com** (free "Web Service" tier, deployed from a Docker image).

### Why Render, and why not AWS/GCP/Azure

The original plan was AWS EC2 (`t2.micro`, free tier). In practice, AWS
requires a verified payment method before it will let you launch an actual
instance — even the "no-cost" `t2.micro` free tier sits behind that wall
once you try to use compute, not just create an account. Without access to
a card, that path was blocked.

Render was chosen as a deliberate substitute: it runs real Docker
containers, requires no payment method at all on its free tier, and gives
a genuine public HTTPS URL — which satisfies the actual goal of Part 3
(a running, publicly reachable service). The trade-off is that Render is a
**PaaS** (Platform-as-a-Service), not raw IaaS, so there is no EC2-style
VM to SSH into, no security group to configure by hand, and no IAM role to
scope — Render manages the underlying infrastructure itself. If a card
becomes available later, the app can be redeployed to AWS EC2 with no
code changes, since the Dockerfile and app are cloud-agnostic.

## 1. Create the Render Web Service

1. Sign up at [render.com](https://render.com) (GitHub login is fastest,
   no card required).
2. Push this repo to GitHub first (see main `README.md`).
3. In the Render dashboard, click **New +** → **Web Service**.
4. Connect your GitHub account and select this repository.
5. Render auto-detects the `Dockerfile` at the repo root — leave
   **Environment: Docker**.
6. **Instance Type**: Free.
7. Add an environment variable: `PORT` = `10000` (Render's free tier
   listens on port `10000` by default — our app already reads `PORT` from
   the environment, so no code change is needed).
8. Click **Create Web Service**. Render will build the Docker image and
   deploy it. First build can take a few minutes.
9. Once live, Render gives you a public URL like
   `https://deployready-xxxx.onrender.com`. That's your server's public
   address for the rest of this guide.

## 2. Get the deploy hook and connect the pipeline

1. In the Render dashboard, go to your service → **Settings** →
   **Deploy Hook**. Copy the URL shown (it's a private, secret URL —
   hitting it with a simple `curl` or browser request triggers a new
   deploy).
2. In your GitHub repo: **Settings → Secrets and variables → Actions →
   New repository secret**. Add:

| Secret name          | Value                                              |
|-----------------------|-----------------------------------------------------|
| `RENDER_DEPLOY_HOOK`  | The deploy hook URL from step 1                    |
| `RENDER_SERVICE_URL`  | Your service's public URL, e.g. `https://deployready-xxxx.onrender.com` |

Push to `main` and watch the **Actions** tab — the pipeline tests, builds
and pushes to GHCR, then hits the Render deploy hook and verifies
`/health`.

## 3. Verify

```bash
curl https://deployready-xxxx.onrender.com/health
# {"status":"ok"}
```

Note: Render's free web services spin down after ~15 minutes of no
traffic and take 30-60 seconds to wake back up on the next request. If
`/health` seems slow the first time you hit it, that's why — it's not
broken, just waking up.

## 4. Checking the container / logs

Render doesn't give SSH access on the free tier, but the dashboard covers
the same needs:

- **Is it running?** Open the service in the Render dashboard — the
  status badge shows **Live** (green) or **Deploying**/**Failed**.
- **View logs**: click the **Logs** tab on the service page — this shows
  the same stdout/stderr `docker logs` would show, streamed live.
- **Restart manually**: **Manual Deploy → Deploy latest commit**, or the
  **Restart** option in the service's menu.

## Bonus (not implemented in this submission)

If a card becomes available, the natural next step is redeploying this
same Docker image to AWS EC2 following the original SSH-based deploy
model. Other optional extensions: Terraform for infra-as-code, a
monitoring alarm on `/health`, or an automatic rollback step if the
post-deploy health check fails.