---
layout: post
title:  "Automate code standards validation with Composer, Git Hooks and PHPCodeSniffer"
slug: 'automate-code-standards-validation-with-composer-git-hooks-and-phpcs'
date:  2018-01-21 13:00:00 +0300
category: clean-code
tags: [clean code, composer, git, phpcs]
icon: terminal
---

Recently I had to setup a new project and like always I prefer to enforce the code standards on the project so that I 
don't need to worry about this ever again. Because I realized there are a lot of people who don't know how this could
be accomplished I decided to write a short post about this, maybe it helps others out there. We will use composer, 
pre-commit git hook and off course phpcs to automate code quality checks.

### Install PHPCodeSniffer as a project dependency

First thing we have to do is to add phpcs as a development dependency.

```
{
    "require-dev": [
        "squizlabs/php_codesniffer": "^3.2"
    ]
}
```

### Automatically setup git hooks

Because we want to verify the project code standards before any commit, we have to setup a pre-commit git hook. In 
order to automate this step we can make use of [composer scripts](https://getcomposer.org/doc/articles/scripts.md) 
which will execute any script or command-line command that we specify at `composer install` or `composer update`.

```
{
    "scripts": {
        "install-hooks": ["rm -rf .git/hooks", "ln -s ../scripts/hooks .git/hooks", "chmod +x .git/hooks/pre-commit"]
        "pre-install-cmd": ["@install-hooks"],
        "post-install-cmd": ["@install-hooks"],
    }
}
```

Composer will copy our pre-commit script from the `scripts` directory into the hooks of the `.git` directory and make it
executable. In any git project, you have this `.git` directory that contains the `hooks` folder where all the git 
hooks script samples can be found. For more info, visit 
[git hooks documentation](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks).

```
➜  hooks git:(master) ll
total 80
-rwxr-xr-x  1 iulian.popa  staff   478B Jul  8  2017 applypatch-msg.sample
-rwxr-xr-x  1 iulian.popa  staff   896B Jul  8  2017 commit-msg.sample
-rwxr-xr-x  1 iulian.popa  staff   189B Jul  8  2017 post-update.sample
-rwxr-xr-x  1 iulian.popa  staff   424B Jul  8  2017 pre-applypatch.sample
-rwxr-xr-x  1 iulian.popa  staff   1.6K Jul  8  2017 pre-commit.sample
-rwxr-xr-x  1 iulian.popa  staff   1.3K Jul  8  2017 pre-push.sample
-rwxr-xr-x  1 iulian.popa  staff   4.8K Jul  8  2017 pre-rebase.sample
-rwxr-xr-x  1 iulian.popa  staff   544B Jul  8  2017 pre-receive.sample
-rwxr-xr-x  1 iulian.popa  staff   1.2K Jul  8  2017 prepare-commit-msg.sample
-rwxr-xr-x  1 iulian.popa  staff   3.5K Jul  8  2017 update.sample

```

### The pre-commit hook

Now whenever a developer attempts to commit their code, it will trigger the `pre-commit` hook. The script will run the 
phpcs rules on the specific files of the commit:

```
#!/bin/sh

PROJECT=`php -r "echo dirname(dirname(dirname(realpath('$0'))));"`
STAGED_FILES_CMD=`git diff --cached --name-only --diff-filter=ACMR HEAD | grep \\\\.php`

# Determine if a file list is passed
if [ "$#" -eq 1 ]
then
    oIFS=$IFS
    IFS='
    '
    SFILES="$1"
    IFS=$oIFS
fi
SFILES=${SFILES:-$STAGED_FILES_CMD}

echo "Checking PHP Lint..."
for FILE in $SFILES
do
    php -l -d display_errors=0 $PROJECT/$FILE
    if [ $? != 0 ]
    then
        echo "Fix the error before commit."
        exit 1
    fi
    FILES="$FILES $PROJECT/$FILE"
done

if [ "$FILES" != "" ]
then
    echo "Running Code Sniffer. Code standard PSR2."
    ./vendor/bin/phpcs --standard=PSR2 --encoding=utf-8 -n -p $FILES
    if [ $? != 0 ]
    then
        echo "Please fix the following errors before commit!"
        exit 1
    fi
fi

exit $?
```

More specific the script will get the staged files before commit, cheking the PHP syntax with `php lint (php -l)` and 
apply the phpcs rules, in our case PSR2, on those files. The commit will be aborted if there is a code standards 
violation.

This is a very simple setup every project should have, it will help you keep the code standards consistent and 
achieve a cleaner codebase. Hope you'll like it.









