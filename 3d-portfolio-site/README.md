# 3D Portfolio Site — Build & Hosting Notes

A personal portfolio site for JudeTheTaken Automations, built to showcase AI
automation projects (n8n workflows, agents and integrations) with an
interactive 3D hero section.

**Live site:** https://judethetaken-portfolio.vercel.app/
**Source:** https://github.com/Judeintechwastaken/3d-portfolio

## Stack

- **Next.js** (App Router) — React framework, server routes
- **TypeScript**
- **Tailwind CSS** — styling
- **Framer Motion** — scroll and hover animations
- **React Three Fiber + drei** (Three.js) — the 3D wireframe sphere
- **Vercel** — hosting and CI/CD
- **GitHub** — version control, connected to Vercel for auto-deploy

## What the site includes

- A 3D wireframe sphere in the hero section that rotates and drifts toward
  the cursor / touch position
- Scroll-triggered fade-and-slide animations on each section, built with the
  browser's `IntersectionObserver` API
- A project showcase pulling from real automation builds, each card linking
  out to its GitHub repo
- A contact section

## How it was built

1. **Scaffolded the project locally** with `create-next-app`, using
   TypeScript, Tailwind and the App Router.
2. **Built the 3D scene** in `app/page.tsx` with `@react-three/fiber` and
   `@react-three/drei`: an icosahedron mesh with a wireframe material,
   ambient and point lighting, and `Sparkles` for background particles.
   `OrbitControls` gives it a slow auto-rotation.
3. **Made the sphere reactive to the cursor** by tracking `pointermove`
   events on the window and lerping the sphere's group position/rotation
   toward the pointer position each frame inside `useFrame`.
4. **Added scroll reveals** by tagging each section with a `.reveal` class,
   observing them with an `IntersectionObserver`, and toggling an
   `.in-view` class that triggers a CSS opacity/transform transition
   defined in `globals.css`.
5. **Built the project cards** as data objects (title, type, summary, tags,
   GitHub link) rendered into `<a>` cards, so adding a new project is a
   one-line edit rather than a new component.
6. **Version control:** initialized a Git repo, added a `.gitignore`
   (`node_modules`, `.next`, `.env*`, `.vercel`), committed, and pushed to a
   new GitHub repository (`3d-portfolio`).
7. **Deployment:** connected the GitHub repo to Vercel via
   "Import Git Repository." Vercel auto-detected the Next.js framework and
   deployed with default settings. Every push to `main` triggers an
   automatic rebuild and redeploy.
8. **Repo housekeeping:** added a description, the live URL and topic tags
   (`nextjs`, `threejs`, `ai-automation`, `n8n`, `portfolio`) to the GitHub
   repo's About section, and pinned the repo to the GitHub profile.

## Local development

```bash
npm install --legacy-peer-deps
npm run dev
```

Visit `http://localhost:3000`.

## Deploying changes

```bash
git add .
git commit -m "Describe the change"
git push
```

Vercel picks up the push automatically and redeploys within about a minute.

## Notes / lessons learned

- Vercel deployments only reflect what's pushed to the connected repo and
  branch (`main`) — pushing to a different repo (e.g. a separate projects
  repo) has no effect on the live site.
- Keep secrets (like webhook URLs) in `.env.local` (git-ignored) and mirror
  them in Vercel's Environment Variables settings rather than hardcoding
  them in the source.
- `prefers-reduced-motion` is respected for the scroll-reveal animations,
  so they degrade gracefully for users who've disabled motion.
