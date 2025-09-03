# Contributing

Thank you for your interest in contributing to FastAPI! Your help is essential for keeping it great. This document will guide you on how to contribute, report security issues, and understand the project's internal scripts and maintenance.

There are several ways you can contribute to the project:

<x-cards data-columns="3">
  <x-card data-title="Development Guidelines" data-icon="lucide:code">
    Follow our development process for code and documentation contributions.
  </x-card>
  <x-card data-title="Security Policy" data-icon="lucide:shield">
    Learn how to report security vulnerabilities responsibly.
  </x-card>
  <x-card data-title="Project Maintenance" data-icon="lucide:bot">
    Understand the internal scripts that help maintain the project.
  </x-card>
</x-cards>

## Development Guidelines

For general guidelines on setting up your development environment, running tests, and submitting pull requests, please read the official [Development - Contributing](https://fastapi.tiangolo.com/contributing/) guide on the documentation site. It provides all the necessary information to get you started.

## Security Policy

Security is very important for FastAPI and its community. Here's how we handle it.

### Supported Versions

The latest version of FastAPI is always the supported version. You are encouraged to write tests for your application and update your FastAPI version frequently after ensuring your tests pass. This way you will benefit from the latest features, bug fixes, and **security fixes**.

You can learn more about [FastAPI versions and how to manage them](https://fastapi.tiangolo.com/deployment/versions/) in the documentation.

### Reporting a Vulnerability

If you think you have found a vulnerability, even if you are not sure about it, please report it right away by sending an email to `security@tiangolo.com`. Please try to be as explicit as possible, describing all the steps and providing example code to reproduce the security issue. The author will review it thoroughly and get back to you.

### Public Discussions

Please restrain from publicly discussing a potential security vulnerability. It's better to discuss it privately to find a solution first, which helps limit the potential impact as much as possible.

## Project Maintenance and Scripts

FastAPI uses a collection of internal scripts to automate project maintenance, including documentation management, translations, and updating community-related data. Understanding these scripts can be helpful if you plan to contribute to the project's infrastructure.

Here is a summary of the main scripts:

| Script File | Purpose |
|---|---|
| `docs.py` | Manages the documentation site, including building for multiple languages, creating new language setups, and serving locally. |
| `translate.py` | Automates the translation of documentation content using AI, including updating outdated and adding missing translations. |
| `people.py` | Gathers and updates data about community experts from GitHub Discussions. |
| `contributors.py` | Collects data on code contributors, translators, and reviewers from GitHub Pull Requests. |
| `sponsors.py` | Fetches and updates the list of GitHub Sponsors to recognize their support. |

### Documentation Management (`docs.py`)

This script is the main tool for handling the MkDocs-based documentation. Its key functions include:

- **Building the site**: The `build-lang` and `build-all` commands compile the markdown files into a static website for one or all languages.
- **Creating new translations**: The `new-lang` command sets up the necessary directory structure and configuration files for a new language translation.
- **Local development**: The `serve` and `live` commands provide a local server with livereload for previewing changes as you work on the documentation.
- **Verification**: Commands like `verify-readme` and `verify-config` ensure that generated files like the main `README.md` are up-to-date with the documentation content.

### Translation Management (`translate.py`)

To facilitate the translation of documentation into multiple languages, this script leverages AI to automate the process. It can:

- **Translate a single page**: Takes an English source file and generates its translation for a specified language.
- **Update outdated translations**: It can check if the source English file has been updated more recently than the translated file and trigger a re-translation to keep content synchronized.
- **Add missing translations**: The script can identify English documents that do not yet have a translation in a specific language and generate them.

### Community Data Automation

Several scripts work together to keep the **FastAPI People** section of the website current by automatically fetching data from GitHub:

- **`people.py`**: This script analyzes GitHub Discussions in the "Questions" category to identify active and helpful community members, recognizing them as "Experts". It categorizes them based on their activity over different time periods (last month, three months, etc.).
- **`contributors.py`**: It queries GitHub's API for all pull requests to build a list of contributors. It distinguishes between code contributors, translators, and translation reviewers based on PR labels and review activity.
- **`sponsors.py`**: This script fetches the latest list of sponsors from the official `tiangolo` GitHub Sponsors account, grouping them by sponsorship tier. This ensures that everyone who supports the project is properly acknowledged.

These automation scripts help reduce manual maintenance and ensure that community contributions are recognized in a timely manner. If you wish to help improve these processes, you can find the scripts in the `scripts/` directory of the project repository.