# GitHub Actions Workflow Copilot



A basic Angular app deployed to **GitHub Pages** via a multi-stage **GitHub Actions CI/CD pipeline**.

## 🌐 Live Demo

View the deployed app at: `https://rj010899-beep.github.io/Github_Action_Workflow_Copilot/`

## 🚀 CI/CD Pipeline Stages

The GitHub Actions workflow (`.github/workflows/ci-cd.yml`) runs on every push/PR to `main` and consists of 4 stages:

### Stage 1: Workspace INIT
- Checkout source code
- Setup Node.js 22
- Cache and install npm dependencies

### Stage 2: Automated Testing
- **Unit Tests** — Vitest unit tests via `npm test`
- **Code Review** — ESLint static analysis via `npm run lint`
- **Security** — Dependency vulnerability scan via `npm audit --audit-level=high`

### Stage 3: Build
- Production Angular build with correct `--base-href` for GitHub Pages
- Upload build artifact

### Stage 4: Deploy to GitHub Pages
- Deploy the build artifact to GitHub Pages (main branch only)

## 🛠️ Local Development

```bash
# Install dependencies
npm install

# Start dev server
npm start

# Run unit tests
npm test

# Run linter
npm run lint

# Production build
npm run build
```

## 📦 Tech Stack

- **Angular 21** — Frontend framework
- **TypeScript** — Language
- **SCSS** — Styling
- **Vitest** — Unit testing
- **ESLint + angular-eslint** — Code quality
- **GitHub Actions** — CI/CD
- **GitHub Pages** — Hosting
