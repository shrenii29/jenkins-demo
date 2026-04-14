# Jenkins Complete Study Guide
### From Scratch to Automated CI/CD Pipeline — Everything You Need to Know

---

## Table of Contents
1. [What is Jenkins?](#1-what-is-jenkins)
2. [Key Concepts & Terminology](#2-key-concepts--terminology)
3. [Step A — Create a GitHub Repository](#step-a--create-a-github-repository)
4. [Step B — Install and Launch Jenkins](#step-b--install-and-launch-jenkins)
5. [Step C — Install Required Plugins](#step-c--install-required-plugins)
6. [Step D — Create a New Pipeline Job](#step-d--create-a-new-pipeline-job)
7. [Step E — Configure Pipeline Script from SCM](#step-e--configure-pipeline-script-from-scm)
8. [Step F — Add a Jenkinsfile to the Repository](#step-f--add-a-jenkinsfile-to-the-repository)
9. [Step G — Configure Build Triggers](#step-g--configure-build-triggers)
10. [Step H — Push Code Changes to GitHub](#step-h--push-code-changes-to-github)
11. [Step I — Verify Automatic Build Trigger](#step-i--verify-automatic-build-trigger)
12. [Alternatives & Comparisons](#12-alternatives--comparisons)
13. [Frequently Asked Exam Questions](#13-frequently-asked-exam-questions)

---

## 1. What is Jenkins?

### Definition
Jenkins is a **free, open-source automation server** written in Java. It is used to automate the parts of software development related to building, testing, and deploying code. This practice is known as **CI/CD** — Continuous Integration and Continuous Deployment/Delivery.

### Why Do We Need Jenkins?
Imagine a team of 10 developers all pushing code to GitHub every day. Without Jenkins:
- Someone has to **manually** run tests every time code changes
- Someone has to **manually** build the project
- Someone has to **manually** deploy it to the server
- Human error is very likely

With Jenkins:
- Every code push is **automatically** detected
- Tests run **automatically**
- The project builds **automatically**
- Deployment happens **automatically**
- If anything fails, the team is **notified immediately**

### The Big Picture — How Jenkins Works

```
Developer pushes code to GitHub
            ↓
Jenkins detects the new code (via Poll SCM or Webhook)
            ↓
Jenkins reads the Jenkinsfile in the repository
            ↓
Jenkins runs each Stage one by one:
    Stage 1: Clone → Stage 2: Build → Stage 3: Test → Stage 4: Deploy
            ↓
Jenkins shows SUCCESS ✅ or FAILURE ❌ on the dashboard
            ↓
Team is notified of the result
```

### Jenkins Key Facts
| Property | Detail |
|---|---|
| Created by | Kohsuke Kawaguchi (2011) |
| Language | Java |
| License | Open Source (MIT) |
| Default Port | 8080 |
| Config File | Jenkinsfile (Groovy DSL) |
| Type | CI/CD Automation Server |

---

## 2. Key Concepts & Terminology

### CI/CD
- **CI (Continuous Integration):** Every time code is pushed, it is automatically integrated, built, and tested. The goal is to catch bugs early.
- **CD (Continuous Delivery):** After CI, the code is automatically prepared for release to production.
- **CD (Continuous Deployment):** The code is automatically deployed to production without manual intervention.

### Pipeline
A **Pipeline** is a series of automated steps that Jenkins runs in order. Think of it as an assembly line in a factory — each station does one job and passes the product to the next station.

### Jenkinsfile
The **Jenkinsfile** is a plain text file that defines all the stages of your pipeline. It is written in **Groovy DSL** (Domain Specific Language) and lives inside your GitHub repository. Because it lives inside the repo, it is version controlled just like your code — this is called **Pipeline as Code**.

### SCM (Source Code Management)
SCM refers to tools used to manage and track changes to source code. **GitHub** is an SCM. When Jenkins says "Pipeline script from SCM", it means: go to the GitHub repository and fetch the Jenkinsfile from there.

### Agent
The **agent** is the machine that actually runs the pipeline. `agent any` means Jenkins can use any available machine to run the pipeline.

### Stage
A **Stage** is a named phase of the pipeline. Examples: Build, Test, Deploy. Stages appear as visual blocks in the Jenkins dashboard so you can see exactly which step failed.

### Step
A **Step** is a single task inside a Stage. For example, `echo 'Hello'` is a step that prints text to the console.

### Build
A **Build** is one complete run of the pipeline. Every time Jenkins runs your pipeline, it creates a new build with a build number (#1, #2, #3...).

### Build Trigger
A **Build Trigger** is the event or condition that causes Jenkins to start a new build automatically. Examples: Poll SCM, GitHub Webhook, manual trigger.

### Node / Executor
The **Node** is the Jenkins server itself (or a connected agent machine). The **Executor** is a thread/slot on that node that runs one build at a time.

---

## Step A — Create a GitHub Repository

### What is GitHub?
GitHub is a cloud-based platform for hosting code using **Git** (a version control system). It allows multiple developers to collaborate on the same codebase. Jenkins integrates with GitHub to watch for code changes.

### Why Do We Need a GitHub Repository?
Jenkins needs a place to fetch your code and your Jenkinsfile from. GitHub serves as the **single source of truth** for your code. Every time you push changes to GitHub, Jenkins can detect them and trigger a build.

### Steps to Create the Repository

1. Go to **github.com** and sign in
2. Click the **"+"** icon (top right) → **"New repository"**
3. Fill in:
   - **Repository name:** `jenkins-demo`
   - **Visibility:** Public *(Private repos need credentials configured in Jenkins)*
   - ✅ Check **"Add a README file"**
4. Click **"Create repository"**

### Upload Project Files

Create a file called `app.py` in the repository:

```python
print("Hello from Jenkins Pipeline!")
print("Build successful!")
```

### Why a Public Repository?
A **public** repository can be accessed by Jenkins without any authentication (no username/password needed). A **private** repository would require you to set up **credentials** in Jenkins (using a Personal Access Token or SSH key).

### What is README.md?
The `README.md` is a Markdown file that describes your project. It is displayed on the GitHub repository homepage. GitHub creates it automatically if you check the option during repo creation.

---

## Step B — Install and Launch Jenkins

### Prerequisites — Java

Jenkins is built on Java, so Java must be installed first.

**Why Java?**
Jenkins is a `.war` (Web Application Archive) file — a Java web application. The Java Runtime Environment (JRE) or JDK is needed to execute it.

**Check if Java is installed:**
```
java -version
```

**If not installed:**
- Download JDK 17 from: https://adoptium.net/
- Install and restart Command Prompt

### Installation Methods

#### Method 1 — Using .war file (our method)
The `.war` file is a standalone, portable version of Jenkins that runs without a full installation.

```
java -jar jenkins.war --httpPort=8080
```

**How to run it:**
1. Open Command Prompt in the folder containing `jenkins.war`
2. Run the command above
3. Keep the Command Prompt window open (closing it stops Jenkins)
4. Open browser → `http://localhost:8080`

**Advantage:** No installation required, works on any OS with Java  
**Disadvantage:** Must manually start it every time, stops when terminal closes

#### Method 2 — Using Windows .msi Installer
Download the `.msi` installer from jenkins.io, run it, and Jenkins installs as a **Windows Service** that starts automatically.

**Advantage:** Runs in background, starts with Windows, no manual start needed  
**Disadvantage:** More complex setup

#### Method 3 — Using Docker
```
docker run -p 8080:8080 jenkins/jenkins:lts
```
**Advantage:** Isolated, easy to remove, consistent environment  
**Disadvantage:** Requires Docker installed

### What is localhost:8080?
- **localhost** = your own computer (IP address 127.0.0.1)
- **8080** = the port number Jenkins listens on
- Together: `http://localhost:8080` means "open Jenkins running on my own machine on port 8080"

### Unlocking Jenkins — Initial Admin Password

The first time Jenkins starts, it creates a secret password to prevent unauthorized access.

**Location of password (for .war file):**
```
C:\Users\YourUsername\.jenkins\secrets\initialAdminPassword
```

Open this file with Notepad, copy the long alphanumeric string, and paste it into the browser.

**Why does this exist?**
Security — if Jenkins were publicly accessible, anyone could log in without this step. The initial password ensures only the person who installed Jenkins (who has file system access) can set it up.

### Plugin Installation During Setup

After entering the password, Jenkins offers two options:
- **Install suggested plugins** — Installs a standard set of commonly used plugins
- **Select plugins to install** — Choose manually

Select **"Install suggested plugins"** for a quick setup.

### Create Admin User

After plugins install, create your admin account:
- **Username:** admin
- **Password:** anything you'll remember
- **Email:** can be anything

This is the account you'll use to log in to Jenkins every time.

---

## Step C — Install Required Plugins

### What is a Jenkins Plugin?
A **plugin** is an add-on that extends Jenkins's functionality. Out of the box, Jenkins is minimal. Plugins add support for Git, GitHub, different build tools, notification systems, and much more.

Jenkins has over **1,800 plugins** available in the Jenkins Plugin Index.

### Where to Install Plugins
**Manage Jenkins** → **Plugins** → **Available plugins** tab → Search → Check → Install

### Required Plugins and Why We Need Each One

#### 1. Git Plugin
**What it does:** Allows Jenkins to clone (download) code from any Git repository (GitHub, GitLab, Bitbucket, etc.)

**Why we need it:** Without this plugin, Jenkins cannot connect to GitHub or fetch your code. It provides the "Git" option under SCM in job configuration.

**What it installs automatically:** Git Client Plugin (the underlying library that does the actual Git operations)

#### 2. GitHub Plugin
**What it does:** Provides deeper integration specifically with GitHub (not just any Git server). It enables:
- GitHub Webhook integration (GitHub can notify Jenkins of pushes)
- Build status reporting back to GitHub (the ✅/❌ you see on Pull Requests)
- Linking Jenkins builds to GitHub commits

**Why we need it:** While the Git plugin lets Jenkins pull code from GitHub, the GitHub plugin enables **two-way communication** — Jenkins can also report build results back to GitHub.

**What it installs automatically:** GitHub API Plugin (handles GitHub's REST API communication)

#### 3. Pipeline Plugin
**What it does:** Enables the Pipeline job type in Jenkins. It allows you to write your build process as code in a Jenkinsfile using Groovy DSL.

**Why we need it:** Without this plugin, you cannot create Pipeline jobs or use a Jenkinsfile. You would be limited to the older "Freestyle" job type which is less powerful and not version controlled.

**What it installs automatically:** Pipeline: Groovy, Pipeline: SCM Step, Pipeline: Nodes and Processes, and several other supporting pipeline libraries.

### Plugin Dependency Tree
When you install a plugin, Jenkins automatically installs its **dependencies** (other plugins it needs to function):

```
Git Plugin
  └── Git Client Plugin

GitHub Plugin
  └── GitHub API Plugin
  └── Git Plugin

Pipeline Plugin
  └── Pipeline: Groovy
  └── Pipeline: SCM Step
  └── Pipeline: Nodes and Processes
  └── Pipeline: Supporting APIs
```

### Updating Plugins
Go to **Manage Jenkins** → **Plugins** → **Updates** tab to see and install plugin updates.

---

## Step D — Create a New Pipeline Job

### What is a Jenkins Job?
A **Job** (also called a **Project**) is a configured task that Jenkins runs. It defines:
- Where to get the code from
- How to build it
- When to run
- What to do with the results

### Types of Jenkins Jobs

| Job Type | Description | When to Use |
|---|---|---|
| **Freestyle Project** | Simple, GUI-configured job | Simple tasks, legacy setups |
| **Pipeline** | Code-defined pipeline using Jenkinsfile | Modern CI/CD, recommended |
| **Multibranch Pipeline** | Automatically creates pipelines for each branch | Multiple feature branches |
| **Folder** | Organizes jobs into folders | Large organizations with many jobs |
| **Multi-configuration Project** | Runs same job with different configs | Cross-platform testing |

### Why Pipeline and Not Freestyle?
| Feature | Freestyle | Pipeline |
|---|---|---|
| Defined in | Jenkins GUI | Jenkinsfile (code) |
| Version controlled | ❌ No | ✅ Yes |
| Code review possible | ❌ No | ✅ Yes |
| Complex workflows | ❌ Limited | ✅ Full support |
| Parallel stages | ❌ No | ✅ Yes |
| Industry standard | ❌ Outdated | ✅ Yes |

### Steps to Create a Pipeline Job

1. Click **"New Item"** on the Jenkins dashboard (left sidebar)
2. Enter name: `my-first-pipeline`
3. Select **"Pipeline"**
4. Click **"OK"**

You are now on the **Job Configuration Page**.

---

## Step E — Configure Pipeline Script from SCM

### What Does "Pipeline Script from SCM" Mean?
This setting tells Jenkins **where to find the Jenkinsfile**.

There are two options under "Definition":

| Option | What it means |
|---|---|
| **Pipeline script** | You write the Jenkinsfile directly inside Jenkins GUI |
| **Pipeline script from SCM** | Jenkins fetches the Jenkinsfile from your GitHub repository |

**We always use "Pipeline script from SCM"** because:
- The Jenkinsfile lives with the code (Pipeline as Code)
- It is version controlled (you can see its history)
- The whole team can see and edit it
- No need to edit Jenkins settings when pipeline changes — just update the file

### Configuration Steps

1. Under **"Pipeline"** section → **"Definition"** dropdown → Select **"Pipeline script from SCM"**

2. **SCM** dropdown → Select **"Git"**

3. **Repository URL:**
   ```
   https://github.com/YOUR_USERNAME/jenkins-demo.git
   ```
   > Get this from GitHub → your repo → green "Code" button → HTTPS tab → copy URL

4. **Credentials:** Leave as "none" for public repositories

5. **Branch Specifier:** Change from `*/master` to `*/main`
   > GitHub changed the default branch name from `master` to `main` in 2020

6. **Script Path:** Leave as `Jenkinsfile`
   > This is the exact filename Jenkins will look for in your repository

7. Click **"Save"**

### What is a Branch?
A **branch** is a separate version of the codebase. `main` is the default/primary branch. Developers create feature branches to work on new features without affecting `main`.

### What Happens When Jenkins Runs?
1. Jenkins connects to the GitHub repository URL
2. Clones (downloads) the code from the `main` branch
3. Looks for a file named `Jenkinsfile` in the root of the repository
4. Reads and executes the stages defined in that Jenkinsfile

---

## Step F — Add a Jenkinsfile to the Repository

### What is a Jenkinsfile?
A **Jenkinsfile** is a plain text file written in **Groovy DSL** that defines your entire CI/CD pipeline as code. It must be named exactly `Jenkinsfile` (capital J, no file extension) and placed in the **root** of your repository.

### What is Groovy DSL?
- **Groovy** is a programming language that runs on the Java Virtual Machine (JVM)
- **DSL (Domain Specific Language)** means it's a specialized version designed specifically for defining Jenkins pipelines
- You don't need to know full Groovy — the Pipeline DSL has its own simple syntax

### Two Types of Pipeline Syntax

#### 1. Declarative Pipeline (what we use — recommended)
More structured, easier to read, has predefined sections:
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
    }
}
```

#### 2. Scripted Pipeline (older style)
More flexible but more complex, uses full Groovy:
```groovy
node {
    stage('Build') {
        echo 'Building...'
    }
}
```

**Always use Declarative Pipeline** — it is the modern standard.

### The Complete Jenkinsfile

```groovy
pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning the repository...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                echo 'Build step complete!'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                echo 'All tests passed!'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                echo 'Deployment complete!'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

### Line-by-Line Explanation

| Code | Meaning |
|---|---|
| `pipeline { }` | The root block — everything must be inside this |
| `agent any` | Run this pipeline on any available Jenkins node/machine |
| `stages { }` | Container that holds all stage blocks |
| `stage('Clone Repository') { }` | A named phase called "Clone Repository" |
| `steps { }` | The actual commands to execute inside this stage |
| `echo 'message'` | Prints a message to the console output (like print in Python) |
| `checkout scm` | Tells Jenkins to pull the code from the configured GitHub repo |
| `post { }` | Runs after all stages complete, regardless of success or failure |
| `success { }` | Runs only if the entire pipeline succeeded |
| `failure { }` | Runs only if any stage failed |

### agent Options

| Agent | Meaning |
|---|---|
| `agent any` | Run on any available node |
| `agent none` | No global agent; each stage defines its own |
| `agent { label 'linux' }` | Run on a node with the label "linux" |
| `agent { docker 'python:3' }` | Run inside a Docker container |

### post Conditions

| Condition | When it runs |
|---|---|
| `always` | Always runs, no matter what |
| `success` | Only if pipeline succeeded |
| `failure` | Only if pipeline failed |
| `unstable` | If pipeline is unstable (test failures but no crash) |
| `changed` | If result is different from the last build |

### How to Add Jenkinsfile to GitHub

1. Go to your GitHub repository
2. Click **"Add file"** → **"Create new file"**
3. In the filename box, type exactly: `Jenkinsfile`
4. Paste the Jenkinsfile content above
5. Click **"Commit changes"** → **"Commit changes"**

Your repository should now contain:
```
jenkins-demo/
├── README.md
├── app.py
└── Jenkinsfile
```

---

## Step G — Configure Build Triggers

### What is a Build Trigger?
A **Build Trigger** is a condition or event that tells Jenkins to automatically start a new build. Without a trigger, you would have to manually click "Build Now" every single time.

### Types of Build Triggers

#### 1. Poll SCM (what we use)
Jenkins **polls** (checks) the GitHub repository on a defined schedule. If it finds new commits since the last check, it triggers a build.

**Configuration:**
1. In job config → **"Build Triggers"** section
2. Check **"Poll SCM"**
3. Set Schedule to: `* * * * *`

**Cron Syntax Explained:**
```
* * * * *
│ │ │ │ │
│ │ │ │ └── Day of week  (0-7, 0 and 7 = Sunday)
│ │ │ └──── Month        (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour         (0-23)
└────────── Minute       (0-59)

* = every possible value
```

| Schedule | Meaning |
|---|---|
| `* * * * *` | Every minute |
| `H/5 * * * *` | Every 5 minutes |
| `H * * * *` | Every hour |
| `H 8 * * 1-5` | Every weekday at 8 AM |
| `H 2 * * *` | Every day at 2 AM |

**Advantage of Poll SCM:** Simple, works without any special network setup, works on localhost  
**Disadvantage:** Not instant (depends on poll interval), wastes resources by constantly checking

#### 2. GitHub Webhook (alternative)
Instead of Jenkins asking GitHub "any new commits?", GitHub **pushes** a notification to Jenkins the moment a commit is made.

**How it works:**
```
Developer pushes code
        ↓
GitHub sends HTTP POST request to Jenkins URL
        ↓
Jenkins receives it and immediately starts a build
```

**Configuration:**
- In GitHub → repo → Settings → Webhooks → Add webhook
- Payload URL: `http://YOUR_JENKINS_URL/github-webhook/`
- Content type: `application/json`
- Event: "Just the push event"

**Advantage:** Instant trigger, no polling overhead  
**Disadvantage:** Requires Jenkins to be publicly accessible (not possible on localhost without tools like ngrok)

#### 3. Build Periodically
Runs on a schedule regardless of code changes (like a cron job):
```
H 2 * * *
```
Builds every day at 2 AM whether code changed or not.

#### 4. Build After Other Projects
Triggers when another Jenkins job completes successfully. Used for chaining jobs together.

#### 5. Manual Trigger
Click **"Build Now"** in Jenkins dashboard. No automation — just for testing.

### Poll SCM vs GitHub Webhook Comparison

| Feature | Poll SCM | GitHub Webhook |
|---|---|---|
| How it works | Jenkins checks GitHub periodically | GitHub notifies Jenkins instantly |
| Speed | Delayed (up to poll interval) | Instant |
| Works on localhost | ✅ Yes | ❌ No (needs public URL) |
| Network overhead | High (constant polling) | Low (only when push happens) |
| Setup complexity | Simple | Moderate |
| Best for | Development/exam | Production |

---

## Step H — Push Code Changes to GitHub

### Why Push a Code Change?
We need to trigger the automation. Since we configured Poll SCM, Jenkins will only run if it detects a **new commit** in the repository. Pushing a change creates a new commit for Jenkins to detect.

### Making a Change via GitHub UI

1. Go to your GitHub repo
2. Click on `app.py`
3. Click the **pencil icon ✏️** (Edit this file)
4. Change the content:
```python
print("Hello from Jenkins Pipeline!")
print("Build successful!")
print("This is version 2 - triggering Jenkins!")
```
5. Click **"Commit changes"** → **"Commit changes"**

### What is a Commit?
A **commit** is a saved snapshot of your changes with a unique ID (SHA hash like `a1b2c3d4`). Every commit has:
- A **commit message** describing the change
- The **author** who made it
- A **timestamp**
- The **diff** (what changed)

### What is a Push?
- **Commit** = saving changes to your local Git history
- **Push** = uploading those committed changes to GitHub (the remote repository)

When using GitHub's web interface, committing and pushing happen in one step.

### Alternative — Using Git Command Line

If you have Git installed locally:
```bash
git clone https://github.com/YOUR_USERNAME/jenkins-demo.git
cd jenkins-demo
# make changes to files
git add .
git commit -m "Version 2 - trigger Jenkins build"
git push origin main
```

---

## Step I — Verify Automatic Build Trigger

### What to Look For

1. Go to **http://localhost:8080**
2. Click on your job **"my-first-pipeline"**
3. Wait up to 1 minute (Poll SCM interval)
4. A new build appears in **"Build History"** (bottom left) — e.g., **#1**
5. Click on **#1**
6. Click **"Console Output"** in the left sidebar

### Understanding Console Output

The Console Output shows exactly what Jenkins did, line by line:

```
Started by an SCM change
Obtained Jenkinsfile from git https://github.com/YOUR_USERNAME/jenkins-demo.git
[Pipeline] Start of Pipeline
[Pipeline] node
[Pipeline] stage (Clone Repository)
Cloning the repository...
[Pipeline] stage (Build)
Building the project...
Build step complete!
[Pipeline] stage (Test)
Running tests...
All tests passed!
[Pipeline] stage (Deploy)
Deploying application...
Deployment complete!
[Pipeline] End of Pipeline
Pipeline completed successfully!
Finished: SUCCESS
```

**"Started by an SCM change"** — This confirms Jenkins automatically detected your GitHub commit and triggered the build. This is the key line that proves the automation worked!

### Build Status Indicators

| Icon | Color | Meaning |
|---|---|---|
| ● | Blue | Success — all stages passed |
| ● | Red | Failure — at least one stage failed |
| ● | Yellow | Unstable — built but with issues (e.g., test failures) |
| ● | Grey | Not built yet / Aborted |
| ● | Blinking | Currently running |

### Pipeline Stage View

Jenkins shows a visual dashboard of your pipeline stages:

```
Clone Repository → Build → Test → Deploy
      ✅               ✅       ✅       ✅
     2s              1s      1s      1s
```

Each stage shows:
- Whether it passed ✅ or failed ❌
- How long it took
- You can click each stage to see its logs

### What if the Build Doesn't Trigger?

| Problem | Solution |
|---|---|
| Build not triggering after 1 minute | Click "Build Now" manually to test; check Console Output for errors |
| "Branch not found" error | Make sure Branch Specifier is `*/main` not `*/master` |
| "Jenkinsfile not found" | Check filename is exactly `Jenkinsfile` with capital J |
| Git connection error | Check Repository URL is correct and ends in `.git` |
| Plugin missing | Go to Manage Jenkins → Plugins and verify Git/Pipeline are installed |

---

## 12. Alternatives & Comparisons

### Alternatives to Jenkins

| Tool | Type | Key Difference |
|---|---|---|
| **GitHub Actions** | Cloud-based CI/CD | Built into GitHub, no separate server needed, YAML-based |
| **GitLab CI/CD** | Built into GitLab | Integrated with GitLab, YAML-based `.gitlab-ci.yml` |
| **CircleCI** | Cloud-based CI/CD | Fast, easy setup, cloud-hosted |
| **Travis CI** | Cloud-based CI/CD | Simple, popular for open source projects |
| **Azure DevOps** | Microsoft platform | Full DevOps suite, enterprise-focused |
| **TeamCity** | JetBrains product | Powerful, good IDE integration |
| **Bamboo** | Atlassian product | Integrates with Jira and Bitbucket |

### Why Choose Jenkins Over Alternatives?

| Jenkins Advantage | Explanation |
|---|---|
| Free & Open Source | No licensing costs |
| Self-hosted | Full control over your server and data |
| 1800+ Plugins | Integrates with almost any tool |
| Highly customizable | Can be configured for any workflow |
| Large community | Huge support community, lots of documentation |
| Language agnostic | Works with Java, Python, Node.js, anything |

### Alternatives to GitHub (SCM Tools)

| Tool | Description |
|---|---|
| **GitLab** | Full DevOps platform, can be self-hosted |
| **Bitbucket** | Atlassian's Git hosting, integrates with Jira |
| **Azure Repos** | Microsoft's Git hosting in Azure DevOps |
| **Gitea** | Lightweight, self-hosted Git service |

### Alternatives to Poll SCM

| Trigger | Description |
|---|---|
| **GitHub Webhook** | Instant trigger when GitHub pushes notification |
| **Build Periodically** | Time-based schedule regardless of code changes |
| **Manual (Build Now)** | Human clicks the button |
| **Remote API trigger** | Another system calls Jenkins API to start build |
| **Upstream job trigger** | Another Jenkins job triggers this one |

---

## 13. Frequently Asked Exam Questions

**Q: What is Jenkins?**
A: Jenkins is a free, open-source automation server used to implement CI/CD (Continuous Integration and Continuous Delivery/Deployment). It automatically builds, tests, and deploys code whenever changes are pushed to a repository.

**Q: What is CI/CD?**
A: CI (Continuous Integration) is the practice of automatically integrating and testing code changes from multiple developers. CD (Continuous Delivery/Deployment) extends this by automatically preparing or deploying code to production after successful integration.

**Q: What is a Jenkinsfile?**
A: A Jenkinsfile is a text file written in Groovy DSL that defines the pipeline stages and steps. It lives inside the code repository (Pipeline as Code), making the pipeline version-controlled and reviewable.

**Q: What is the difference between Poll SCM and GitHub Webhook?**
A: Poll SCM has Jenkins periodically check GitHub for changes on a schedule (e.g., every minute). GitHub Webhook has GitHub instantly notify Jenkins whenever a push occurs. Webhooks are faster and more efficient but require Jenkins to be publicly accessible.

**Q: Why do we use "Pipeline script from SCM" instead of "Pipeline script"?**
A: "Pipeline script from SCM" fetches the Jenkinsfile from the GitHub repository, making the pipeline code-versioned and part of the project. "Pipeline script" stores it only in Jenkins, making it harder to track changes and collaborate.

**Q: What plugins are required and why?**
A: Git Plugin — allows Jenkins to connect to Git repositories and clone code. GitHub Plugin — enables deeper GitHub integration including webhooks and status reporting. Pipeline Plugin — enables the Pipeline job type and Jenkinsfile support.

**Q: What is the default port for Jenkins?**
A: 8080. Jenkins can be started on a different port using `--httpPort=XXXX`.

**Q: What is `agent any` in a Jenkinsfile?**
A: It means the pipeline can run on any available Jenkins node/machine. Alternative agents include specific labeled nodes, Docker containers, or Kubernetes pods.

**Q: What does `checkout scm` do?**
A: It tells Jenkins to clone/pull the code from the Source Code Management system (GitHub) that is configured in the job settings.

**Q: What is the `post` block in a Jenkinsfile?**
A: The `post` block runs after all stages complete. It can contain conditions like `success`, `failure`, `always`, `unstable`, and `changed` to run different steps based on the pipeline result.

**Q: What is the difference between Declarative and Scripted Pipeline?**
A: Declarative Pipeline uses a structured, predefined syntax with `pipeline { }` as the root block — it is simpler and the modern recommended approach. Scripted Pipeline uses `node { }` as the root and full Groovy programming — it is more flexible but more complex.

**Q: What does the cron schedule `* * * * *` mean?**
A: It means "run every minute, every hour, every day, every month, every day of the week" — effectively triggering Jenkins to poll GitHub every single minute.

**Q: What is `localhost:8080`?**
A: `localhost` refers to your own machine (IP 127.0.0.1). `8080` is the port number Jenkins listens on. Together they form the address to access Jenkins running on your own computer.

**Q: Why do we need Java to run Jenkins?**
A: Jenkins is written in Java and packaged as a `.war` (Web Application Archive) file, which is a Java application format. The Java Runtime Environment (JRE) is required to execute it.

**Q: What is the difference between a Stage and a Step?**
A: A Stage is a named phase of the pipeline (e.g., Build, Test, Deploy) that groups related work. A Step is an individual command or action within a stage (e.g., `echo 'Building...'`, `sh 'python test.py'`).

---

## Quick Reference Cheat Sheet

```
Jenkins Flow:
GitHub Push → Poll SCM detects → Reads Jenkinsfile → Runs Stages → Shows Result

Key Files:
  Jenkinsfile     → Defines your pipeline (must be in repo root)
  app.py          → Your project code
  README.md       → Repository description

Key URLs:
  http://localhost:8080              → Jenkins dashboard
  http://localhost:8080/manage       → Manage Jenkins
  initialAdminPassword location      → C:\Users\USERNAME\.jenkins\secrets\

Cron Examples:
  * * * * *      → Every minute
  H/5 * * * *    → Every 5 minutes
  H 2 * * *      → Daily at 2 AM

Build Status:
  Blue   = Success
  Red    = Failure
  Yellow = Unstable
  Grey   = Not run
```

---

*Guide created for Jenkins Practical Exam Preparation*  
*Jenkins Version: 2.541.3 | Plugins: Git, GitHub, Pipeline*
