# go-xrkit docs

Versioned documentation for [go-xrkit](https://github.com/go-xrkit/xrkit), the
pure-Go geometry an immersive video player needs. Built with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and versioned
with [mike](https://github.com/jimporter/mike); published to the `gh-pages`
branch and served at **<https://go-xrkit.github.io/docs/>**.

## Local preview

```sh
pip install -r requirements.txt
mkdocs serve
```

## Publishing

Pushing to `main` runs `.github/workflows/docs.yml`, which deploys the versioned
site with `mike deploy --push --update-aliases 0.1 latest` and sets `latest` as
the default. `/docs/` redirects to the default version.

## License

BSD-3-Clause — see [LICENSE](LICENSE). Copyright the go-xrkit/docs authors.
