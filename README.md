## Next.js App Router Course - Starter

This is the starter template for the Next.js App Router Course. It contains the starting code for the dashboard application.

For more information, see the [course curriculum](https://nextjs.org/learn) on the Next.js Website.

# running the SPA locally

1. initialize pnpm
```pnpm i```
2. run development server locally
```pnpm dev```

# Repository structure

This repository functions as both a standalone main repo and a Git submodule for broader project integration and tracking.

# Debug log

Following the tutorial blindly resulted in a failed deployment on Vercel. After some deep issue tracking, particularly in [this discussion](https://github.com/vercel/next.js/discussions/76822) and [this stack overflow thread](https://stackoverflow.com/questions/76710159/error-while-deploying-nextjs-app-to-vercel), it turns out the issue stemmed from a dependency conflict.

Fix:
- Switched from bcrypt to bcryptjs
- Adjusted route.ts in /app/seed/

This resolved the issue and allowed for a successful deployment.