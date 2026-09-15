# markbhall.dev

Personal site and technical writing for Mark Hall, built with Hugo and deployed
to GitHub Pages.

## Local preview

```powershell
hugo server
```

The production site is published from `main` by the GitHub Pages workflow.

## Stillwater

The interactive Three.js village is served at [/stillwater/](https://markbhall.dev/stillwater/).
It showcases [Reference Asset Compiler](https://github.com/raydeStar/reference-asset-compiler),
with the original reference, a four-step account of model creation, an explanation
of the compiler's role, and links to the pipeline and workflow documentation.
`static/stillwater/` contains its self-contained Vite build, seven original GLBs
and seven browser derivatives with smaller textures,
references, bundled fonts and licenses. Hugo copies this directory into the
Pages artifact without a separate application server.

The editable scene is maintained in `reference-asset-compiler/apps/lakeside-village`.
To refresh this deployment, restore the accepted local assets, run `npm ci`,
`node prepare_web_assets.mjs` and `npm run build` there, then copy
`dist/index.html`, `dist/reference.png`, `dist/reference.webp`,
`dist/stillwater-*.jpg`, `dist/stillwater-*.webp`, `dist/assets/` and `dist/licenses/`
into `static/stillwater/`. Exclude the internal `review.html`, `review/` images
and `assets/review-*.js`. Keep the Vite base relative (`./`) and update
`release.json` with the source, original/derivative GLB and media hashes. The
1200 × 627 social card and homepage teaser are real scene captures. The viewer
shares its model library with the scene and initializes when visible; original
models and full-size reference images are fetched only on request. Build this site with
`hugo --gc --minify` and test the `/stillwater/` path before pushing to `main`.
