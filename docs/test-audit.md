# Test audit

Newest first.

## 2026-11-26: 14-day check

- Changes since 2026-11-10: three code changes landed later the same day, after the entry for the security read was written. The lint strips HTML tags, and a page slug that matches a collection name gets `-page` (47c0c43). Other plugins register feature sources at `config.custom.docsSources`, in any plugin order (a2ac7d4). `checkDocs` reports two sources with one name as a problem where it used to throw (5c041c0). After those came a README section (cc95ffb), the license line (e37e7bf), the move to `@extantape` (441f503), docs edits (decf25d, 474c9e9) and the docs-only CI gate (c2e6cac), none of which touch `src/` or `test/`.
- Behavior changed in those three commits, so this was a full audit: the README's promises were mapped to tests again. 47c0c43 and a2ac7d4 came with tests (`sees through escaping…`, `allows many hand-written pages…`, `sources from other plugins > reads a source…`). 5c041c0 had none.
- Gaps added, four tests in `refusals the 14-day check added`, 21 tests before and 25 after:
  - the plugin refuses to start without both `access.edit` and `access.review`;
  - only editors make or change a page, and nobody deletes one, a reviewer and a visitor included;
  - an access answer that is truthy without being `true` is a no: such a user reads what a visitor reads, sees no versions, makes and changes nothing, and can't review even past the collection's own access;
  - two sources with one name are reported by `checkDocs` as `duplicate`, and `syncDocs` throws and makes no page while they clash.
- Mutation check: 7 one-line changes, each restored byte for byte (SHA-256 checked) and run against the full suite. All 7 fail it, and only in the new tests: the start-up check dropped; `create` open to all; `update` open to all; a truthy answer accepted; the review rule read as truthy; the duplicate-name check dropped; `checkDocs` throwing on duplicate names.
- Gaps skipped: `config.custom.docsSources` set to something that isn't a list is ignored (one `Array.isArray`, no access or data at stake). A real site and a real model call are still untried.
- The suite passes (25 tests), and so does the typecheck.

## 2026-11-10: 0.1.0

| Promise | Test |
| --- | --- |
| A new feature gets a draft from its fields; planned gets a Planned page, not generated | `a page for every feature > makes a draft…` |
| A changed feature marks the page out of date; the published words stay; a fresh draft waits | `a page for every feature > marks a page out of date…` |
| A removed feature's page is archived; nobody deletes pages | `a page for every feature > archives…` |
| A failing generator never throws: the page is made, out of date | `a page for every feature > never throws…` |
| Every source synced; removed features archived | `a page for every feature > syncs every source…` |
| The lint finds each kind of internal, and lets allowed names through | `the lint, reviews and publishing > finds internals…` |
| Unreviewed, edited-after-review and internal pages can't publish; only reviewers review | `the lint, reviews and publishing > refuses to publish…` |
| The public sees published, unarchived pages only, without review details | `the lint, reviews and publishing > shows the public…` |
| The check finds missing, out-of-date, unpublished, broken links and lint hits; passes when clean | `the check > reports…` |
| The tree and a page by path | `the check > gives the site a tree…` |

Mutation check: 14 guards, all caught (two after tests were added: a stale review is cleared, and an archived page counts as missing).

## 2026-11-10: after the security read of 8250b10

Ten tests added in `what the security read found`:
- drafts and versions for editors only;
- approval needs `approved: true`, and the plugin stamps the hash;
- many extra pages, and distinct paths for same-titled features;
- out-of-date cleared on approval, so the check goes green;
- archived pages hidden from draft reads;
- section and path linted and reviewed, and internal fields hidden;
- failing and empty sources leave pages alone and are reported;
- a failed draft is retried, and the generator gets only the feature's fields;
- the lint sees through escaping (Markdown, entities, comments, invisible characters, look-alike hyphens);
- rich-text links, the allow-list and members pages.

Mutation check: 17 guards; all caught but two that cover each other. Approval needs both the `approved` tick and a hash the plugin stamps only when it's ticked, so dropping either check alone is caught by the other. That's kept as two layers.
