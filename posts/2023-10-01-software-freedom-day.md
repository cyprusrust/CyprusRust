---
title: AI in Rust and the Burn framework!
description: Federico talked about influential AI projects in Rust
slug: software-freedom-day
published_date: 2024-07-20 16:01:48 +0000
layout: post.liquid
is_draft: false
data: {
  nice_date: '20th July 2024, Paphos',
  avatar: 'https://gravatar.com/avatar/21fc27a2ac6cd9094a423997f0344a0b?s=256',
  avatarAlt: 'Federico Rampazzo looking like an Indiana Jones impersonator',
  author: 'Federico Rampazzo',
  bio: 'Federico is a software engineer with a passion for Functional Programming, Category Theory and Strongly Typed languages.',
  contact: 'https://apiplant.com',
}
---

<hgroup>

### {{ page.title }}

#### {{ page.data.nice_date }}

</hgroup>

&nbsp;

Last week we had the pleasure to partecipate [Software Freedom Day](https://wiki.softwarefreedomday.org/2023/Cyprus/Paphos/CDC). We beat all our affluence records and we had 3 great talks discussing Open Source.

Of course, I came up with a creative idea to include Rust in the discussion and prepared [a talk](https://cyprusrust.github.io/generative-ai-slides) about my experience using AI and trying to avoid Python.

Even though the ecosystem is still in its early days, you can use [tch-rs](https://github.com/LaurentMazare/tch-rs) to access torchlib (the very same library which powers Pytorch) and implement plenty of popular models (with a bit of effort).

[Candle](https://github.com/huggingface/candle) is definitely the project to keep an eye on; it's an machine learning framework written in Rust which wants to be minimal and ready for serverless.

It's also growing at a rapid pace: since my presentation, it already gained a `candle-transformers` crate which wants to mirror the popular Python package [transformers](https://github.com/huggingface/transformers).

We also talked about fake OSS licenses like [HuggingFace's OpenRAIL](https://huggingface.co/blog/open_rail), we worried about [European regulations](https://digital-strategy.ec.europa.eu/en/policies/cyber-resilience-act) destroying Open Source software and we discussed the pro and cons of having a personal Open Source project. What a day!

I was blown away by the quality of the talks.
Want to check them out too? Here they are!

[![Generative AI in Rust](http://i3.ytimg.com/vi/m-2-0a1W2kc/hqdefault.jpg)](https://www.youtube.com/watch?v=m-2-0a1W2kc&list=PLmam5HsEG8IhuTli5mf93Wwb6ijYsayA7)

[![Pro & Cons of personal OSS Projects](http://i3.ytimg.com/vi/LCFX4XaUAAU/hqdefault.jpg)](https://www.youtube.com/watch?v=LCFX4XaUAAU&list=PLmam5HsEG8IhuTli5mf93Wwb6ijYsayA7&index=2)

[![Generative AI in Rust](http://i3.ytimg.com/vi/gZYtEGBp1VA/hqdefault.jpg)](https://www.youtube.com/watch?v=gZYtEGBp1VA&list=PLmam5HsEG8IhuTli5mf93Wwb6ijYsayA7&index=3)

As always, if you need support or if you want to chat with the community, drop a message on our [Discord](https://discord.gg/3xKSyZM4mB)!