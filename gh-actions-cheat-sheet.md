# GitHub Actions Cheat Sheet

A quick reference for beginner-to-intermediate GitHub Actions workflows.

---

## 1. What is GitHub Actions?

GitHub Actions is GitHub's automation system.

It can run workflows when events happen in a repository, such as:

- pushing code
- opening a pull request
- creating a comment
- running manually
- running on a schedule

Common uses:

- run tests
- build apps
- deploy apps
- build Docker images
- push to Docker Hub
- call webhooks
- send notifications

---

## 2. Basic Workflow Structure

Workflow files live in:

```text
.github/workflows/
```

Example:

```text
.github/workflows/ci.yml
```

Minimal workflow:

```yaml
name: CI

on: push

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Hello from GitHub Actions"
```

Main parts:

```text
name    = workflow name
on      = event that starts the workflow
jobs    = work to run
runs-on = machine/runner to use
steps   = commands/actions inside a job
```

---

## 3. Common `on:` Events

Run on every push:

```yaml
on: push
```

Run on pull requests:

```yaml
on: pull_request
```

Run manually from the GitHub UI:

```yaml
on: workflow_dispatch
```

Run on push to `main` only:

```yaml
on:
  push:
    branches:
      - main
```

Run on pull requests targeting `main`:

```yaml
on:
  pull_request:
    branches:
      - main
```

Run on issue or main PR conversation comments:

```yaml
on:
  issue_comment:
    types:
      - created
```

Run on inline PR code comments:

```yaml
on:
  pull_request_review_comment:
    types:
      - created
```

Run on a schedule:

```yaml
on:
  schedule:
    - cron: "0 9 * * *"
```

---

## 4. Manual Workflow

Useful for testing without pushing new commits.

```yaml
name: Manual Test

on: workflow_dispatch

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Manual workflow started"
```

Where to run it:

```text
Repository → Actions → select workflow → Run workflow
```

---

## 5. Jobs and Steps

One job:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Running tests"
```

Multiple steps:

```yaml
steps:
  - name: Step one
    run: echo "First"

  - name: Step two
    run: echo "Second"
```

Multiple jobs:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing"

  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building"
```

Make one job wait for another:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing"

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building after tests"
```

---

## 6. `uses` vs `run`

Use `uses` for premade actions:

```yaml
- uses: actions/checkout@v4
```

Use `run` for shell commands:

```yaml
- run: npm test
```

Example:

```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v4

  - name: Run tests
    run: npm test
```

---

## 7. Common Official Actions

Checkout repository:

```yaml
- uses: actions/checkout@v4
```

Set up Node.js:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 24
    cache: npm
```

Upload artifact:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-files
    path: dist/
```

Deploy GitHub Pages:

```yaml
- uses: actions/upload-pages-artifact@v3
  with:
    path: dist/

- uses: actions/deploy-pages@v4
```

Use GitHub API with JavaScript:

```yaml
- uses: actions/github-script@v7
  with:
    script: |
      console.log(context.repo)
```

---

## 8. Environment Variables

Workflow-level env:

```yaml
env:
  APP_ENV: production

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - run: echo "$APP_ENV"
```

Step-level env:

```yaml
steps:
  - name: Print value
    run: echo "$APP_ENV"
    env:
      APP_ENV: production
```

---

## 9. Repository Variables

Repository variables are for non-secret configuration.

Create them in:

```text
Repository → Settings → Secrets and variables → Actions → Variables
```

Example variable:

```text
APP_ENV=production
```

Use it:

```yaml
- run: echo "Environment: ${{ vars.APP_ENV }}"
```

Good for:

```text
APP_ENV
APP_NAME
PUBLIC_API_URL
DOCKER_IMAGE_NAME
```

---

## 10. Secrets

Secrets are for sensitive values.

Create them in:

```text
Repository → Settings → Secrets and variables → Actions → Secrets
```

Use them:

```yaml
- run: echo "Using secret"
  env:
    API_KEY: ${{ secrets.API_KEY }}
```

Do not print secrets directly.

Good for:

```text
API keys
tokens
passwords
Docker Hub tokens
Render deploy hooks
Discord webhooks
```

---

## 11. GitHub Context Values

Useful built-in values:

```yaml
${{ github.repository }}
${{ github.actor }}
${{ github.ref }}
${{ github.ref_name }}
${{ github.sha }}
${{ github.event_name }}
${{ github.run_number }}
```

Example:

```yaml
- name: Print info
  run: |
    echo "Repository: ${{ github.repository }}"
    echo "Actor: ${{ github.actor }}"
    echo "Branch: ${{ github.ref_name }}"
    echo "Commit: ${{ github.sha }}"
```

---

## 12. Conditions

Use `if:` to control when a step or job runs.

Run only on `main`:

```yaml
if: github.ref == 'refs/heads/main'
```

Run only on pull requests:

```yaml
if: github.event_name == 'pull_request'
```

Run only when previous steps succeeded:

```yaml
if: success()
```

Run only when something failed:

```yaml
if: failure()
```

Run no matter what:

```yaml
if: always()
```

Run only if workflow was cancelled:

```yaml
if: cancelled()
```

---

## 13. Success and Failure Example

```yaml
name: Conditions Demo

on: workflow_dispatch

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - name: Pass
        run: echo "This works"

      - name: Fail
        run: exit 1

      - name: Runs only if success
        if: success()
        run: echo "This will not run"

      - name: Runs only if failure
        if: failure()
        run: echo "Something failed"

      - name: Always runs
        if: always()
        run: echo "Workflow finished"
```

---

## 14. Multiline Scripts

Use `run: |` for multiple shell commands.

```yaml
- name: Script example
  run: |
    echo "Starting..."
    npm install
    npm test
    echo "Finished"
```

Check if a file exists:

```yaml
- name: Check Dockerfile
  run: |
    if [ -f Dockerfile ]; then
      echo "Dockerfile found"
    else
      echo "Dockerfile missing"
      exit 1
    fi
```

---

## 15. Node.js CI Workflow

```yaml
name: CI

on:
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 24
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build app
        run: npm run build
```

---

## 16. PR Comment Auto Reply

Replies to comments in the main PR conversation.

```yaml
name: PR Comment Auto Reply

on:
  issue_comment:
    types:
      - created

permissions:
  issues: write
  pull-requests: write

jobs:
  reply:
    if: ${{ github.event.issue.pull_request && github.event.comment.user.type != 'Bot' }}
    runs-on: ubuntu-latest

    steps:
      - name: Reply to comment
        uses: actions/github-script@v7
        with:
          script: |
            const commenter = context.payload.comment.user.login;

            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: `Thanks @${commenter} for your message on this PR.`
            });
```

---

## 17. Webhooks

A webhook is an HTTP request sent to another service.

Simple webhook:

```yaml
- name: Call webhook
  run: curl -X POST https://example.com/webhook
```

Webhook stored as a secret:

```yaml
- name: Call webhook
  run: curl -X POST "$WEBHOOK_URL"
  env:
    WEBHOOK_URL: ${{ secrets.WEBHOOK_URL }}
```

Webhook with JSON:

```yaml
- name: Send JSON webhook
  run: |
    curl -X POST "$WEBHOOK_URL"       -H "Content-Type: application/json"       -d '{"message":"Hello from GitHub Actions"}'
  env:
    WEBHOOK_URL: ${{ secrets.WEBHOOK_URL }}
```

Common webhook uses:

```text
Trigger Render deploy
Send Discord notification
Send Slack notification
Call your own backend API
Test with webhook.site
```

---

## 18. Docker Build Workflow

```yaml
name: Docker Build

on: workflow_dispatch

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t my-app .
```

---

## 19. Docker Hub Login and Push

Required secrets:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

Workflow:

```yaml
name: Docker Publish

on:
  push:
    branches:
      - main

env:
  IMAGE_NAME: ${{ secrets.DOCKERHUB_USERNAME }}/my-app

jobs:
  publish:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build image
        run: |
          docker build             -t $IMAGE_NAME:latest             -t $IMAGE_NAME:${{ github.sha }}             .

      - name: Push image
        run: |
          docker push $IMAGE_NAME:latest
          docker push $IMAGE_NAME:${{ github.sha }}
```

---

## 20. Render Deploy Hook

Required secret:

```text
RENDER_DEPLOY_HOOK
```

Step:

```yaml
- name: Trigger Render deploy
  run: curl -X POST "$RENDER_DEPLOY_HOOK"
  env:
    RENDER_DEPLOY_HOOK: ${{ secrets.RENDER_DEPLOY_HOOK }}
```

---

## 21. Full CI/CD Example

```yaml
name: Final DevOps Pipeline

on:
  workflow_dispatch:
  push:
    branches:
      - main

env:
  NODE_VERSION: 24
  IMAGE_NAME: ${{ secrets.DOCKERHUB_USERNAME }}/${{ vars.DOCKER_IMAGE_NAME }}

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Show workflow info
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"
          echo "Triggered by: ${{ github.actor }}"

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build app
        run: npm run build

      - name: Check Dockerfile exists
        run: |
          if [ -f Dockerfile ]; then
            echo "Dockerfile found"
          else
            echo "Dockerfile missing"
            exit 1
          fi

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Docker image
        run: |
          docker build             -t $IMAGE_NAME:latest             -t $IMAGE_NAME:${{ github.sha }}             .

      - name: Push Docker image
        run: |
          docker push $IMAGE_NAME:latest
          docker push $IMAGE_NAME:${{ github.sha }}

      - name: Trigger Render deployment
        run: curl -X POST "$RENDER_DEPLOY_HOOK"
        env:
          RENDER_DEPLOY_HOOK: ${{ secrets.RENDER_DEPLOY_HOOK }}

      - name: Send success notification
        if: success()
        run: |
          curl -X POST "$WEBHOOK_URL"             -H "Content-Type: application/json"             -d '{
              "status": "success",
              "message": "Deployment successful",
              "repository": "${{ github.repository }}",
              "branch": "${{ github.ref_name }}",
              "commit": "${{ github.sha }}"
            }'
        env:
          WEBHOOK_URL: ${{ secrets.WEBHOOK_URL }}

      - name: Send failure notification
        if: failure()
        run: |
          curl -X POST "$WEBHOOK_URL"             -H "Content-Type: application/json"             -d '{
              "status": "failure",
              "message": "Pipeline failed",
              "repository": "${{ github.repository }}",
              "branch": "${{ github.ref_name }}",
              "commit": "${{ github.sha }}"
            }'
        env:
          WEBHOOK_URL: ${{ secrets.WEBHOOK_URL }}
```

---

## 22. Artifacts

Artifacts are files saved from a workflow run.

Upload `dist/`:

```yaml
- name: Upload build artifact
  uses: actions/upload-artifact@v4
  with:
    name: app-build
    path: dist/
```

For Create React App:

```yaml
path: build/
```

Find artifacts here:

```text
Repository → Actions → workflow run → Artifacts
```

---

## 23. Permissions

Some workflows need explicit permissions.

Example for writing issue/PR comments:

```yaml
permissions:
  issues: write
  pull-requests: write
```

Example for GitHub Pages:

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

If you see:

```text
Resource not accessible by integration
```

it usually means the workflow token does not have enough permission.

---

## 24. Common Problems

### Workflow does not appear

Check file location:

```text
.github/workflows/name.yml
```

Check it is committed to GitHub.

---

### YAML error

Check indentation.

Wrong:

```yaml
jobs:
test:
runs-on: ubuntu-latest
```

Right:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
```

---

### `npm ci` fails

Make sure `package-lock.json` exists.

---

### Secret is empty

Check secret name spelling.

This is different:

```text
DOCKER_TOKEN
```

from:

```text
DOCKERHUB_TOKEN
```

---

### Permission error

Add permissions:

```yaml
permissions:
  contents: read
  issues: write
  pull-requests: write
```

Also check:

```text
Repository → Settings → Actions → General → Workflow permissions
```

---

## 25. Quick Mental Model

```text
on:
  When should this run?

jobs:
  What work should be done?

runs-on:
  What machine should run it?

steps:
  What commands/actions should run?

uses:
  Use someone else's action

run:
  Run my own command

env:
  Environment variables

vars:
  Repository configuration

secrets:
  Private values

if:
  Run only under certain conditions
```

---

## 26. CI/CD Mental Model

```text
CI = verify the code
```

Usually:

```text
install dependencies
run tests
build app
```

```text
CD = release the code
```

Usually:

```text
build Docker image
push image
deploy
notify
```

Full flow:

```text
Push to main
  ↓
GitHub Actions
  ↓
Test
  ↓
Build
  ↓
Docker image
  ↓
Docker Hub
  ↓
Render deploy hook
  ↓
Notification
```
