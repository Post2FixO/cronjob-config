# Git Synchronization via Cronjob Setup Documentation

WARNING: destructive operation. The script will delete anything that won't sync with the repository before replacing and updating it with the repository content.

Sync a Github repository to a Linux host with 2 intervals (CI/CD workaround when Linux host doesn't have a public IP/URL)

## Overview

This document outlines the steps to set up an automated interval based process to synchronize a local repository with a specific branch on GitHub using a cronjob. The setup ensures that the local server remains up-to-date with the latest changes pushed to a branch of choice.

The cronjob periodically pulls changes from a specific branch in a GitHub repository to a local directory. The setup involves:

- Cloning the repository with a specific branch.
- Configuring a script to fetch and reset the repository regularly.
- Automating the script execution using cron.
- Setting up Git to store credentials for automatic authentication.

## Steps

### 1. Clone the Repository

Start by cloning the repository with the specific branch you intend to track:

```bash
git clone -b "<branch name>" --single-branch https://github.com/<github user or org>/<repo name> ~/<destination folder>
```

### 2. Create a Bash Script

Create a bash script `<script_name>.sh` to handle the fetching, resetting, and pulling of the repository. You can place this script anywhere or in a dedicated scripts directory (e.g. `~/cronjobs`) to avoid accidental deletion or interference.

**Example Script Content (`~/cronjobs/<script_name>.sh`):**

```bash
#!/bin/bash
cd ~/<destination folder>
# Reset local changes and ensure the repo is clean
git fetch --all
git reset --hard "<branch name>"
git clean -fdx

# Pull the latest changes from the correct branch
for i in {1..29}; do
  git pull origin "<branch name>"
  sleep 20
done
```

This script runs at minute intervals (based on the cron job config in step 4), and includes a loop to allow sub minute syncs.
Each time the script runs it will first clean up the folder and sync it with the repository (DELETING ANYTHING THAT WON'T SYNC) and then continue pulling the branch at your set interval.
The example script is recommended. It is designed to run a cleanup every 10 minutes (heavy operation) and to pull the repo every 20 seconds (light operation). The launch script interval is set in the next sections.
The loop will run 3 times a minute, for 10 minutes until the script is called again.

### 3. Make the Script Executable

Change the permissions of the script to make it executable:

```bash
chmod +x ~/cronjobs/api-gateway.sh
```

### 4. Set Up Cron

Configure the cronjob to run the script at the desired frequency. For instance, to execute every 10 minutes (recommended):

```cron
*/10 * * * * /<script path>/<script_name>.sh >> /<script_path>/pull.log 2>&1
```

This will show logs in the terminal until stopped (control + c)

### 5. Configure Git Credential Storage

To avoid entering credentials repeatedly, configure Git to store your credentials:

```bash
git config credential.helper store
```

The manually run `git pull` to prompt for credentials and enter them. They will be saved for subsequent pulls. (note that as of the time of this writing, only classic PAT tokens seem to work, not the new PAT (still in Beta) tokens)

### 6. Handling Branch Switches

If you need to switch to another branch, update the `<script_name>.sh` script with the new branch name (2 entries). Re-run the script manually to verify that it operates correctly with the new branch.
