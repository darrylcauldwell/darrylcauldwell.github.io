+++
title = "Post To Slack From Powershell"
date = "2015-04-09"
description = "Post To Slack From Powershell"
tags = [
    "powershell"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
Working in a technical team distributed across multiple time zones makes it difficult to collaborate and keep everyone on the same page. I’ve used various communications tools to try and help with this and today the team I am working with started to look at Slack.  Slack seems ideal for our needs as it provides easily searchable discussion forums and has apps available for Mac, Windows, iOS and Android, it has a free usage model with limitations which won’t effect our working and if we grow we can extend to a paid license.

The other really neat thing is that all its features are fully exposed by its API and it integrates via webhooks with many other applications including GitHub, GoogleDrive and Jira. As we added integrations and started to look at the API documentation as a trial I explored posting messages using the REST api from Powershell. It turned out to be really easy to do,  here are my notes on how you can.

## How To
While logged into slack use the following URL https://api.slack.com/web#authentication

This should take you to screen to issue a OAuth2 token.

![Slack OAuth2 Token](/images/slack-powershell-token.png)

Click Create token and copy the text string for the token. We then need to format the message and call postMessage method with Powershell.

```
$postSlackMessage = @{token="<your-token>";channel="#general";txt="Hello from PowerShell!";username="via API"}
Invoke-RestMethod -Uri https://slack.com/api/chat.postMessage -Body $postSlackMessage
```

This should return a message like

```bash
ok   channel   ts                message
--   -------   --                -------
True C04BHJ94Z 1428592795.000059 @{text=Hello from PowerShe...
```

You can do everything you want via the API not just post messages,  check out the [API documentation](https://api.slack.com/) for the other methods.