# GitLab Daily Stand-Up Workflow Documentation

![Workflow Overview](./gitlab-slack%20automation.png)

## Summary

This n8n workflow automates daily stand-up meetings for a freelance team using Slack and GitLab. It runs on weekdays and performs two main functions:

1. **Morning Post (12:45 PM)**: Posts a daily update message in Slack with each team member's GitLab tasks
2. **Afternoon Reminder (12:50 PM)**: Sends a reminder to team members who haven't replied to yesterday's stand-up

The workflow pulls open GitLab issues, matches them to freelancers, formats them into a Slack message, and tracks who responds.

## Data Flow Summary

1. Workflow triggers on schedule (weekdays only)
2. Fetches all open GitLab issues across multiple pages
3. Retrieves freelancer list and Slack channel members
4. Matches freelancers to their assigned tasks
5. Posts daily update message with task overview
6. Saves message metadata to Google Sheets
7. Later checks previous day's thread for responses
8. Sends reminders to team members who haven't replied to yesterday's update

## Key Integrations

* **Slack**: For posting messages and tracking responses
* **GitLab API**: For fetching open issues
* **Google Sheets**: For storing freelancer list and tracking message metadata

## Node Breakdown

### Triggers

* **Schedule Trigger** - Runs daily at 12:45 PM to initiate the morning stand-up workflow
* **Schedule Trigger2** - Runs daily at 12:50 PM to check for non-responders from the previous day

### Weekday Check

* **If** - Checks if today is a weekday (Monday-Friday) before running the morning workflow
* **If1** - Checks if today is a weekday before sending yesterday's reminders

### GitLab Data Collection

* **HTTP Request** - Fetches page 1 of open issues from GitLab API (100 issues per page)
* **Code in JavaScript1** - Checks response headers to determine total number of pages
* **HTTP Request Page2+** - Fetches additional pages of issues if more than 100 exist
* **Split Out** - Splits the first page response body into individual issue items
* **Merge1** - Combines issues from all pages into a single dataset
* **Edit Fields** - Extracts relevant fields from each issue (title, labels, assignee, web URL)
* **Filter** - Removes issues that don't have an assignee
* **Code in JavaScript2** - Transforms issue data into a cleaner format with assignee information

### Team Member Management

* **All freelancers** - Retrieves the list of freelancers from Google Sheets
* **All freelancers names** - Extracts just the full names from the freelancer list
* **Get members of a channel** - Gets all members from the Slack channel
* **Get a user's profile** - Fetches Slack profile details for each channel member
* **Channel UserIDs** - Extracts real names and Slack member IDs
* **Freelancers Check** - Matches freelancers from Google Sheets with Slack channel members

### Message Formatting

* **Merge** - Combines freelancer data with their assigned GitLab issues
* **Main Message Formated1** - Formats the daily overview message showing each freelancer's tasks with Slack mentions and issue links
* **UserID CleanUP** - Extracts unique member IDs of people who have tasks assigned

### Slack Message Posting

* **Main Message** - Posts the initial "Daily Update" message to Slack asking for status updates
* **IDs & TS** - Combines the member IDs with the message timestamp
* **Aggregate** - Aggregates data for storage
* **Report Message** - Posts the formatted GitLab task overview as a threaded reply

### Tracking & Storage

* **Append row in sheet** - Saves the message ID, date, and list of user IDs to Google Sheets for tracking

### Response Monitoring (Yesterday's Thread)

* **Get row(s): PreviousDay** - Retrieves yesterday's message data from Google Sheets (accounting for weekends)
* **Code in JavaScript** - Gets the last row if multiple results exist
* **Thread of message** - Fetches all replies from yesterday's Slack thread
* **Replied Users** - Extracts user IDs of people who replied
* **Filter_Replied Users** - Keeps only records with actual replies
* **Received Message** - Retrieves the list of users who were supposed to respond
* **Non-Responding** - Identifies users who didn't reply and formats Slack mentions
* **Send a message** - Posts a reminder to non-responders in yesterday's thread
