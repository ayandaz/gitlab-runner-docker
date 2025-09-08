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

### Clone the repository:

```bash
git clone https://github.com/ayandaz/gitlab-runner-docker.git
cd gitlab-runner-docker
```

### Start the Runner
```bash
docker-compose up -d
```
This will start the GitLab Runner container in detached mode.

### Register the Runner
Get the registration token from GitLab.
Settings > CI/CD > Runners > Set up a specific runner (or Group Runners).
Run the following
```bash
docker exec -it gitlab-runner gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --registration-token "YOUR_REGISTRATION_TOKEN" \
  --executor "docker" \
  --tag-list "docker-unix-maven-runner" \
  --description "Docker Unix Runner" \
  --docker-image "maven:3.9.2-eclipse-temurin-17" \
  --docker-privileged \
  --docker-volumes "/config:/etc/gitlab-runner" \
  --docker-volumes "/cache:/cache:rw" \
  --docker-volumes "/var/run/docker.sock:/var/run/docker.sock"

```
After registration, your runner will appear in the GitLab project and be ready to run jobs.

## Stop the Runner
```bash
docker-compose down
```
This stops and removes the container but keeps the configuration in ./config.

### Example command to make sure that the runner can actually talk to the host Docker daemon through the mapped socket. Run this from inside the docker container.
```bash
docker exec -it gitlab-runner docker info
```
---
Notes
The Docker socket is mounted to allow the runner to use Docker commands inside jobs.
Runner configuration is persisted in the ./config directory.
You can restart the container anytime with docker-compose restart.

## SSH Configuration
- Generate SSH Key
  ```bash
  ssh-keygen -t ed25519 -C "ci-runner@container" -f id_ed25519 -N ""
  ssh-keyscan -t ed25519 gitlab.com >> known_hosts
  ```
- Run these commands to generate Base64 files
  ```bash
  openssl base64 -A -in ~/.ssh/id_ed25519 -out id_ed25519.b64
  openssl base64 -A -in ~/.ssh/known_hosts -out known_hosts.b64
  ```
- Copy the public key to gitlab - GitLab → Settings → SSH Keys → Paste the .pub key
- Create CICD variable `SSH_PRIVATE_KEY_B64` with the value of id_ed25519.b64
  ```bash
  cat /root/.ssh/id_ed25519.b64
  ```
- Create CICD variable `SSH_KNOWN_HOSTS_B64` and copy the value of known_hosts.b64 
  ```bash
  cat /root/.ssh/known_hosts.b64
  ```
### Use this script in CI pipeline.
```bash
echo "Setting up SSH connection to connect to GitLab"
eval "$(ssh-agent -s)"

mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Decode base64 key into file
echo "$SSH_PRIVATE_KEY_B64" | base64 -d > ~/.ssh/id_ed25519
chmod 400 ~/.ssh/id_ed25519
ssh-add ~/.ssh/id_ed25519

echo "$SSH_KNOWN_HOSTS_B64" | base64 -d > ~/.ssh/known_hosts
chmod 644 ~/.ssh/known_hosts
export GIT_SSH_COMMAND="ssh -i ~/.ssh/id_ed25519 -o UserKnownHostsFile=~/.ssh/known_hosts -o StrictHostKeyChecking=yes"

echo "Testing SSH connection to GitLab..." #todo fail this if theres no connection
ssh -T git@gitlab.com || echo "SSH test failed!"
```
