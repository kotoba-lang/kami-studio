# kami-studio

The **Kami Studio hub page** — a static, link-out landing page for the separate
`kami-app-*` 3D editors. It is *not* an editor itself and holds no scene,
geometry, or document state; every card on the page is an external link.

## What this repo actually contains

| Path | Role |
|---|---|
| `src/kami/studio/ui.cljc` | The whole page: the `apps` link table, a `css/css` stylesheet, and `page` returning hiccup |
| `build.clj` | Two forms — `(spit "public/index.html" (ui/page))` |
| `public/index.html` | **Committed generated output.** Do not hand-edit; regenerate from `ui.cljc` |
| `deps.edn` | `kotoba-lang/html` + `kotoba-lang/css`, pinned by `:git/sha` |

There is no reagent, no bundler, and no runtime JavaScript — `page` renders to a
single HTML document at build time.

## The links

Seven cards resolve to six apps: Modeler, Animator, BIM Editor, CAD, Sculpt, and
Amenominaka. **Vehicle Physics is not a seventh app** — it is `kami-app-modeler`
opened at `?workspace=vehicle-physics`. All seven point at
`kotoba-lang.github.io/kami-app-*`.

To add or move a card, edit the `apps` vector in `src/kami/studio/ui.cljc` and
**rebuild** (see below). Nothing else reads that table.

## Build and deploy — the generated file is the deployed file

```
clojure -M build.clj      # rewrites public/index.html from src/kami/studio/ui.cljc
```

`.github/workflows/pages.yml` **does not run this build.** It runs
`upload-pages-artifact` with `path: public` and publishes whatever `public/` is
committed on `main`. So the deploy step cannot notice that `index.html` is stale:

> Editing `ui.cljc` without rebuilding changes nothing about the live site, and
> nothing fails. The page still renders — it just renders the older link table.

This has already happened once. `5192750` added the Vehicle Physics card to
`ui.cljc`; `public/index.html` was last written by `2ef356b`, before it. The live
page served six cards while the source described seven, and no check was in a
position to say so. Regenerating is what fixed it.

**Therefore: any commit touching `ui.cljc` must include the regenerated
`public/index.html` in the same commit.** Reviewing the source alone is not
enough to know what is deployed.

## Boundary

Per the kami stack rule (`com-junkawasaki/root` CLAUDE.md, 3D section), the
authority for geometry, scene, animation, and rendering lives in `kami-engine-*`
and the `webgpu` / `webgl` libraries — never in an app or in this hub. This repo
owns exactly one thing: which apps exist and how to reach them.

Styling is hand-written CSS in `ui.cljc` predating the `jp-go-dds` design-system
rule; new UI in this workspace starts from `jp-go-dds`, so treat this stylesheet
as legacy rather than as a pattern to copy.

## Known gap

No test signal. Nothing asserts that `page` renders, that `public/index.html`
matches what `build.clj` would write, or that the `kami-app-*` URLs resolve. The
staleness above is the shape this gap takes: the cheapest check would rebuild
into a temp file and diff it against the committed `public/index.html`.

## License

Apache-2.0 (see `LICENSE`).
