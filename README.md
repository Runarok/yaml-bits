# Auto-Committer


```yml
name: Push 100 Empty Commits  # Name of the workflow

on:
  push:
    branches:
      - main  # Trigger on pushes to the 'main' branch.
  
  schedule:
    - cron: "0 0 1 1 *"  # This runs every hour.

  workflow_dispatch:  # This allows the workflow to be manually triggered.

jobs:
  push_100_empty_commits:
    runs-on: ubuntu-latest  # Use the latest Ubuntu runner.

    steps:
      # Step 1: Checkout the repository to make changes.
      - name: Checkout repository
        uses: actions/checkout@v3  # Checkout the repository.
        with:
          persist-credentials: false  # Avoid storing credentials permanently.
          fetch-depth: 0  # Fetch the full history to avoid issues with git operations.

      # Step 2: Configure Git identity
      - name: Configure Git identity
        run: |
          git config --global user.email "RunarokHrafn@gmail.com"  # Set commit email.
          git config --global user.name "Runarok"  # Set commit author name.

      # Step 3: Pull the latest changes from the remote repository to avoid reference lock.
      - name: Pull latest changes
        run: git pull origin main  # Pull the latest changes from the 'main' branch.

      # Step 4: Create Empty.Readme file and commit 100 times
      - name: Create Empty.Readme and commit 100 times
        run: |
          # Create an empty README file.
          touch Empty.Readme
          
          # Loop to make 100 empty commits with commit number in the file.
          for i in {1..100}; do
            echo "Commit number: $i" >> Empty.Readme  # Append commit number to file.
            git add Empty.Readme  # Stage the file for commit.
            git commit -m "Empty commit #$i"  # Commit with a unique message.
          done

      # Step 5: Delete the Empty.Readme file after committing.
      - name: Delete Empty.Readme
        run: |
          rm -f Empty.Readme  # Delete the Empty.Readme file.
          git add -A  # Stage changes (file deletion).
          git commit -m "Delete Empty.Readme after 100 commits"  # Commit file deletion.

      # Step 6: Pull the latest changes again to ensure no reference lock issues before pushing.
      - name: Pull latest changes before push
        run: git pull origin main  # Pull again to ensure there are no conflicts or locks.

      # Step 7: Push the commits to the remote repository.
      - name: Push changes to remote
        uses: ad-m/github-push-action@master  # Use GitHub Push Action to push changes.
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}  # Use GitHub token to authenticate the push.
```

```yml
name: update-time  # Name of the workflow

# Trigger the workflow on the following events:
on:
  # Trigger on push events to the 'main' branch.
  push:
    branches:
      - main  # Trigger on pushes to the 'main' branch.
  
  # Trigger every hour based on cron schedule.
  schedule:
    - cron: "1 * * * *"  # This runs every hour.

      # Add manual dispatch option
  workflow_dispatch:  # This allows the workflow to be manually triggered.

jobs:
  # Define the job that will run on an Ubuntu machine.
  auto_commit:
    runs-on: ubuntu-latest  # Use the latest Ubuntu runner.

    steps:
      # Step 1: Checkout the repository to make changes.
      - uses: actions/checkout@v3  # Checkout the repository.
        with:
          persist-credentials: false  # Avoid storing credentials permanently.
          fetch-depth: 0  # Fetch the full history to avoid issues with git operations.

      # Step 2: Set date and time variables.
      - name: Set date and time variables  # Define a step to set date and time variables.
        run: |
          # Get the current date in 'YYYY-MM-DD' format.
          current_date=$(date +'%Y-%m-%d')  # Format the current date.
          # Get the current military time (24-hour format).
          military_time=$(date +'%H:%M:%S')  # Format the time in 24-hour format.
          # Get the current time in 12-hour format with AM/PM.
          normal_time=$(date +'%I:%M:%S %p')  # Format time in 12-hour format.
          # Get the current UTC time.
          utc_time=$(date -u +'%H:%M:%S UTC')  # Get UTC time in 24-hour format.
          
          # Set these variables for use in later steps.
          echo "DATE=$current_date" >> $GITHUB_ENV  # Set the current date as a GitHub environment variable.
          echo "MILITARY_TIME=$military_time" >> $GITHUB_ENV  # Set the military time.
          echo "NORMAL_TIME=$normal_time" >> $GITHUB_ENV  # Set the normal time.
          echo "UTC_TIME=$utc_time" >> $GITHUB_ENV  # Set UTC time.

      # Step 3: Create a JSON file with the time data and save it to time.json.
      - name: Write JSON into time.json  # Define a step to write the time data into a JSON file.
        run: |
          echo "{
            \"Date\": \"$DATE\",
            \"MilitaryTime\": \"$MILITARY_TIME\",
            \"NormalTime\": \"$NORMAL_TIME\",
            \"UTC\": \"$UTC_TIME\"
          }" | jq . > time.json  # Format and save the time data into time.json using jq.

      # Step 4: Commit the changes (time.json) to the repository.
      - name: Commit  # Define a step to commit the changes.
        run: |          
          git config --local user.email "RunarokHrafn@gmail.com"  # Set commit email.
          git config --local user.name "Runarok"  # Set commit author name.
          git add -A  # Stage all changes for commit.
          # Create a commit message with the date and UTC time.
          git commit -m "$DATE ($UTC_TIME)"  # Commit with a message that includes the date and UTC time.

      # Step 5: Push the commit to the remote repository.
      - name: Push  # Define a step to push the commit to GitHub.
        uses: ad-m/github-push-action@master  # Use GitHub Push Action to push changes.
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}  # Use GitHub token to authenticate the push.
```
          
          
