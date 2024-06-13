---
layout: post
title:  "Leverage Ollama in iTerm2 AI integration"
date:   2024-06-13 22:46:53 +0800
tags:   os--macos
---

## Introduction

Recently, [iTerm2](https://iterm2.com/) released version [3.5.0](https://iterm2.com/downloads/stable/iTerm2-3_5_0.changelog), which includes generative AI integration in OpenAI API.
[Ollama](https://ollama.com/) is an open platform for large language models (LLM).
Starting from February 2024, Ollama has built-in [support](https://ollama.com/blog/openai-compatibility) of OpenAI chat completions API.
Putting them together, we can [now](https://gitlab.com/gnachman/iterm2/-/issues/11455) ask AI to compose commands for us seamlessly in iTerm2 interface, using Ollama bot locally.

## Configuration

Here are the steps to start using the AI integration in iTerm2:

1. Install the AI plugin from [iTerm2 site](https://iterm2.com/ai-plugin.html).
2. In iTerm2 preferences, under `General` section and `AI` tab, enter "OpenAI API key" with anything non-empty, fill in the [AI prompt](https://gitlab.com/gnachman/iterm2/-/wikis/AI-Prompt), specify the model and the custom URL.

For example, mine is like below:

- OpenAI API key: `abc`
- AI prompt: `Return commands suitable for copy/pasting into \(shell) on \(uname). Do NOT include commentary NOR Markdown triple-backtick code blocks as your whole response will be copied into my terminal automatically. If not otherwise specified, you should always give at most one line of command. The command should do this: \(ai.prompt)`.
- Model: `codegemma:instruct`.
- Token limit: `16384`.
- Custom URL: `http://localhost/v1/chat/completions`.
- Use legacy "completions" API: false.

Remarks:

- If your Ollama runs on a server in WLAN, e.g. at IP address `192.168.0.107`, just replace the `localhost` in custom URL with that IP address.
- Don't forget to start Ollama by `ollama serve` before using iTerm2's AI integration.

## Workflow

My favorite iTerm2 workflow after the configuration above:

1. Press `command + shift + .` to activate the composer.
2. Specify my need in plain English, and press `command + y` to send the input text to Ollama.
3. After a few seconds, the text should be replaced by Ollama's response.
4. Press `shift + enter` to send the response to the terminal.

A demo:

![demo](/assets/posts_imgs/2024-06-13/iterm2-ai-demo.gif)
