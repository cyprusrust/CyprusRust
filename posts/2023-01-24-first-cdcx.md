---
title: We talked Rust at the first CDCx ever!
description: A brief recap of our appeareance at the CDCx conference
slug: first-cdcx
published_date: 2023-01-24 04:16:22 +0000
layout: post.liquid
is_draft: false
data: {
  nice_date: '23th January 2023, Paphos',
  avatar: 'https://gravatar.com/avatar/21fc27a2ac6cd9094a423997f0344a0b?s=256',
  avatarAlt: 'Federico Rampazzo looking like a smug insurance salesman',
  author: 'Federico Rampazzo',
  bio: 'Federico is a software engineer with a passion for Functional Programming, Category Theory and Strongly Typed languages.',
  contact: 'https://framp.me',
}
---

<hgroup>

### {{ page.title }}

#### {{ page.data.nice_date }}

</hgroup>

&nbsp;

Last weekend we had the pleasure to give [a talk](https://cyprusrust.github.io/rust-backend-slides/presenter/1) about our experience writing Backend Services in Rust at the first [CDCx](https://cdc.cy) conference, hosted in Paphos.

We were joined by [CyprusJS](https://cyprusjs.org/) who delivered a brilliant talk on [Solid.js](https://solidjs.com/).

It was great to meet all of you in person!

We've been playing with [Axum](https://github.com/tokio-rs/axum/) and [sqlx](https://github.com/launchbadge/sqlx) for a few versions now and we're very pleased with the workflow and with how fast and safely we're able to iterate.

Our services proved to be stable, use little memory and let us sleep peacefully at night, knowing that silly mistakes are kept at bay by Rust's compile-time checks.

If you want to have a look at a simple production application, we decided to open source [one of our services](https://github.com/apiplant/anonpaste-backend). Feedbacks, contributions and discussions are highly appreciated!

I think that Rust web ecosystem is mature enough to allow you to build productive applications.

Why not give it a go next time you start a new project?

If you need support feel free to come and drop a message on our [Discord](https://discord.gg/3xKSyZM4mB)!