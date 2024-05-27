[![build status](https://github.com/joooostb/clean-direnv/actions/workflows/main.yml/badge.svg)](https://github.com/joooostb/clean-direnv/actions/workflows/main.yml)
[![PyPI version](https://badge.fury.io/py/clean-dotenv.svg)](https://badge.fury.io/py/clean-dotenv)

clean-dotenv (and direnv)
======================

Automatically creates an `.env.example`/`.envrc.example` which creates the same keys as your `.env` or `.envrc` file, but without the values.

## Why?
There are projects which make use of an `.env` or `.envrc` file. These files contain environment variables which are used during runtime, such as API keys.
A typical `.env` file might look like this:

```bash
OPENAI_KEY="12345"
S3_BUCKET_NAME="testbucket"
```

A typical `.envrc` file looks very similar, but as it's mounted by [direnv](https://direnv.net/) automatically it needs to be prefixed with export:
```bash
export OPENAI_KEY="12345"
export S3_BUCKET_NAME="testbucket"
```

An `.env` or `.envrc` file should not be commited into a repo, since it can contain sensitive information[^1] (you should probably add them into your `.gitignore`). However, one needs to know which variables can be set in the file and as a result, a lot of projects (such as laravel), provide an example file which is a template file. So you would copy the `.env.template`, rename it to `.env` and fill in the environment variables.

However, there is one issue with this approach: If one introduces a new environment variable, they need to remember to add it to the `.env.example` or `.envrc.example` file. Unfortunately, if they forget this, it will not be noticed, since the program is using the `.env` file and not the `.env.example`. This pre-commit hook tries to mitigate this problem by creating an example file automatically (based on your `.env`/`.envrc` file), so the example file would become the following example:

```bash
OPENAI_KEY=""
S3_BUCKET_NAME=""
```

And for `.envrc` an `.envrc.example` file:
```bash
export OPENAI_KEY=""
export S3_BUCKET_NAME=""
```

[^1]: https://www.zdnet.com/article/botnets-have-been-silently-mass-scanning-the-internet-for-unsecured-env-files/
 
## As a pre-commit hook

See [pre-commit](https://github.com/pre-commit/pre-commit) for instructions

Sample `.pre-commit-config.yaml`

```yaml
...
-   repo: https://github.com/joooostb/clean-direnv
    rev: v0.0.7
    hooks:
    -   id: clean-dotenv
...
```

## What does it do?
The tool looks for `.env` and `.envrc` files in all directories and creates a new, corresponding filename `.env.example` which is save to commit, since it contains all the keys from your `.env`/`.encrc` file, but without its values.

As a result, you always have an up-to-date `.env.example` or `.envrc.example` file. This shall help to reduce forgetting updating the `.example` files!

## Technical Background
Since a `.env` file is probably in the `.gitignore` file, we cannot rely on pre-commits [`files`-filter](https://pre-commit.com/#hooks-files). Instead, we tell pre-commit to run always. We then check for each subdirectory if an `.env` or `.envrc` file exists. If it exists, we automatically create an `.example` file.
