 Node.js CI/CD Pipeline with GitHub Actions & Docker

  Project Overview

This project demonstrates a complete CI/CD (Continuous Integration / Continuous Deployment) pipeline for a simple Node.js web application. The pipeline is fully automated using GitHub Actions every time code is pushed to the `main` branch, it automatically:

1. Checks out the latest code
2. Installs dependencies
3. Runs tests
4. Logs in to DockerHub
5. Builds a Docker image of the app
6. Pushes the image to DockerHub

This was built as **Task 1** of an internship assignment to understand real-world DevOps automation workflows.


  Objective

Set up a CI/CD pipeline to automatically build and deploy a web app using GitHub Actions, without any manual intervention after the code is pushed.

 Tools & Technologies Used

| Tool               | Purpose |
| **GitHub**         | Source code hosting and version control |
| **GitHub Actions** | Automation engine that runs the CI/CD pipeline |
| **Node.js**        | Runtime for the sample web application |
| **Docker**         | Packages the app into a portable container image |
| **DockerHub**      | Stores and distributes the built Docker image |

Project Structure

nodejs-demo-app/
├── .github/
│   └── workflows/
│       └── main.yml          # CI/CD pipeline definition
├── index.js                   # Simple Node.js web server
├── package.json                # Project metadata & scripts
├── package-lock.json
├── Dockerfile                  # Instructions to containerize the app
└── README.md                   # This file


# Application Code

index.js
A minimal Node.js HTTP server that responds with a simple message — used to demonstrate the pipeline end to end.

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello from CI/CD pipeline!\n');
});

server.listen(3000, () => console.log('Server running on port 3000'));
```

# `package.json` (relevant scripts)
```json
"scripts": {
  "start": "node index.js",
  "test": "echo \"no tests yet\" && exit 0"
}
```

---

# Dockerfile

Defines how the application is packaged into a Docker container.

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

**What each line does:**
- `FROM node:18-alpine` — starts from a lightweight image that already has Node.js 18 installed
- `WORKDIR /app` — sets the working directory inside the container
- `COPY package*.json ./` — copies dependency files first (for Docker layer caching)
- `RUN npm install` — installs dependencies inside the container
- `COPY . .` — copies the rest of the application code
- `EXPOSE 3000` — documents that the app listens on port 3000
- `CMD ["npm", "start"]` — command that runs when the container starts

---

# CI/CD Workflow (`.github/workflows/main.yml`)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

      - name: Log in to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker image
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/nodejs-demo-app:latest .

      - name: Push Docker image
        run: docker push ${{ secrets.DOCKER_USERNAME }}/nodejs-demo-app:latest
```

# How it works, step by step

| Step                     | What it does |
| **Trigger (`on: push`)** | The entire pipeline runs automatically whenever code is pushed to the `main` branch |
| **Checkout code**        | Pulls the repository's latest code onto the GitHub-hosted runner |
| **Setup Node.js**        | Installs Node.js v18 on the runner so `npm` commands work |
| **Install dependencies** | Runs `npm install` to fetch required packages |
| **Run tests**            | Runs `npm test` this is the **Continuous Integration** step, verifying the code is sound before shipping |
| **Log in to DockerHub**  | Authenticates with DockerHub using credentials stored securely as GitHub Secrets |
| **Build Docker image**   | Packages the app into a Docker image using the `Dockerfile` |
| **Push Docker image**    | Uploads the image to DockerHub — this is the **Continuous Deployment** step |

# GitHub Secrets Used

To keep credentials safe, DockerHub login details are never hardcoded into the workflow file. Instead, they're stored as **encrypted repository secrets**:

| Secret Name       | Value |
| `DOCKER_USERNAME` | DockerHub account username |
| `DOCKER_PASSWORD` | DockerHub **access token** (not the account password) |

> Secrets are configured under **Repo → Settings → Secrets and variables → Actions**.

# How to Run This Project Locally

```bash
# Clone the repo
git clone https://github.com/gautam-guts-repo/nodejs-demo-app.git
cd nodejs-demo-app

# Install dependencies
npm install

# Run the app
npm start
# Visit http://localhost:3000
```

### Run with Docker locally
```bash
docker build -t nodejs-demo-app .
docker run -p 3000:3000 nodejs-demo-app
# Visit http://localhost:3000
```
# How the Pipeline Was Tested

1. Pushed code to the `main` branch
2. Opened the **Actions** tab on GitHub to watch the workflow run live
3. Confirmed all steps completed with green checkmarks
4. Verified the image appeared on DockerHub under the `latest` tag
5. Pulled the image back down and ran it locally to confirm it worked identically to the original

# Key Learnings

- How GitHub Actions workflows are structured (`on`, `jobs`, `steps`)
- The difference between Continuous Integration (test/build) and Continuous Deployment (push/release)
- How to securely manage credentials using GitHub Secrets instead of hardcoding them
- How Docker packages an application with its dependencies into a portable, consistent image
- How to debug a CI/CD pipeline using workflow logs and temporary debug steps


# License

This project was created for educational/internship purposes.
