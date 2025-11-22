# AWS CI CD MLProject
# Student Performance Indicator – Docker + GitHub Actions + AWS

Python 3.8 ML app (student performance prediction) containerized with Docker, built and pushed to a private AWS ECR repository, and deployed automatically to an EC2 Ubuntu instance via a GitHub Actions CI/CD pipeline.

## 1. Flow
Commit to main (README.md ignored) → CI (lint/tests placeholder) → Build & push Docker image to ECR → Self‑hosted runner on EC2 pulls image → Runs container on port 8080.

## 2. Docker
```dockerfile
FROM python:3.8-slim-buster
WORKDIR /app
COPY . /app
RUN apt-get update -y && pip install --no-cache-dir -r requirements.txt
EXPOSE 8080
CMD ["python3","app.py"]
```
Build & run locally:
```bash
docker build -t student-performance-app .
docker run -p 8080:8080 student-performance-app
```

## 3. GitHub Actions ( .github/workflows/main.yaml )
Jobs:
- Continuous Integration (ubuntu-latest): checkout, lint, unit test placeholder.
- Continuous Delivery: login to ECR, build, tag, push image (latest).
- Continuous Deployment (self-hosted EC2 runner): pull image, run container.
Note: README.md changes do not trigger the workflow (paths-ignore).

## 4. Required GitHub Secrets
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- AWS_REGION (e.g. us-east-1)
- AWS_ECR_LOGIN_URI (e.g. <account>.dkr.ecr.<region>.amazonaws.com)
- ECR_REPOSITORY_NAME (e.g. student-performance)

Common mistakes: putting full URI (with repo) into ECR_REPOSITORY_NAME or wrong login URI.

## 5. AWS Setup
1. IAM User: attach AmazonEC2FullAccess + AmazonEC2ContainerRegistryFullAccess (or narrower).
2. ECR: create private repo (student-performance).
3. EC2: Ubuntu (open inbound TCP 8080; also 22 for SSH). Install Docker:
   ```bash
   sudo apt update && sudo apt upgrade -y
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   sudo usermod -aG docker $USER
   ```
4. Self‑hosted Runner: add via GitHub → Settings → Actions → Runners (Linux). Run `./run.sh` and keep active.

## 6. Deployment Test
Access:
- Home: http://<ec2_public_ip>:8080/
- Predict: http://<ec2_public_ip>:8080/predictdata

Ensure Security Group has Custom TCP 8080 open.

## 7. Troubleshooting
- Empty registry tag: check step id (login-ecr) and secrets.
- Repo not found: fix ECR_REPOSITORY_NAME.
- Port blocked: add inbound 8080 rule.
- Runner idle: keep `./run.sh` session running.
- AWS_REGION in deployment job: use `${{ secrets.AWS_REGION }}` (env not defined).

## 8. Cleanup
Terminate EC2, delete ECR repo, remove runner, revoke IAM keys to avoid charges.
