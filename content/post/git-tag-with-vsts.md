---
title: "Git Tag with VSTS"
date: 2018-04-26T21:19:50+01:00
tags: ["VSTS"]
draft: true
---


```powershell
$postUrl = $ENV:BUILD_REPOSITORY_URI -replace "_git", "_apis/git/repositories"
$body = @{
name = "refs/tags/$Env:FullVersion"
oldObjectId = "0000000000000000000000000000000000000000"
newObjectId = "$ENV:BUILD_SOURCEVERSION"
}
Write-Host "Tagging $ENV:BUILD_SOURCEVERSION with $ENV:FullVersion"
$result = Invoke-RestMethod "$postUrl/refs/tags?api-version=2.0" -Method Post -Body (ConvertTo-Json @($body)) -ContentType "application/json" -UseDefaultCredentials
```