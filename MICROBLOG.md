# Ananke, ported for Micro.blog

This is Ananke **v2.12.1** (the last release before Hugo's v0.146 template-system
rewrite) adapted to work as a Micro.blog custom theme. Original theme:
https://github.com/theNewDynamic/gohugo-theme-ananke

## What changed from stock Ananke

1. **Categories instead of tags.** Micro.blog posts are tagged with
   `categories`, not `tags`. `layouts/partials/tags.html` is now
   `categories.html` and reads `.GetTerms "categories"`. The new-post
   archetype front matter was updated to match.

2. **Title-optional microposts.** Most Micro.blog posts have no title.
   Every place that used to link on `.Title` alone
   (`layouts/_default/summary.html`, `summary-with-image.html`,
   `layouts/post/summary.html`, `layouts/_default/single.html`,
   `layouts/index.html`) now:
   - only renders the `<h1>`/heading and "Read more" link when `.Title` is
     actually set, and
   - always renders the post's date as a link, so title-less posts still
     have a clickable permalink on list/summary views.

3. **Styled `/archive` page.** Micro.blog always serves a `/archive` URL for
   every blog, whether or not it's in your nav, falling back to its own
   plain "blank" theme template if your custom theme doesn't provide one.
   Added `layouts/_default/list.archivehtml.html`: a simple date-sorted
   list, titled or untitled posts both handled.

4. **Home page post source.** Micro.blog doesn't put posts in a `post/`
   content section — it sets `Type: post` in front matter on posts that
   live at the content root. `layouts/index.html` now filters
   `RegularPages` by `Type` first, falling back to `Section` if nothing
   matches (so this still works if you ever reuse it on a normal Hugo
   site with a `content/post/` folder).

## What wasn't touched

- The Tachyons-based CSS/SCSS build (`assets/ananke/...`) is untouched — it
  uses Hugo Pipes only (`css.Sass`, `resources.Concat`, `resources.Minify`),
  no npm/node build step, so it should Just Work on Micro.blog's Hugo build.
- Social share/follow icons, Disqus/Commento comments, i18n strings,
  the contact-form shortcode, and everything else are unchanged from
  upstream Ananke.
- `layouts/robots.txt` is left in place, but Micro.blog generates its own
  robots.txt for your blog by default — if you see two robots.txt behaviors
  fighting, this file is the one to check first.

## Installing on Micro.blog

1. Push this folder to its own GitHub repository.
2. In Micro.blog, go to **Design → Edit Custom Themes → Create New Theme**,
   name it something with a version marker (e.g. `ananke-mb-v1`) since
   Micro.blog does *not* auto-update from your repo on every push — you'll
   re-clone/rename when you make changes.
3. Paste in the **HTTPS** clone URL of your repo (not SSH).
4. Back on the Design page, select the new custom theme from the dropdown.
5. **Design → Hugo Version**: this theme declares `min_version = 0.128.0`
   and was tested against the pre-0.146 Hugo template system. Try Micro.blog's
   **0.91** default first; if you hit template errors, step up to whatever
   higher version Micro.blog currently offers (check Design → Hugo Version
   for the current list — it's changed a few times).
6. Click **Update Blog Settings** to publish. Errors will surface on the
   Design page — most commonly a missing file that needs to be committed,
   or (per Micro.blog's own guidance) home page not listing posts, which
   usually means the `Type`/`Section` filter in `layouts/index.html` isn't
   matching how your posts are actually stored.

## Known unknowns

I ported this by reading the templates directly — I don't have a live
Micro.blog blog or a local Hugo binary to build-test against, so treat this
as a strong first draft rather than a guaranteed-working theme. Things worth
watching for once you deploy to a test blog:

- Whether Micro.blog's actual post `Type` value is exactly `"post"` (it's
  what Micro.blog's own porting documentation recommends filtering on, but
  worth confirming against your own blog's rendered front matter).
- Whether `layouts/_default/list.archivehtml.html` is the exact filename
  Micro.blog's `archivehtml` output format looks for — this is based on a
  theme developer's public write-up, not Micro.blog's official docs.
- Whether Micro.blog's build passes `hugo.IsExtended` (needed for the Sass
  compilation) — highly likely, since Sass-based themes are common on the
  platform, but worth confirming if styles don't load.

## Fixes applied after real deployment testing

- **`_internal/pagination.html` — removed from Hugo entirely** (around
  v0.147+), not just a Micro.blog quirk. Both call sites
  (`layouts/_default/list.html`, `layouts/post/list.html`) now build
  prev/next links by hand from `.Paginator.HasPrev/HasNext/Prev/Next`
  instead of relying on the internal template.
- `layouts/_default/baseof.html` still calls three other Hugo internal
  templates: `_internal/opengraph.html`, `_internal/schema.html`,
  `_internal/twitter_cards.html`, plus `_internal/google_analytics.html`
  and `_internal/disqus.html` used elsewhere. I have no evidence these were
  removed the way `pagination.html` was, but if you see the same
  `no such template "_internal/..."` error for any of them, it's the same
  class of problem — tell me which one and I'll hand-roll a replacement the
  same way.
- **Reply posts break the author field.** Micro.blog "replies" (replies to
  a fediverse/ActivityPub post) store `author` in front matter as a
  structured object (`name`, `username`, `avatar`, `activitypub.url`)
  instead of a plain string or list. Stock Ananke assumes string/list and
  fed it straight into `transform.Markdownify`, which errors on a map.
  Fixed in `layouts/_default/single.html` and
  `layouts/_default/summary-with-image.html` to detect a map and render
  the author's name (or username as fallback) instead. If other content
  types Micro.blog generates (bookmarks, likes, etc.) have their own
  unusual front-matter shapes, the same fix pattern applies — send me the
  error and I'll patch it.

