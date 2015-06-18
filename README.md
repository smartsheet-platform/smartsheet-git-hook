# smartsheet-git-hook

This project contains a **post-receive** git hook python script that updates a bug tracking sheet in Smartsheet. It detects that a defect has been resolved by inspecting the commit messages that are pushed to a git branch. Please consult [Customizing Git - Git Hooks](http://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) for more information about Server-Side Hooks.

## Overview

As implemented, a git push that contains commit message text that includes a case-insensitive match for **fix(es) bug-xxxx** will trigger a search for the bug in the tracking sheet configured in the post-receive.cfg json file. 

The script searches for the bug specified in the commit message and validates that it matches a single row in the sheet with a Status column value of **Open**.

Upon such a match, the script updates the matched row with the following changes:
- **Status** column set to **Resolved**
- **Resolution** column set to **Fixed**
- **Fix in Branch** column set to the git branch as deteremined by the stdin stream sent to the script.
- **Fixed By** column set to the email address of committer.
- **Target Release** column set to **xx - yyyyy** if the git branch is named **release-xx-yyyyy**, otherwise, left unchanged
- **Comments** column is set or appended with **Fixed in commit zzzz by nnnn** where *zzzz* is the commit reference and *nnnn* is the committer's name. 
- If the optional **url** configuration attribute is set to something like https://my.git.lab.host/group-name/project-name (assuming GitLab), it also appends a line containing a link to the code diff https://my.git.lab.host/group-name/project-name/commit/zzzz When appending, the added lines are preceeded by two newlines.

For each successful update, the script outputs a message of the form: **Bug-bbbb updated in Bug Tracker** where *bbbb* is the bug number.

## Configuration
The configuration file is a json file that contains the following:

```json
{
   "sheetId" : "0000000000000000",
   "prefix" : "Bug-",
   "token" : "abcdefghijklmnopqrstuvwxyz",
   "git" : "/usr/local/git/bin/git",
   "url" : "https://my.git.lab.host/group-name/project-name"
}
```

The **sheetId** attribute is the identifier for the Smartsheet containing bug tracking information discoverable as the **Sheet ID** attribute when inspecting Sheet Properties.

The **prefix** attribute is configured to match the prefix in the Display Format of the bug tracking sheet whose column type is set to **Auto-Number/System** and the system-generated column is set to **Auto-Number**.

The **token** attribute is the Smartsheet REST API access token generated in the Personal Settings dialog box of the user under which the script will run.

The **git** attribute is the absolute path to the git command used by the script to retrieve the commit information corresponding to the commit reference received.

The **url** is an optional base URL for the git project's web interface for viewing commits.

## Deployment
Follow your git server's instructions for installing git hooks which likely entails placing both the **post-receive** file and a modified **post-receive.cfg** file in the hooks directory of the bare git repository.

If the **post-recieve.cfg** file is missing or unreadable, the **post-recieve** script even if deployed becomes a no-op but will output the following message: **Warning: unable to read post-receive.cfg**

## Bug Tracking Sheet

The script assumes the existence of the following columns in the bug tracking sheet in Smartsheet. The columns do not have to appear in any particular order, as they are referenced by their name:

| Column Name | Description | Needed for |
|-------------|-------------|------------|
| **ID** | The auto-generated column that contains the defect identifier | Identifying the defect |
| **Status** | A Dropdown List with valid values: `Open`, `Resolved`, `Closed` | Updated by script |
| **Resolution** | A Dropdown List with valid values: `Fixed`, `Cannot Repro`, `Not a Bug`, `Won't Fix`, and `Duplicate` | Updated by script |
| **Fix in Branch** | A Dropdown List containing valid branches | Updated by script |
| **Fixed By** | A Contact List that minimally contains potential committers | Updated by script |
| **Target Release** | A Dropdown list containing valid releases of the form **xx - nnnn**, where *xx* is a number and *nnnn* is a name | Conditionally updated by script |
| **Comments** | A column that allows Text/Number values | Updated by script |

Alternatively, the more specific Dropdown list and Contact List columns can be relaxed to simply allow Text/Number values or be set up with alternate valid values.

Any number of additional columns are allowed.

#License

Copyright 2015 [Smartsheet, Inc.](https://www.smartsheet.com)

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
