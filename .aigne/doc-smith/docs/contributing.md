# Contributing

Thank you for your interest in contributing to FastAPI! Your help is essential for keeping the project great. Whether you're fixing a bug, improving the documentation, or adding a new feature, your contributions are welcome.

This document provides an overview of how you can contribute. For detailed instructions, please refer to the official guidelines.

<x-card data-title="Development Contribution Guidelines" data-icon="lucide:book-open-check" data-href="https://fastapi.tiangolo.com/contributing/" data-cta="Read the Full Guide">
The primary guide for all development contributions, including code, documentation, and issue reporting, is located on the official documentation site. Please read it carefully before starting.
</x-card>

## Security Policy

Security is a top priority for FastAPI. If you believe you have found a security vulnerability, we appreciate your help in disclosing it to us responsibly.

Please do not discuss potential security vulnerabilities publicly. Instead, report them privately by sending an email to `security@tiangolo.com`.

For more detailed information on supported versions and the full reporting process, please see our dedicated [Security Policy](./contributing-security-policy.md).

## Translations

Contributing translations is another fantastic way to help the global FastAPI community. The project uses a sophisticated set of scripts to manage the translation workflow, ensuring that content for different languages stays up-to-date.

This process includes tools for adding new pages, updating outdated content, and automatically creating pull requests. This makes it easier for translators to focus on the content itself.

```python Translation Management Script icon=logos:python
# Example commands from the translation management script

@app.command()
def update_outdated(language: Annotated[str, typer.Option(envvar="LANGUAGE")]) -> None:
    outdated_paths = list_outdated(language)
    for path in outdated_paths:
        print(f"Updating lang: {language} path: {path}")
        translate_page(language=language, en_path=path)
        print(f"Done updating: {path}")
    print("Done updating all outdated paths")


@app.command()
def add_missing(language: Annotated[str, typer.Option(envvar="LANGUAGE")]) -> None:
    missing_paths = list_missing(language)
    for path in missing_paths:
        print(f"Adding lang: {language} path: {path}")
        translate_page(language=language, en_path=path)
        print(f"Done adding: {path}")
    print("Done adding all missing paths")
```

## Contributor Recognition

We value all contributions and have an automated process to recognize top contributors, translators, and translation reviewers. This system analyzes pull request activity to generate lists of community members who have made significant contributions to the project.

Your efforts help make FastAPI better for everyone, and we believe in giving credit where it's due.

We look forward to your contributions!