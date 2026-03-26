# Sake – Complete Installation Guide

>Self-hosted Library Management with KOReader sync, deployed for free on Render + Turso + Backblaze B2.  
Sake is not made by me — all credits go to [Sudashiii](https://github.com/Sudashiii/).  
This tutorial is detailed and well-formatted, but I may have missed something — if so, just comment below.  
If you have any problems also comment bellow.  
To see more about Sake, check out the [official repository](https://github.com/Sudashiii/Sake) or my [Reddit post](https://www.reddit.com/r/koreader/comments/1s2apdl/comment/oc8lr9d/).

# Prerequisites

>All of these accounts don't require a credit card.

* GitHub account
* Turso account (free)
* Backblaze account (free)
* Render account (free)

# Part 1 – GitHub (Fork the Repository)

1. Go to [github.com/Sudashiii/Sake](https://github.com/Sudashiii/Sake)
2. Click **"Fork"** → **"Create Fork"**
3. You now have your own copy at [`github.com/YOURNAME/Sake`](http://github.com/YOURNAME/Sake)

# Part 2 – Fix the Dockerfile

In your fork, go to `sake/Dockerfile` and click the **pencil (edit)** icon.

Replace the entire contents with the following(this tells render to actually use the .env file while building **otherwise your deploy will fail!**):

    FROM oven/bun:1 AS deps
    WORKDIR /app
    RUN if command -v apk >/dev/null 2>&1; then \
    		apk add --no-cache python3 make g++; \
    	elif command -v apt-get >/dev/null 2>&1; then \
    		apt-get update && apt-get install -y --no-install-recommends python3 make g++ && rm -rf /var/lib/apt/lists/*; \
    	else \
    		echo "Unsupported base image: no apk/apt-get found" >&2; exit 1; \
    	fi
    COPY package.json bun.lock ./
    COPY .npmrc ./
    RUN bun install --frozen-lockfile --omit peer
    
    FROM deps AS migrator
    COPY . .
    
    FROM deps AS builder
    ARG PUBLIC_WEBAPP_VERSION=dev-local
    ARG PUBLIC_WEBAPP_GIT_TAG=
    ARG PUBLIC_WEBAPP_COMMIT_SHA=
    ARG PUBLIC_WEBAPP_RELEASED_AT=
    ARG LIBSQL_URL
    ARG LIBSQL_AUTH_TOKEN
    ARG S3_ENDPOINT
    ARG S3_REGION
    ARG S3_BUCKET
    ARG S3_ACCESS_KEY_ID
    ARG S3_SECRET_ACCESS_KEY
    ARG S3_FORCE_PATH_STYLE
    ARG ACTIVATED_PROVIDERS
    ENV PUBLIC_WEBAPP_VERSION=${PUBLIC_WEBAPP_VERSION}
    ENV PUBLIC_WEBAPP_GIT_TAG=${PUBLIC_WEBAPP_GIT_TAG}
    ENV PUBLIC_WEBAPP_COMMIT_SHA=${PUBLIC_WEBAPP_COMMIT_SHA}
    ENV PUBLIC_WEBAPP_RELEASED_AT=${PUBLIC_WEBAPP_RELEASED_AT}
    ENV LIBSQL_URL=${LIBSQL_URL}
    ENV LIBSQL_AUTH_TOKEN=${LIBSQL_AUTH_TOKEN}
    ENV S3_ENDPOINT=${S3_ENDPOINT}
    ENV S3_REGION=${S3_REGION}
    ENV S3_BUCKET=${S3_BUCKET}
    ENV S3_ACCESS_KEY_ID=${S3_ACCESS_KEY_ID}
    ENV S3_SECRET_ACCESS_KEY=${S3_SECRET_ACCESS_KEY}
    ENV S3_FORCE_PATH_STYLE=${S3_FORCE_PATH_STYLE}
    ENV ACTIVATED_PROVIDERS=${ACTIVATED_PROVIDERS}
    COPY . .
    RUN bun run build
    
    FROM oven/bun:1 AS runtime
    WORKDIR /app
    ARG PUBLIC_WEBAPP_VERSION=dev-local
    ARG PUBLIC_WEBAPP_GIT_TAG=
    ARG PUBLIC_WEBAPP_COMMIT_SHA=
    ARG PUBLIC_WEBAPP_RELEASED_AT=
    COPY --from=builder /app/build build/
    COPY --from=deps /app/node_modules node_modules/
    COPY --from=builder /app/package.json ./
    COPY --from=builder /app/drizzle.config.ts ./
    COPY --from=builder /app/drizzle ./drizzle
    COPY --from=builder /app/scripts ./scripts
    COPY --from=builder /app/src/lib/server/config/infrastructure.shared.js ./src/lib/server/config/infrastructure.shared.js
    COPY --from=builder /app/src/lib/server/infrastructure/db/schema.ts ./src/lib/server/infrastructure/db/schema.ts
    EXPOSE 3000
    ENV NODE_ENV=production
    ENV PUBLIC_WEBAPP_VERSION=${PUBLIC_WEBAPP_VERSION}
    ENV PUBLIC_WEBAPP_GIT_TAG=${PUBLIC_WEBAPP_GIT_TAG}
    ENV PUBLIC_WEBAPP_COMMIT_SHA=${PUBLIC_WEBAPP_COMMIT_SHA}
    ENV PUBLIC_WEBAPP_RELEASED_AT=${PUBLIC_WEBAPP_RELEASED_AT}
    CMD bun run db:migrate && bun ./build

Click **"Commit changes"**.

# Part 3 – Turso (Database)

1. Go to [turso.tech](https://turso.tech) → **"Start for free"** → sign in with GitHub
2. Click **"Create Database"**
3. Click on your new database → **"Connect"** → **"Generate Token"**
4. Save these two values:

    LIBSQL_URL        = libsql://sake-db-YOURNAME.turso.io
   
    LIBSQL_AUTH_TOKEN = eyJ...

# Part 4 – Backblaze B2 (File Storage)

1. Go to [backblaze.com](https://www.backblaze.com/b2/cloud-storage.html) → **"Sign Up for Free"** (no credit card needed)
2. In the dashboard → **"Buckets"** → **"Create a Bucket"**
3. Note your **endpoint URL** shown on the bucket page, e.g.:
4. Go to **"App Keys"** → **"Add a New Application Key"**
5. Save the 3 values immediately (shown only once!):

# Part 5 – Render (Deploy the App)

1. Go to [render.com](https://render.com) → **"Get Started for Free"** → sign in with GitHub
2. **"New +"** → **"Web Service"**
3. Select your forked Sake repository → **"Connect"**
4. Configure the service:
   ```Name:       sake
   Region:     Closest Place To your Region
   Branch:     master
   Root Dir:   sake
   Runtime:    Docker
10. Scroll down to **"Environment Variables"** and add all of these:

|Key|Value|
|:-|:-|
|LIBSQL\_URL|libsql://sake-db-YOURNAME.turso.io|
|LIBSQL\_AUTH\_TOKEN|your Turso token(eyJhbGciOiJIUzI1NiIsInR5cCI6IkpX...)|
|S3\_ENDPOINT|[https://s3.eu-central-003.backblazeb2.com](https://s3.eu-central-003.backblazeb2.com)|
|S3\_REGION|eu-central-003|
|S3\_BUCKET|sake-storage|
|S3\_ACCESS\_KEY\_ID|your Backblaze Key ID(0065d4f1a2b....)|
|S3\_SECRET\_ACCESS\_KEY|your Backblaze Secret Key(Kp8fG7hJ6l9qR3sT2uV0wXyZ1a....)|
|S3\_FORCE\_PATH\_STYLE|true|
|ACTIVATED\_PROVIDERS|openlibrary,gutenberg,zlib|
|PORT|10000|

6. Click **"Create Web Service"**

7. Wait 3–5 minutes for the build process to complete.

8. Once you see **"Live"** at the top, your app is running at [`https://sake-YOURNAME.onrender.com`](https://sake-YOURNAME.onrender.com)

# Part 6 – Create Your Account

1. Open your Sake URL in the browser
2. You should see a register screen - register
3. This is your account — keep these credentials safe

# Part 7 – KOReader Plugin Setup

1. Copy the `sake.koplugin` folder from your fork into KOReader's plugins directory
2. In KOReader: **Settings → More Tools → Sake**
3. Set your Sake URL (e.g. `https://sake-YOURNAME.onrender.com`)
4. Enter your username and password
5. Press **"Login and fetch device key"**
6. Press **"Sync books"** to download your library

# Important Notes

# Updates

Since you forked your own repository, you’ll need to update it whenever the original repository is updated. You’ll see a message at the top of your repo saying: ‘This branch is … commits ahead and … behind.’ If there is a number before ‘behind,’ click ‘Sync’ and then ‘Merge’ (**not ‘Discard commits’!**).

# Free Tier Sleep

Render's free tier pauses the app after 15 minutes of inactivity. The first request after a pause takes \~2-3+ minutes to wake up. This is normal and does not trigger a redeploy.

# Free Tier Limits

|Service|Free Limit|
|:-|:-|
|Render|750 hours/month|
|Turso|5 GB, 500M row reads/month|
|Backblaze B2|10 GB storage|
