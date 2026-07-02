# GitHub Pages URL Notes

## What changed

The repository was renamed from `java-learn` to `java`.

For a GitHub Pages project site, the public path is based on the repository
name:

- Old repo: `java-learn`
- Old URL: `https://piyushkapoor.me/java-learn/`
- New repo: `java`
- New URL: `https://piyushkapoor.me/java/`

## Why the slash matters

`/java/` means "open the index page inside the java project folder".

`/java` normally redirects to `/java/`. This is the same pattern as opening a
folder on a static website: the final slash tells the server to load
`index.html` from that folder.

So the canonical URL should be:

```text
https://piyushkapoor.me/java/
```

## Why `.nojekyll` exists

This repository is a plain static website. It already has an `index.html` file,
so GitHub Pages does not need to run Jekyll.

The `.nojekyll` file tells GitHub Pages:

```text
Serve these files directly as static files.
```

That keeps the deployment simple and avoids Jekyll build problems for a site
that does not use Jekyll features.
