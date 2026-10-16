# florenzo-sec.github.io

Personal blog on cybersecurity, networking and Hack The Box. Built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme (MIT, bundled in `themes/PaperMod`), published with GitHub Pages through GitHub Actions.

## Writing a new post

```bash
hugo new content posts/htb-machine-name.md
```

Edit the file in `content/posts/`, then `git push` to `main`: the workflow builds and publishes the site.

## Local preview

```bash
hugo server -D
```

## Publishing rules

- Never publish the flag, and never show the target or VPN IP in screenshots.
- Check Hack The Box's write-up policy before publishing.
- Raw notes belong in a private repository, not here.
