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

# Part 1. – Turso (Database)

1. Go to [turso.tech](https://turso.tech) → **"Start for free"** → sign in with GitHub
2. Click **"Create Database"**
3. Click on your new database → **"Connect"** → **"Generate Token"**
4. Save these two values:

    LIBSQL_URL        = libsql://sake-db-YOURNAME.turso.io
   
    LIBSQL_AUTH_TOKEN = eyJ...

# Part 2 – Backblaze B2 (File Storage)

1. Go to [backblaze.com](https://www.backblaze.com/b2/cloud-storage.html) → **"Sign Up for Free"** (no credit card needed)
2. In the dashboard → **"Buckets"** → **"Create a Bucket"**
3. Note your **endpoint URL** shown on the bucket page, e.g.:
4. Go to **"App Keys"** → **"Add a New Application Key"**
5. Save the 3 values immediately (shown only once!):

# Part 3 – Render (Deploy the App)

1. Go to [render.com](https://render.com) → **"Get Started for Free"** → sign in with GitHub
2. **"New +"** → **"Web Service"**
3. Go to public repo and paste this link: https://github.com/ThePixelPro366/Sake/
5. Configure the service:
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
|S3\_ENDPOINT|Your endpoint url(fpund in the bucket overview|
|S3\_REGION|your region|
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

The repo you cloned is my fork and I will try my best to Update everything as soon as possible 

# Free Tier Sleep

Render's free tier pauses the app after 15 minutes of inactivity. The first request after a pause takes \~30 seconds to wake up. This is normal and does not trigger a redeploy.

# Free Tier Limits

|Service|Free Limit|
|:-|:-|
|Render|750 hours/month|
|Turso|5 GB, 500M row reads/month|
|Backblaze B2|10 GB storage|
