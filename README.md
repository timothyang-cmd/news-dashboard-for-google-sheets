# News Dashboard

## Purpose

 News Dashboard is a Google Sheets + Google Apps Script tool that collects news headlines from several sources and displays them in one clean spreadsheet tab.

It is designed for educators, researchers, and curious readers who want a simple way to monitor current news related to education, Japan, English learning, technology, and world affairs.

The dashboard currently pulls from:

* The New York Times Top Stories API
* The New York Times Most Popular API
* Google News RSS feeds
* Yahoo News Japan RSS feeds

The result is a single Google Sheet tab called `News`, showing each article’s source, feed/category, publication date, and clickable headline.

## Important Security Note

Do not put your API key directly inside the code.

Instead, this project uses Google Apps Script **Script Properties** to store the NYT API key safely.

In the code, this line:

```javascript
NYT_API_KEY: getScriptProperty_('NYT_API_KEY')
```

means:

“Look inside the Apps Script project settings and get the saved value called `NYT_API_KEY`.”

This keeps your API key out of the public GitHub repository.

## Files in This Project

```text
README.md
appsscript.json
.gitignore
src/Code.gs
```

### `README.md`

This file explains what the project does and how to set it up.

### `src/Code.gs`

This is the main Google Apps Script code. Paste this code into the Apps Script editor connected to your Google Sheet.

### `.gitignore`

This file helps prevent private or unnecessary files from being uploaded to GitHub.

### `appsscript.json`

This file contains Apps Script project settings, including the runtime and permission scopes needed for the script.

## How to Set It Up in Google Sheets

### 1. Create a Google Sheet

Create a new Google Sheet in your Google Drive.

You can name it something like:

```text
Schematic News Dashboard
```

### 2. Open Apps Script

In the Google Sheet, go to:

```text
Extensions → Apps Script
```

This opens the Google Apps Script editor.

### 3. Add the Code

Open the file:

```text
src/Code.gs
```

Copy all of the code from that file.

Paste it into the Apps Script editor.

### 4. Add Your NYT API Key Safely

In Apps Script, go to:

```text
Project Settings → Script Properties
```

Add a new script property:

```text
Property: NYT_API_KEY
Value: your actual NYT API key
```

Do not write the API key directly into the code.

### 5. Run the Script

In the Apps Script editor, choose this function:

```javascript
updateNewsDashboard
```

Then click **Run**.

The first time you run it, Google will ask you to approve permissions.

Approve the permissions so the script can:

* Access your Google Sheet
* Fetch news data from the web
* Translate Japanese headlines when needed

### 6. Check the Google Sheet

After the script runs, return to your Google Sheet.

You should see a tab called:

```text
News
```

The tab will contain the news dashboard.

## Optional: Make It Update Automatically

You can set the script to update automatically.

In Apps Script, go to:

```text
Triggers → Add Trigger
```

Choose:

```text
Function: updateNewsDashboard
Event source: Time-driven
```

Then choose how often you want it to run, such as once per day or once every few hours.

## Notes

This project does not include an API key. You must add your own NYT API key in Apps Script Script Properties.

Google News and Yahoo News Japan use RSS feeds, so they do not require API keys.

The dashboard is meant to be simple, readable, and easy to modify.
