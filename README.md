# Auto-Committer

<details>
<summary>Push 100 Empty Commits</summary>
  
```yml
name: Push 100 Empty Commits  # Workflow name that describes the purpose

on:
  push:
    branches:
      - main  # Trigger this workflow when changes are pushed to the 'main' branch.
  
  schedule:
    - cron: "0 0 1 1 *"  # Schedule the workflow to run on the 1st day of every year at midnight (UTC).

  workflow_dispatch:  # Allow the workflow to be manually triggered via GitHub Actions UI.

jobs:
  push_100_empty_commits:
    runs-on: ubuntu-latest  # The job will run on the latest version of an Ubuntu runner.

    steps:
      # Step 1: Checkout the repository to make changes
      - name: Checkout repository
        uses: actions/checkout@v3  # Use the checkout action to clone the repository.
        with:
          persist-credentials: false  # Avoid storing credentials permanently (security reason).
          fetch-depth: 0  # Fetch the entire git history, required for git operations.

      # Step 2: Set up Git identity for commits
      - name: Configure Git identity
        run: |
          git config --global user.email "RunarokHrafn@gmail.com"  # Set commit author's email.
          git config --global user.name "Runarok"  # Set commit author's name.

      # Step 3: Pull the latest changes from the remote to avoid conflicts
      - name: Pull latest changes
        run: git pull origin main  # Pull the latest changes from the 'main' branch before starting work.

      # Step 4: Create an empty README file and commit 100 times
      - name: Create Empty.Readme and commit 100 times
        run: |
          # Create an empty README file.
          touch Empty.Readme
          
          # Loop to create 100 empty commits, appending commit number to the README each time.
          for i in {1..100}; do
            echo "Commit number: $i" >> Empty.Readme  # Append the commit number to the README file.
            git add Empty.Readme  # Stage the changes for commit.
            git commit -m "Empty commit #$i"  # Create a commit with a message indicating the commit number.
          done

      # Step 5: Remove the Empty.Readme file after all commits
      - name: Delete Empty.Readme
        run: |
          rm -f Empty.Readme  # Delete the Empty.Readme file that was used for commits.
          git add -A  # Stage the deletion of the file.
          git commit -m "Delete Empty.Readme after 100 commits"  # Commit the file deletion.

      # Step 6: Pull latest changes again to ensure no conflicts before pushing
      - name: Pull latest changes before push
        run: git pull origin main  # Pull the latest changes from the 'main' branch again to avoid reference locks or conflicts.

      # Step 7: Push the changes to the remote repository
      - name: Push changes to remote
        uses: ad-m/github-push-action@master  # Use the GitHub push action to push the local commits.
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}  # Use the GitHub token to authenticate and authorize the push.
```
</details>

<details>
<summary>update-time</summary>

```yml
name: update-time  # Workflow name that describes its purpose.

# Trigger the workflow on the following events:
on:
  # Trigger on push events to the 'main' branch.
  push:
    branches:
      - main  # Trigger this workflow when changes are pushed to the 'main' branch.
  
  # Schedule the workflow to run every hour.
  schedule:
    - cron: "1 * * * *"  # This runs every hour, on the 1st minute of the hour.

  # Allow manual triggering of the workflow via GitHub's interface.
  workflow_dispatch:  # This option allows the workflow to be triggered manually.

jobs:
  # Define the job that will execute the update on an Ubuntu runner.
  auto_commit:
    runs-on: ubuntu-latest  # Use the latest Ubuntu runner for the job.

    steps:
      # Step 1: Checkout the repository to make changes.
      - uses: actions/checkout@v3  # Checkout the repository to enable working with the files.
        with:
          persist-credentials: false  # Prevent storing credentials permanently (for security).
          fetch-depth: 0  # Fetch the full git history to avoid issues with git operations.

      # Step 2: Set date and time variables.
      - name: Set date and time variables  # Define a step to set variables for date and time.
        run: |
          # Get the current date in 'YYYY-MM-DD' format.
          current_date=$(date +'%Y-%m-%d')  # Get current date in 'YYYY-MM-DD' format.
          # Get the current military time (24-hour format).
          military_time=$(date +'%H:%M:%S')  # Get the time in 24-hour (military) format.
          # Get the current time in 12-hour format with AM/PM.
          normal_time=$(date +'%I:%M:%S %p')  # Get the time in 12-hour format with AM/PM.
          # Get the current UTC time.
          utc_time=$(date -u +'%H:%M:%S UTC')  # Get the UTC time in 24-hour format.
          
          # Set these date and time variables for use in later steps.
          echo "DATE=$current_date" >> $GITHUB_ENV  # Save the current date as a GitHub environment variable.
          echo "MILITARY_TIME=$military_time" >> $GITHUB_ENV  # Save military time (24-hour format).
          echo "NORMAL_TIME=$normal_time" >> $GITHUB_ENV  # Save normal time (12-hour format with AM/PM).
          echo "UTC_TIME=$utc_time" >> $GITHUB_ENV  # Save UTC time as an environment variable.

      # Step 3: Create a JSON file with the time data and save it to time.json.
      - name: Write JSON into time.json  # Write the time data into a JSON file.
        run: |
          # Create a JSON structure containing the time variables.
          echo "{
            \"Date\": \"$DATE\",
            \"MilitaryTime\": \"$MILITARY_TIME\",
            \"NormalTime\": \"$NORMAL_TIME\",
            \"UTC\": \"$UTC_TIME\"
          }" | jq . > time.json  # Format the data using jq and save it into time.json.

      # Step 4: Commit the changes (time.json) to the repository.
      - name: Commit changes  # Step to commit the time.json file to the repository.
        run: |          
          git config --local user.email "RunarokHrafn@gmail.com"  # Set the Git commit email.
          git config --local user.name "Runarok"  # Set the Git commit author name.
          git add -A  # Stage all the changes (including time.json) for commit.
          # Commit with a message that includes the current date and UTC time for easy tracking.
          git commit -m "$DATE ($UTC_TIME)"  # Commit with a message containing the date and UTC time.

      # Step 5: Push the commit to the remote repository.
      - name: Push changes to GitHub  # Push the committed changes to the remote repository.
        uses: ad-m/github-push-action@master  # Use the GitHub Push Action to push the changes.
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}  # Use the GitHub token to authenticate the push operation.
```
          
</details>          
