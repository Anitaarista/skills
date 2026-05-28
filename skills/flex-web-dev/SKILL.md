---
name: flex-web-dev
description: "Flexible web development without framework lock-in. Build websites and web apps using ANY technology: pure HTML/CSS/JS (no build tools), React + Vite, Vue.js, Svelte, Astro, Express.js backend, or any other web framework. Use this skill whenever the user wants to build a website, web app, landing page, SPA, dashboard, e-commerce site, blog, portfolio, or any web-based project — even if they don't specify a framework. This skill intentionally avoids Next.js lock-in and gives the user full freedom to choose their tech stack. If the user explicitly requests Next.js, use the fullstack-dev skill instead. Otherwise, always use this skill for web development."
argument-hint: "Describe the website or web app you want to build, and optionally specify the tech stack"
version: 1.0.0
---

# Flexible Web Development Skill

Build websites and web applications with complete framework freedom. This skill supports **any** web technology the user chooses, with sensible defaults for common patterns.

## Core Philosophy

The user is in control. Ask about their technology preferences when not specified, but don't over-ask — provide a good default recommendation and let them confirm or change it. The goal is to build what the user wants, with the tools the user prefers (or the best tool for the job if they don't have a preference).

## Environment

You are running inside a Kubernetes sandbox pod. Key facts:

- **Node.js**: v24.14.1 available at `/usr/bin/node`
- **Python**: 3.12.13 available at `/home/z/.venv/bin/python3`
- **npm**: v11.11.1 available
- **GCC**: available for native modules
- **Working directory**: `/home/z/my-project/`
- **Persistent storage**: `/home/z/my-project/upload/` (OSS mount, 16EB, survives restarts)
- **Ephemeral storage**: Everything else is lost on pod restart
- **Available ports**: Port 81 is served by Caddy reverse proxy (mapped to gateway)
- **Internal service**: Port 12600 (ZAI service, do not use)
- **No Docker** available in this environment
- **AI SDK**: `z-ai-web-dev-sdk` is installed — use it in backend code for AI features

### Serving the Application

For serving web applications in this sandbox:

- **Static sites** (HTML/CSS/JS, or built output from Vite/etc): Use `npx serve` or a simple Python `http.server` on port 3000, then configure Caddy to proxy
- **Dev servers** (Vite, Next.js dev, etc): Run on port 3000
- **Backend servers** (Express, FastAPI, etc): Run on port 3001 or any free port

The Caddy reverse proxy on port 81 handles routing to the outside world. For development, just tell the user the app is running and provide the sandbox preview URL.

## Tech Stack Selection

When the user doesn't specify a technology, recommend based on the project type:

| Project Type | Recommended Stack | Why |
|---|---|---|
| Landing page / Portfolio | HTML + CSS + JS (vanilla) | Zero dependencies, fast, no build step |
| Single page app (SPA) | React + Vite + TypeScript | Most ecosystem, great DX |
| Content-focused site / Blog | Astro | Built-in content collections, islands architecture |
| Complex dashboard | React + Vite + Tailwind | Rich component ecosystem |
| E-commerce | React + Vite or Vue + Vite | Good ecosystem for shopping patterns |
| Real-time app | React + Vite + Socket.io | WebSocket support |
| Progressive web app | React + Vite + PWA plugin | Offline support |
| Minimalist / retro | HTML + CSS + JS (vanilla) | Full control, no overhead |

**Always ask the user** which tech stack they prefer if they haven't specified. Present options briefly and let them choose. Don't spend more than one message on this — if they say "whatever you think is best" or similar, pick the recommended stack and proceed.

## Development Workflow

### Phase 1: Setup & Scaffolding

1. Create the project in `/home/z/my-project/` (the working directory)
2. Initialize the project with the chosen framework's standard tooling
3. For vanilla projects, just create the file structure directly
4. Install dependencies with `npm install`
5. Verify the setup works before moving on

### Phase 2: Build

1. Implement the user's requirements iteratively
2. Use component-based architecture for framework projects
3. Use modular file organization for vanilla projects
4. Follow the framework's best practices and conventions
5. Test frequently during development

### Phase 3: Serve & Preview

1. Start the development server
2. Verify the application works correctly
3. Provide the user with the preview URL
4. Make any final adjustments based on testing

## Framework-Specific Guides

For detailed setup and patterns for each framework, read the corresponding reference file:

- `references/vanilla-html.md` — Pure HTML/CSS/JS websites (no build tools)
- `references/react-vite.md` — React + Vite + TypeScript
- `references/vue-vite.md` — Vue.js 3 + Vite
- `references/svelte-kit.md` — SvelteKit
- `references/astro.md` — Astro static site generator
- `references/express-backend.md` — Express.js backend server

**When to read**: Read only the reference file for the chosen framework. Don't read all of them — that wastes context.

## Design & Styling Guidelines

### Default Styling Approach
- **CSS**: Use CSS custom properties (variables) for theming
- **Utility-first**: Tailwind CSS is great when the user wants rapid development
- **Component libraries**: shadcn/ui (React), Headless UI, Radix UI — recommend when appropriate
- **Custom CSS**: For vanilla projects or when the user wants full design control

### Design Principles (Regardless of Framework)
- **Mobile-first responsive** — Design for small screens first, scale up
- **Accessibility (a11y)** — Semantic HTML, ARIA labels, keyboard navigation, WCAG AA minimum
- **Performance** — Lazy loading, image optimization, code splitting
- **Dark mode support** — Include unless the user explicitly says otherwise
- **Smooth animations** — Use CSS transitions/animations, respect `prefers-reduced-motion`

### Color & Typography
- Avoid default blue/indigo as primary unless user requests it
- Choose distinctive color palettes that match the project's brand
- Use a maximum of 2-3 brand colors + neutral grays
- Typography: Use Google Fonts or system font stack, avoid overused combinations

### Layout Patterns
- Consistent spacing scale (4px or 8px base)
- Container widths with max-width constraints
- Proper heading hierarchy (h1 → h2 → h3, not skipping levels)
- Sticky navigation on scroll (for content-heavy pages)

## AI Features Integration

When building features that need AI capabilities (chat, image generation, etc), use the `z-ai-web-dev-sdk` package:

```javascript
import ZAI from 'z-ai-web-dev-sdk';
const zai = await ZAI.create();
const completion = await zai.chat.completions.create({
  messages: [{ role: 'user', content: 'Hello' }]
});
```

**IMPORTANT**: Use `z-ai-web-dev-sdk` ONLY in backend/server code, never in client-side browser code.

## Important Rules

1. **No framework lock-in** — The user can use any technology. Never force Next.js unless explicitly requested.
2. **Ask before assuming** — When the user doesn't specify tech, recommend but let them confirm.
3. **Save to /home/z/my-project/** — All project files go in the working directory.
4. **Persistent files to /home/z/my-project/upload/** — Any files the user needs to survive pod restarts.
5. **Use npm, not bun** — In this environment, `npm` is the reliable package manager.
6. **Don't run build commands unnecessarily** — During development, use dev servers. Only build for production.
7. **Test before presenting** — Always verify the application actually works before telling the user it's ready.
8. **Mobile-first** — Always make responsive designs that work well on all screen sizes.
9. **Clean code** — Write well-structured, documented code. The user may want to maintain it.
