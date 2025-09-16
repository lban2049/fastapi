# 贡献

感谢您有兴趣为 FastAPI 做出贡献！您的帮助对于保持项目的出色至关重要。无论是修复漏洞、改进文档还是添加新功能，我们都欢迎您的贡献。

本文档概述了您的贡献方式。有关详细说明，请参阅官方指南。

<x-card data-title="开发贡献指南" data-icon="lucide:book-open-check" data-href="https://fastapi.tiangolo.com/contributing/" data-cta="阅读完整指南">
所有开发贡献（包括代码、文档和问题报告）的主要指南均位于官方文档网站上。请在开始前仔细阅读。
</x-card>

## 安全策略

安全是 FastAPI 的重中之重。如果您认为自己发现了安全漏洞，我们非常感谢您以负责任的方式向我们披露。

请不要公开讨论潜在的安全漏洞。请发送电子邮件至 `security@tiangolo.com` 进行私下报告。

有关受支持版本和完整报告流程的更多详细信息，请参阅我们专门的[安全策略](./contributing-security-policy.md)。

## 翻译

贡献翻译是帮助全球 FastAPI 社区的另一种绝佳方式。项目使用一套复杂的脚本来管理翻译工作流程，以确保不同语言的内容保持最新。

该流程包含用于添加新页面、更新过时内容以及自动创建拉取请求的工具，使翻译人员可以更轻松地专注于内容本身。

```python Translation Management Script icon=logos:python
# 翻译管理脚本中的示例命令

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

## 贡献者致谢

我们重视所有贡献，并通过自动化流程来表彰杰出的贡献者、翻译者和翻译审校者。该系统会分析拉取请求活动，以生成为项目做出重大贡献的社区成员列表。

您的努力使 FastAPI 变得更好，我们坚信应当给予应有的认可。

我们期待您的贡献！