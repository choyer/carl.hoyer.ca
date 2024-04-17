---
title: Dump MySQL Database with Git Post-Commit Hook
date: 2013-03-14
author: Carl Hoyer
description: A git post-commit hook to dump a mysql database with commit SHA id appended the filename.
keywords: git, post-commit, hooks, mysql
tags: db, git
---

Lately, most of my web development projects have been [ExpressionEngine](http://ellislab.com/expressionengine) sites. I have adopted a [bootstrapped](http://ee-garage.com/nsm-config-bootstrap) multi environment [Git](http://git-scm.com/) + [Beanstalk](http://beanstalkapp.com/) (with deployments) workflow to manage all changes.

Rather than put the database under direct source control, I have opted to dump the database to a file using a Git post-commit hook while appending a timestamp and the git commit SHA id to the filename for reference.

Here is my `.git/hooks/post-commit` to do that:

```sh
#!/bin/sh
# ./.git/hooks/post-commit
# dump local database and store

# Time and commit id
current_time=$(date +%F-%H_%M_%S)
git_commit_id=$(git rev-parse --short HEAD)

# CHANGE THIS: Database details
db_user='root'
db_pwd='root'
db_name='khu-ee'
dump_filename=$db_name-$current_time-$git_commit_id.sql

# Go go database dump
printf "==> Dumping DB ..."
/Applications/MAMP/Library/bin/mysqldump --add-drop-table --user=$db_user --password=$db_pwd $db_name | cat > ~/Sites/local_krisholm_com/db/$dump_filename
printf " Done. [$dump_filename]\n\n"
```
Replace mysql login details and *database-name* with your settings.
