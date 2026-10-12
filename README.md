# Deploy and Host Cal.com on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/calcom-production?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=calcom-production)

[Cal.com](https://cal.com/) is the open-source Calendly alternative: public booking pages, event types with custom durations and questions, availability schedules, buffer times, Google Calendar and Meet sync, video links, webhooks and embeddable booking widgets. This template runs the official Cal.com image with Postgres, generates every secret for you, and points the app at your Railway domain without rebuilding the image.

## About Hosting Cal.com

The stack is two services: Cal.com and Postgres.

- **Upstream's own image, pinned.** Cal.com runs from the official `calcom/cal.com` image at a fixed tag (`v6.2.0`), not `latest`, so a redeploy never surprises you with an untested version.
- **Your URL, no rebuild.** Cal.com bakes its public URL into the Next.js build. The start command rewrites the built URL to `NEXT_PUBLIC_WEBAPP_URL` on every boot (upstream's own replace script), so booking links, emails and OAuth callbacks use your Railway domain or custom domain.
- **Migrations on every boot.** The start command waits for Postgres, applies Cal.com's Prisma migrations and seeds the app store before the server starts. Railway's health check (`/api/auth/session`) only switches traffic once that is done; first boot applies about 590 migrations and takes a minute or two.
- **Lean start.** The server is started with `next start` directly instead of upstream's `yarn` + `turbo` wrapper, which cuts about 350 MB of idle memory and keeps every environment variable visible to the app.
- **Secrets generated.** `NEXTAUTH_SECRET` and the 32-character `CALENDSO_ENCRYPTION_KEY` (which encrypts stored calendar credentials) are generated per deployment.
- **No Redis, no volume on the app.** The web app keeps everything, including avatars, in Postgres, which has its own volume.

## Common Use Cases

- A personal booking page (`your-domain/you`) that replaces Calendly, on your own infrastructure
- Sales, support or interview scheduling with event types, questions and buffer times
- Booking widgets embedded on your website, with webhooks into your CRM or automation tools
- Scheduling where booking data must stay on servers you control

## Dependencies for Cal.com Hosting

- Postgres 17 (included, private network only)

### Deployment Dependencies

- [Cal.com self-hosting documentation](https://cal.com/docs/self-hosting/installation)
- [Cal.com on GitHub](https://github.com/calcom/cal.diy)
- [Template source on GitHub](https://github.com/nomideusz/calcom-railway)

### Implementation Details

**Create the admin account first.** Open the Cal.com service's Railway domain as soon as the deploy is green: a new instance sends you to the setup wizard, and the first account created there becomes the instance admin. Until then anyone who finds the URL could claim it.

**Turn on 2FA to unlock admin settings.** Cal.com treats an admin without two-factor authentication as inactive in production. Enable 2FA under Settings, Security, then sign in again to reach `/settings/admin`.

**Public sign-up is open by default.** Anyone can create an account on your instance at `/signup`. To close it, enable the `disable-signup` flag under Settings, Admin, Features (`/settings/admin/flags`), or run `UPDATE "Feature" SET enabled = true WHERE slug = 'disable-signup';` in the Postgres service's database console.

**Custom domain.** After adding one, set both `NEXT_PUBLIC_WEBAPP_URL` and `NEXTAUTH_URL` to `https://your-domain` and redeploy; the start command rewrites the app to the new URL.

**Google Calendar and Meet (optional).** Create an OAuth client (Web application) in Google Cloud, enable the Google Calendar API, add `https://your-domain/api/integrations/googlecalendar/callback` (and `https://your-domain/api/auth/callback/google` for Sign in with Google) as redirect URIs, and paste the downloaded client JSON into `GOOGLE_API_CREDENTIALS`. Set `GOOGLE_LOGIN_ENABLED=true` to show the Google sign-in button.

**Email (optional).** Booking confirmations, reminders and password resets need SMTP: fill in the `EMAIL_*` variables. Railway only allows outbound SMTP on the Pro plan; without mail, bookings still work and show in the app.

**License key (optional).** The image is Cal.com's AGPL edition. Enterprise features (teams, workflows, insights, SSO) need a commercial key in `CALCOM_LICENSE_KEY`; everything for individual scheduling works without one.

**Back up the encryption key.** `CALENDSO_ENCRYPTION_KEY` encrypts connected calendar and app credentials in the database. Changing or losing it breaks every connected calendar.

**About versions.** In 2026 Cal.com moved its open-source code to [Cal.diy](https://github.com/calcom/cal.diy) (MIT, enterprise code removed), which publishes no Docker image. `v6.2.0` is the last official Cal.com image and includes the fixes for every Cal.com security advisory published as of October 2026. The template will move to an official Cal.diy image once one is released.

**Resources.** In testing the Cal.com server idled at 520–650 MB of RAM and Postgres at about 40 MB. Next.js runs out of heap at 512 MB, so Cal.com needs the Hobby plan or higher.

## Why Deploy Cal.com on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying Cal.com on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
