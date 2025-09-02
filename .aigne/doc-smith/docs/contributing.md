# Contributing

Thank you for your interest in contributing to FastAPI! Community contributions are the lifeblood of the project, and there are many ways to get involved, from writing code and improving documentation to reporting security issues. This guide provides an overview of the tools and processes used in the project's maintenance.

For specific guidelines on development, please read the official [Development - Contributing](https://fastapi.tiangolo.com/contributing/) documentation.

## Reporting Security Vulnerabilities

Security is a top priority for FastAPI and its community. If you believe you have found a security vulnerability, please follow these steps to report it responsibly.

**Please do not discuss potential security vulnerabilities in public forums.** It's crucial to discuss and resolve the issue privately to limit any potential impact.

1.  **Report Immediately**: Send an email directly to `security@tiangolo.com`. Please be as explicit as possible, describing the steps and providing example code to reproduce the issue.
2.  **Latest Versions**: Ensure you are using the latest supported version of FastAPI. We encourage you to write tests for your application and update your FastAPI version frequently to benefit from the latest features, bug fixes, and security patches.
3.  **Review**: The author ([@tiangolo](https://x.com/tiangolo)) will thoroughly review your report and get back to you.

Your help in maintaining the security of FastAPI is greatly appreciated.

## Project Scripts and Maintenance

The project uses a collection of scripts to automate various maintenance tasks, including managing documentation, translations, and generating lists of contributors and sponsors. Understanding these scripts can be helpful if you plan to contribute to the project's infrastructure.

### Documentation Management

The documentation site is managed using a set of commands available in the `scripts/docs.py` file. These tools help create new language translations, build the site, and serve it locally for development.

Here are some of the key commands:

| Command | Description |
|---|---|
| `new-lang <LANG>` | Initializes a new documentation translation directory for the specified language. |
| `build-lang <LANG>` | Builds the documentation for a single language. |
| `build-all` | Builds the documentation for all available languages. The final site is placed in the `./site/` directory. |
| `live <LANG>` | Serves a specific language's documentation with livereload for development. Defaults to `en`. |
| `serve` | Runs a simple server to preview a fully built site with all translations. You must run `build-all` first. |
| `verify-docs` | Runs a series of checks to ensure the README, configuration, and translation files are valid and up-to-date. |

### Automated Translation

The project leverages automation to help create and update documentation translations. The `scripts/translate.py` script uses AI to generate initial translations and update existing ones when the source English content changes.

Key commands include:

| Command | Description |
|---|---|
| `translate-page` | Translates a single English documentation file to a target language. |
| `add-missing <LANG>` | Finds and translates all English documentation files that are missing in the target language. |
| `update-outdated <LANG>` | Finds and updates translations that are older than their corresponding English source files. |
| `update-and-add <LANG>` | A convenient command that runs both `update-outdated` and `add-missing` for a language. |
| `make-pr` | Automates the process of committing translation changes and creating a pull request on GitHub. |

### Community Data Generation

Several scripts automate the generation of data files used in the documentation to recognize community members. These scripts query the GitHub GraphQL API to fetch up-to-date information.

- **Contributors (`scripts/contributors.py`)**: Generates lists of code contributors, translators, and translation reviewers by analyzing pull requests.
- **Sponsors (`scripts/sponsors.py`)**: Fetches the list of GitHub Sponsors to keep the sponsor data current.
- **Experts (`scripts/people.py`)**: Identifies community experts by analyzing activity in GitHub Discussions, particularly in the Q&A category.

These automated processes ensure that contributions are consistently and accurately recognized.