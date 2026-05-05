# pablosproject.com

Based on [Hugo Researcher](https://github.com/ojroques/hugo-researcher) which in turn is based on the Jekyll theme [Researcher](https://github.com/ankitsultana/researcher).

## Analytics

This site is prepared for **Cloudflare Web Analytics**.

- **Cloudflare Pages automatic setup**: no code changes needed, enable it in **Workers & Pages → your project → Metrics → Enable**.
- **Manual setup**: paste your token in `config.toml` under `params.analytics.cloudflareToken`.

The theme will load the Cloudflare beacon only when `cloudflareToken` is set.

## Cloudflare Pages migration

Recommended Cloudflare Pages settings for this Hugo site:

- **Production branch**: `main`
- **Build command**: `hugo`
- **Build output directory**: `public`
- **Environment variable**: `HUGO_VERSION=0.161.1`

Migration checklist:

1. Create a **Pages** project from this GitHub repo in Cloudflare.
2. Use the build settings above.
3. Verify the generated `*.pages.dev` preview works.
4. In **Workers & Pages → your project → Custom domains**, add `pablosproject.com`.
5. Update DNS to Cloudflare when ready.
6. Enable **Web Analytics** in the project **Metrics** tab.
7. After traffic is serving correctly from Cloudflare, remove the Netlify site.

## Useful command

- **Serve**: `hugo server -D` (-D for drafts)
- **Build**: `hugo -D`
- **Add a post**: `hugo new posts/POST.md`

## Installation
This theme uses Sass to generate CSS files so make sure you have the
*extended* Hugo version installed.

Add the theme to your site's `themes` directory:
```bash
git submodule add https://github.com/ojroques/hugo-researcher.git themes/researcher
# if your website is not managed by git:
# git clone https://github.com/ojroques/hugo-researcher.git themes/researcher
```

Update the theme option in `config.toml`:
```toml
theme = "researcher"
```

## Configuration
A self-explanatory configuration file is present in
[exampleSite/config.toml](https://github.com/ojroques/hugo-researcher/blob/master/exampleSite/config.toml),
along the files of a demo website.

## KaTeX
You can enable [KaTeX](https://katex.org/) (math typesetting) by including
`math: true` in your content files. Or you can enable it globally by setting
`math` to `true` in your project config.

Hugo introduces tags when it sees newlines which breaks KaTeX block
environments. The theme has a `math` shortcode to circumvent this issue:
```md
{{< math >}}
\begin{pmatrix}
a & b \\
c & d
\end{pmatrix}
{{< /math >}}
```
Check [this
issue](https://github.com/ojroques/hugo-researcher/issues/1#issuecomment-697247056)
for more details.
