# Jenkins Learning — Day 1: From Zero to Your First Pipeline

> **Level:** Absolute beginner → DevOps beginner  
> **Goal:** Understand *why* Jenkins exists, what it does, and run your first Jenkins job and pipeline.  
> **Prerequisite:** Docker installed and running on Linux, macOS, or Windows.

## Table of Contents

1. [The Problem Before Jenkins](#1-the-problem-before-jenkins)
2. [What Is Jenkins?](#2-what-is-jenkins)
3. [Why Do We Need Jenkins?](#3-why-do-we-need-jenkins)
4. [What Are CI and CD?](#4-what-are-ci-and-cd)
5. [How Jenkins Works](#5-how-jenkins-works)
6. [Real-World Use Case](#6-real-world-use-case)
7. [Important Jenkins Terms](#7-important-jenkins-terms)
8. [Hands-on Lab: Install Jenkins with Docker](#8-hands-on-lab-install-jenkins-with-docker)
9. [Hands-on Lab: Your First Freestyle Job](#9-hands-on-lab-your-first-freestyle-job)
10. [Hands-on Lab: Your First Pipeline](#10-hands-on-lab-your-first-pipeline)
11. [Troubleshooting](#11-troubleshooting)
12. [Day 1 Practice Checklist](#12-day-1-practice-checklist)
13. [Learning Roadmap](#13-learning-roadmap)
14. [Interview Questions](#14-interview-questions)
15. [References](#15-references)

---

## 1. The Problem Before Jenkins

Imagine a food-delivery company with an online ordering application. A developer creates a new **Apply Coupon** feature.

Without automation, every release may require someone to:

1. Get the latest code from the repository.
2. Install dependencies.
3. Build the application.
4. Run tests.
5. Copy or deploy it to a server.
6. Verify that the application works.

If engineers repeat these steps manually many times per day, mistakes and delays become more likely. **Jenkins automates repeatable steps.**

## 2. What Is Jenkins?

**Jenkins is a free, open-source automation server** widely used to automate software build, test, and delivery workflows.

For a non-technical analogy, think of Jenkins as a reliable **workflow assistant**. You write down the instructions once; Jenkins runs them when requested or when a configured event occurs.

> Jenkins does not write your application by itself. It runs the tools and commands you configure.

## 3. Why Do We Need Jenkins?

- **Consistency:** The same defined steps run every time.
- **Speed:** Repetitive commands no longer need to be executed manually.
- **Feedback:** Developers can quickly discover build and test failures.
- **Traceability:** Builds have logs, status, and history.
- **Integration:** Jenkins works with Git, GitHub, Docker, test tools, cloud platforms, and Kubernetes.

## 4. What Are CI and CD?

### CI — Continuous Integration

Developers frequently integrate code changes. Automated checks (such as build and tests) help find issues early.

**Example:** Push code to GitHub → Jenkins checks out code → installs dependencies → runs tests.

### CD — Continuous Delivery / Continuous Deployment

- **Continuous Delivery:** Changes are tested and kept ready for release; production deployment may require approval.
- **Continuous Deployment:** Changes that pass configured checks are automatically released to production without manual approval.

## 5. How Jenkins Works

```text
Developer
    |
    | pushes code
    v
Git Repository (e.g., GitHub)
    |
    | webhook / polling / manual trigger
    v
Jenkins Controller
    |
    | schedules the work
    v
Jenkins Agent / Executor
    |
    +--> Checkout Source
    +--> Build
    +--> Test
    +--> Package
    +--> Deploy (if configured and authorized)
    |
    v
Build Status + Logs + Notifications
```

**Important:** A production pipeline usually adds security scans, approval gates, environment configuration, monitoring, and rollback procedures.

## 6. Real-World Use Case

A food-delivery company updates its order-service backend:

1. A developer opens a pull request with a bug fix.
2. Jenkins runs unit tests and other checks.
3. A reviewer approves and merges the change.
4. Jenkins builds a Docker image and publishes it to an authorized registry.
5. Jenkins deploys it to a staging environment.
6. After verification and approval, a production deployment occurs.
7. The team monitors the rollout and can roll back if necessary.

This is an **example workflow**—specific steps depend on the company's policies and infrastructure.

## 7. Important Jenkins Terms

| Term | Simple Meaning |
| --- | --- |
| Job | A task Jenkins is configured to run |
| Build | One run of a job |
| Pipeline | A sequence of automated steps |
| Stage | A named phase, such as Build or Test |
| Jenkinsfile | A file containing pipeline instructions |
| Controller | Main Jenkins service that schedules and manages work |
| Agent | Machine or container that executes jobs |
| Executor | Execution slot available for running a build |
| Plugin | Extension that adds functionality |
| Trigger | Event, schedule, or manual action that starts a build |
| Workspace | Directory used to perform a job's work |

---

## 8. Hands-on Lab: Install Jenkins with Docker

> **Local learning environment only.** Do not expose this tutorial instance directly to the public internet. For production, follow Jenkins' security, backup, and agent recommendations.

### Step 1 — Verify Docker

```bash
docker --version
docker info
```

If `docker info` fails, start Docker Desktop or the Docker daemon first.

### Step 2 — Pull the Official Jenkins LTS Image

```bash
docker pull jenkins/jenkins:lts
```

### Step 3 — Create Persistent Storage

```bash
docker volume create jenkins_home
```

### Step 4 — Start Jenkins

```bash
docker run -d \
  --name jenkins \
  -p 127.0.0.1:8080:8080 \
  -p 127.0.0.1:50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

**Explanation:**

- `-d`: Run in the background.
- `--name jenkins`: Name the container `jenkins`.
- `8080`: Jenkins web interface, bound to localhost for this lab.
- `50000`: Optional inbound-agent port, also bound to localhost.
- `jenkins_home`: Persistent Jenkins configuration and job data.

### Step 5 — Check Jenkins Status

```bash
docker ps
docker logs -f jenkins
```

Use **Ctrl+C** to stop following logs (the container keeps running).

### Step 6 — Unlock Jenkins

Open **http://localhost:8080** and retrieve the initial administrator password:

```bash
docker exec jenkins \
  cat /var/jenkins_home/secrets/initialAdminPassword
```

Then:

1. Paste the password into the setup screen.
2. Choose **Install suggested plugins**.
3. Create an administrator user.
4. Complete setup and open the dashboard.

**Security tip:** Do not share your admin password or paste it into public screenshots or repositories.

---

## 9. Hands-on Lab: Your First Freestyle Job

1. On the dashboard, click **New Item**.
2. Name it `my-first-job`.
3. Select **Freestyle project** and click **OK**.
4. Under **Build Steps**, select **Add build step → Execute shell**.
5. Paste these commands:

```bash
echo "Welcome to Jenkins!"
echo "I am learning CI/CD"
echo "Current date:"
date
echo "Current user:"
whoami
echo "Current directory:"
pwd
echo "My first Jenkins job completed!"
```

6. Click **Save → Build Now**.
7. Open **Build History → #1 → Console Output**.

Expected output includes:

```text
Welcome to Jenkins!
I am learning CI/CD
Current date:
...
Current user:
jenkins
Current directory:
/var/jenkins_home/workspace/my-first-job
My first Jenkins job completed!
Finished: SUCCESS
```

Exact dates, paths, and details depend on your environment.

### What Did We Learn?

Jenkins executed configured shell commands and saved the result as a build with logs.

---

## 10. Hands-on Lab: Your First Pipeline

1. Click **New Item**.
2. Enter `my-first-pipeline`.
3. Select **Pipeline** and click **OK**.
4. Scroll to **Pipeline → Definition: Pipeline script**.
5. Paste the following:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running application tests...'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying the application...'
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

6. Click **Save → Build Now**.
7. Check the stage results and **Console Output**.

### Understand the Pipeline

| Syntax | Meaning |
| --- | --- |
| `pipeline {}` | Defines the Pipeline |
| `agent any` | Run on any available suitable agent/executor |
| `stages {}` | Groups the stages |
| `stage('Build')` | Names a stage |
| `steps {}` | Commands/actions executed within that stage |
| `echo` | Prints a message |
| `post {}` | Runs status-dependent actions after the pipeline |

> **Note:** The Build, Test, and Deploy stages above **only print messages**. They do not yet build, test, or deploy a real application. We'll add genuine commands in later lessons.

### Bonus — Make a Stage Fail

Change the **Test** stage to:

```groovy
stage('Test') {
    steps {
        sh 'exit 1'
    }
}
```

Run it again. You should see **FAILURE** in the logs and the Deploy stage skipped. Change it back afterward.

---

## 11. Troubleshooting

### Jenkins webpage does not open

```bash
docker ps -a
docker logs --tail 100 jenkins
```

Confirm Docker is running and port `8080` is not used by another application. If necessary, recreate the container with a different host port, e.g. `-p 127.0.0.1:8081:8080`, then open `http://localhost:8081`.

### Container stopped

```bash
docker start jenkins
```

### Job remains queued

Check **Manage Jenkins → Nodes** and verify an available online agent/executor. Some Jenkins installations are configured with **zero executors on the controller**. In that case, add/configure a build agent rather than allowing work on a production controller.

### Check Jenkins container logs

```bash
docker logs --tail 100 jenkins
```

### Stop or restart Jenkins

```bash
docker stop jenkins
docker start jenkins
docker restart jenkins
```

### Remove only the container (keep data)

```bash
docker stop jenkins
docker rm jenkins
```

The `jenkins_home` Docker volume remains. **Do not remove the volume unless you deliberately want to delete Jenkins state and builds.**

---

## 12. Day 1 Practice Checklist

- [ ] Explain Jenkins in simple language.
- [ ] Explain the difference between CI and CD.
- [ ] Start Jenkins using Docker.
- [ ] Unlock Jenkins and create an admin account.
- [ ] Create a Freestyle job and run shell commands.
- [ ] Inspect the build history and console output.
- [ ] Create a three-stage Declarative Pipeline.
- [ ] Simulate a failed test and inspect the result.
- [ ] Restart Jenkins and verify that your jobs still exist.

## 13. Learning Roadmap

| Day | Topic | Practical Goal |
| --- | --- | --- |
| 1 | Jenkins basics | Install Jenkins and run first jobs |
| 2 | Jobs and Freestyle projects | Parameters, triggers, logs |
| 3 | Jenkins Pipeline | Jenkinsfile, stages, steps |
| 4 | Git / GitHub Integration | Checkout code and webhooks |
| 5 | Node.js CI | Install, lint, test, build |
| 6 | Docker + Jenkins | Build and push images |
| 7 | AWS / Kubernetes | Deploy an application safely |
| 8 | Security and RBAC | Credentials, roles, users |
| 9 | Agents and advanced pipelines | Parallel stages, approval, notification |
| 10 | End-to-end project | CI/CD pipeline and troubleshooting |

## 14. Interview Questions

**Q1. What is Jenkins?**  
An open-source automation server used to automate tasks including software builds, tests, and delivery.

**Q2. Why is Jenkins useful?**  
It creates a repeatable workflow, reduces manual work, and provides build feedback and logs.

**Q3. What is a Jenkins job?**  
A configured task that Jenkins can execute.

**Q4. What is a Jenkins pipeline?**  
A series of automated stages and steps, often defined in a `Jenkinsfile`.

**Q5. What is the difference between controller and agent?**  
The controller manages the Jenkins system and schedules jobs; agents provide execution environments for jobs.

**Q6. What is the difference between CI and Continuous Deployment?**  
CI integrates and checks code frequently; Continuous Deployment automatically releases changes that pass the configured pipeline.

**Q7. Does Jenkins automatically deploy every code change?**  
No. Deployment occurs only when a pipeline is configured and triggered to do it, with any required permissions and approvals.

**Q8. What is a Jenkins plugin?**  
An add-on that extends Jenkins functionality, such as Git integration or role-based authorization.

## 15. References

- [Official Jenkins Documentation](https://www.jenkins.io/doc/)
- [Jenkins Tutorials](https://www.jenkins.io/doc/tutorials/)
- [Jenkins with Docker](https://www.jenkins.io/doc/book/installing/docker/)
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Role-based Authorization Strategy Plugin](https://plugins.jenkins.io/role-strategy/)

---

**Next:** Day 2 — Jobs, Freestyle Projects, Triggers, Parameters, and Build Logs.
