# GitLab Runner Docker Setup

This repository provides a Docker Compose setup to run a GitLab Runner in a container.

## Git Repository

[https://github.com/ayandaz/gitlab-runner-docker.git](https://github.com/ayandaz/gitlab-runner-docker.git)

---

## Prerequisites

- [Docker](https://www.docker.com/get-started) installed on your system.
- [Docker Compose](https://docs.docker.com/compose/install/) installed.
- GitLab account and access to a project to register the runner.

---

## Installation

1. Clone the repository:

```bash
git clone https://github.com/ayandaz/gitlab-runner-docker.git
cd gitlab-runner-docker
```

2. Create a configuration directory for the runner:

```bash
mkdir config
```
## Start the Runner
```bash
docker-compose up -d
```
This will start the GitLab Runner container in detached mode.

## Registering the Runner
Enter the running container:
```bash
docker exec -it gitlab-runner gitlab-runner register
```
Follow the prompts:

- GitLab instance URL: Enter your GitLab URL (e.g., https://gitlab.com/ or your self-hosted instance).
- Registration token: Get this from your GitLab project under Settings > CI/CD > Runners > Set up a specific runner.
- Description: My very own docker container
- Tags: docker-unix-runner
- Executor: Choose docker.
- Default Docker image: Specify an image to use for jobs (e.g., docker:latest).

After registration, your runner will appear in the GitLab project and be ready to run jobs.

## Stop the Runner
```bash
docker-compose down
```
This stops and removes the container but keeps the configuration in ./config.

### Example command to make sure that the runner can actually talk to the host Docker daemon through the mapped socket
```
docker exec -it gitlab-runner docker info
```
---
Notes
The Docker socket is mounted to allow the runner to use Docker commands inside jobs.
Runner configuration is persisted in the ./config directory.
You can restart the container anytime with docker-compose restart.

## SSH Configuration
Generate SSH Key
```
docker exec -it gitlab-runner bash
mkdir -p /root/.ssh
chmod 700 /root/.ssh
cd /root/.ssh
ssh-keygen -t ed25519 -C "ci-runner@container" -f id_ed25519 -N ""
ssh-keyscan -t ed25519 gitlab.com >> known_hosts
chmod 644 known_hosts
```
Verify Permissions
```
ls -l
```
Copy the public key to gitlab - GitLab → Settings → SSH Keys → Paste the .pub key
```
cat /root/.ssh/id_ed25519.pub
```
Verfiy log in works
```
ssh -T git@gitlab.com
```
