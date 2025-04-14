---
title: Why Bluesky's ATProtocol Sucks
date: 2025-04-09
draft: false
description: "Why Bluesky's ATProtocol Sucks - An opinion piece of its flaws and why it is currently not a truly decentralized social media platform (in practice)."
summary: "Opinion on the flaws of Bluesky & ATProtocol in practice."
tags:
  - Decentralization
  - Social Media
  - Opinion
---

## Quick Rundown

Bluesky's ATProtocol is a "[Distributed Social Networking Protocol](https://en.wikipedia.org/wiki/Distributed_Social_Networking_Protocol)" - _Essentially_, it's a way to create a social media platform that is **not controlled by a single entity**. This sounds great in theory (as [ActivityPub](https://en.wikipedia.org/wiki/ActivityPub) & [Mastodon] is proving), but the ATProtocol in practice has some serious _and seemingly intentional_ flaws.

A lot of people may not agree with me on a technical standpoint, but keep in mind there are more non-technical users than there are technically adept -- Just because it does this cool encryption thing A, or Identity thing B, which other protocols don't do (or at least not in the same way), doesn't mean it's a good idea for the average person and the 'industry' of decentralized social networking at a high level.

I will be focusing on the **practical** implications of ATProtocol, and why I think it is not a good idea to use it (especially as a replacement for ActivityPub).

## 1. NOT Decentralized (for the most part)

Assuming you're familiar with the concept--When you think of a decentralized social media platform, you probably think of something like [Mastodon] or [Lemmy](https://join-lemmy.org/). In Mastodon, you can run your own server (aka "instance") and connect to other servers, meaning said servers own and moderate their own content, contributing to the larger picture (via ActivityPub). This is **true** decentralization - Full control over data and how it's shared across the respective protocols [Fediverse](https://en.wikipedia.org/wiki/Fediverse).

Whilst [ATProtocol allows you to run your own server](https://atproto.com/guides/self-hosting) (called a Personal Data Server in ATP terms), it is **not** decentralized in the same way. Right now, you can only connect your PDS to Bluesky's servers, meaning you are still reliant on Bluesky to connect to other servers. Any "competitors" currently using the ATProtocol, is largely incompatible with Bluesky itself and the rest of the "ecosystem".

This means that Bluesky (and competitors) can still control the protocol and how it's used, which defeats the purpose entirely.

Perhaps this might change in the future, but as it stands, ATProtocol is not truly decentralized, regardless of what Bluesky claims.

## 2. Exclusivity

Bluesky is very much an exclusive platform, for Bluesky users only. You can't just create an account on a random server/instance of your choosing, and connect to Bluesky -- You have to sign up TO Bluesky itself. Again, this is not decentralization, this is just a different way of doing things under the guise of its umbrella.
The reason ActivityPub works so well, is because each server/instance is **independent** AND _mostly_ compatible with/of each other. ATProtocol is not like this, and it is not designed to be like this. It is designed to be a closed ecosystem, where Bluesky can control/moderate everything that happens on it. Your **personal data server is just a glorified backup of your Bluesky account**, and you are still reliant on Bluesky to connect to other servers.

What happens if Bluesky decides to shut down _tomorrow_? Your PDS is nearly useless, and you have no way of connecting to other servers.

## 3. Lack of Interoperability

ATProtocol is not interoperable with anything, meaning you can't connect to other social media platforms like Mastodon. This is a huge problem, as it means you are **stuck** in their ecosystems bubble, and you can't connect with other users on different platforms.

Even [Meta Threads](https://www.threads.net) is more interoperable than ATProtocol now, as they are starting to [support ActivityPub federation](https://engineering.fb.com/2024/03/21/networking-traffic/threads-has-entered-the-fediverse/) for public accounts, which means you can follow your friends/family (assuming they've opted in) from your Mastodon or other ActivityPub instances.

Whilst there are third-party clients to support both, they are not the same as being able to connect to other platforms.

## 4. Pure Arrogance

This is where I get a bit more opinionated than before, but I think it's important to note.

ActivityPub has been around for a while now, and it has a huge community of developers and users behind it with nearly a decade of history. So why would you want to create a new protocol that is not compatible with it?

[ATProtocol argues it's own points](https://atproto.com/guides/faq#why-not-use-activity-pub), and they want to create something new and better. **But in reality**, it's just a way for them to control the narrative and keep users locked into their ecosystem.

Think about it, if you don't like the way an **open-source** product works, why not contribute back to try and make it better? It feels like Bluesky is stamping their feet and saying "I don't like this, so I'm going to create my own thing".

## Final Remarks

**ATProtocol is not a new idea**, and (in my opinion) **it is not a better idea**. They should have just contributed to ActivityPub's spec (and ultimately the Fediverse), instead of trying to capitalize on their own product. It is a **purely corporate move**, and it is not in the best interest of the users.

At the end of the day, Bluesky is simply _another_ "Alternative to Twitter", and **cannot be compared to other Fediverse platforms** like [Mastodon], [Pleroma](https://pleroma.social/), or [Pixelfed](https://pixelfed.org/) (which are all ActivityPub compatible). Until there are **more** ATProtocol compatible platforms where I can connect a PDS to, I will not be using or recommending it. Keep in mind that ATProtocol is still in it's early stages, but lack of adoption outside of Bluesky itself is a huge red flag.
At the end of the day, Bluesky is simply _another_ "Alternative to Twitter", and **cannot be compared to other Fediverse platforms** like [Mastodon], [Pleroma](https://pleroma.social/), or [Pixelfed](https://pixelfed.org/) (which are all ActivityPub compatible). Until there are **more** ATProtocol compatible platforms where I can connect a PDS to, I will not be using or recommending it. Keep in mind that ATProtocol is still in its early stages, but lack of adoption outside of Bluesky itself is a huge red flag.

For the non-technical & non-caring, Bluesky is probably the best option for you -- But for the tech-savvy and high-profile users, I would personally recommend looking into [Mastodon](https://joinmastodon.org/) or other ActivityPub compatible platforms.

## Further Reading & Resources

- [Official ATProtocol Overview](https://atproto.com/guides/overview)
- [HackerNews Forum | Okay well. I work on Bluesky and helped build the AT Protocol.](https://news.ycombinator.com/item?id=35881905)
- [bluesky-social/atproto | #1716 - Interoperability with Mastodon/ActivityPub](https://github.com/bluesky-social/atproto/discussions/1716)
- [Bridgy Fed](https://brid.gy)
- [Justin McAfee | Comparative Analysis of Decentralized Social Protocols](https://medium.com/1kxnetwork/a-comparative-analysis-of-decentralized-social-protocols-84914d9fca83)
- [Dennis Schubert | ActivityPub - One protocol to rule them all?](https://overengineer.dev/blog/2018/02/01/activitypub-one-protocol-to-rule-them-all/)

[Mastodon]: https://joinmastodon.org/

Article Photo by [Elena Rossini](https://unsplash.com/@elenarossini) on [Unsplash](https://unsplash.com/photos/9Xf-jxvfpW8?&utm_medium=referral&utm_source=unsplash)
