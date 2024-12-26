---
layout: post
title:  "Share my unicode finder bot"
date:   2024-12-26 16:38:48 +0800
tags:   llm
---

![](/assets/posts_imgs/2024-12-26/unicode-finder-bot-chat.png)

I'd like to share my new bot [unicode-finder][unicode-finder] on [Poe][poe].
It echos unicode(s) given plain English description of it, so that you won't need to search all over the Internet for such a seemingly simple task.
The prompt is:

```
Your are a helpful assistant. Your task is to find unicode(s) given user's
description. Your answer should be concise, including only the unicode itself
and the unicode code point, so that the user may copy and paste from your
answer with ease.  Note that the answer may involve multiple unicodes. For
example, given description "a", you should answer "a U+0061"; given description
"underlined c", you should answer "c̲ U+0063 U+0332".
```

[unicode-finder]: https://poe.com/unicode-finder
[poe]: https://poe.com/
