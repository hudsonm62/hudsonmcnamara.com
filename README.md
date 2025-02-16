# My Website

Static site & blog written in [Markdown](https://www.markdownguide.org/), built on [Hugo](https://gohugo.io/) & [Blowfish](https://blowfish.page/) - Hosted with GitHub Pages.

## Developing

Run the [DevContainer](.devcontainer/) in VSCode (requires Docker) or in a [GitHub Codespace](https://github.com/codespaces) - It's pre-configured with the required dependencies.

Theme downloaded/installed automatically via Hugo Module - Which needs internet.

```bash
# localhost:1313
npm run serve
```

Use [Prettier](https://prettier.io/) & [markdownlint](https://github.com/DavidAnson/markdownlint) to check changes:

```bash
npm run lint

npm run lint:write
```

Lifecycle Scripts accessible via [VSCode Editor Tasks](https://code.visualstudio.com/docs/editor/tasks) and [`package.json`](./package.json).
