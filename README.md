# Infinion-test-runner
# Self-Hosted GitHub Actions Runner 

This project demonstrates the setup and configuration of a **self-hosted GitHub Actions runner** on my local environment(Ubuntu 22.02).  
It includes documentation of the setup process, challenges faced, security considerations, and notes on how this would be implemented in a production environment.

---

## 🧰 Setup Steps

1. **Created a repository** on GitHub for this as instructed.
2. **Generated a self-hosted runner token** from  
   Settings > Actions > Runners > New self-hosted runner.
3. **Downloaded and configured the runner** on my local machine:
   ```bash
   mkdir actions-runner && cd actions-runner
   curl -o actions-runner-linux-x64.tar.gz -L https://github.com/actions/runner/releases/latest/download/actions-runner-linux-x64.tar.gz
   tar xzf ./actions-runner-linux-x64.tar.gz
   ./config.sh --url https://github.com/<username>/<repo> --token <token>     

4. Started the runner manually:
./run.sh

Output showed:

√ Connected to GitHub
Listening for Jobs

5. Created a workflow file at `.github/workflows/test-runner.yml`:

```yaml
name: Temitope Self-Hosted Runner

on:
  workflow_dispatch:
  push:
    branches: [ main ]

jobs:
  localrunner-test:
    runs-on: self-hosted
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Display runner info
        run: |
          echo "Running on $(hostname)"
          uname -a

```
6. Pushed the workflow and triggered it manually.
The job was successfully picked up by my local runner and completed with a green “Job succeeded” status.

⚙️ Challenges Faced and How I Solved Them
1. Job stuck in "Queued" state

The issue was caused by a label mismatch (runs-on: [self-hosted, local] while my runner had self-hosted, Linux, X64).

I resolved it by editing the workflow file to use only runs-on: self-hosted.

2. Runner connection conflict error

Initially, I got:

Runner connect error: Error: Conflict. A session for this runner already exists.


Fixed it by stopping old runner processes using:

sudo pkill -f Runner


Then restarted with:

./run.sh

3. Workflow didn’t trigger immediately

Ensured the runner was online and assigned to the same repository before re-triggering the job.

🔒 Security Considerations

Stored and used the GitHub runner token only once during registration, never saved it in any file.

Verified that only trusted users have write access to the repo (to prevent malicious workflow execution).

Avoided running arbitrary scripts as root.

Used the least-privilege principle and removed the runner after testing:

./config.sh remove

🏭 What I Would Do Differently in a Production Environment

Use dedicated VMs or containers for self-hosted runners, isolated from personal environments.

Enable automatic scaling using GitHub Actions Runner Controller (ARC) on Kubernetes.

Secure secrets using GitHub Encrypted Secrets or HashiCorp Vault.

Implement CI/CD monitoring tools (like Prometheus + Grafana) to track runner health and job performance.

Automate runner cleanup to prevent token leaks or stale connections.
