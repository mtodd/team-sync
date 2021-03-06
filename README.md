# Team-Sync

This project demonstrates how to sync one Team to another Team using the GitHub API.

## Setup

Requires running your own Heroku cron daemon.

```
# clone the repo
$ git clone https://github.com/mtodd/team-sync
$ cd team-sync

# create a heroku app
heroku create
git push heroku master

# configure settings
# first generate a personal access token (with admin:org scope)
$ heroku config:set GITHUB_ACCESS_TOKEN=abcd1234
# set source team
$ heroku config:set SOURCE=myorg/source-team
# set target team(s)
$ heroku config:set TARGET=myorg/target-team

# configure cron task
$ heroku addons:add scheduler:standard
$ heroku addons:open scheduler
```

Schedule the `rake sync` task:

![](https://camo.githubusercontent.com/845a96db7011c1180ca149d20a89c3c9e245dd25/68747470733a2f2f662e636c6f75642e6769746875622e636f6d2f6173736574732f3133372f3332363137312f37633361346534382d396232342d313165322d383331332d6663363539323331656663312e706e67)
