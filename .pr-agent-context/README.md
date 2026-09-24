# XWiki PR-Agent context — provenance

Vendored from https://github.com/xwiki/xwiki-dev-llm (`xwiki/` tree) so that
PR-Agent's text-only skill loader has everything it needs in one place.

- `xwiki/skills/` — 9 of 28 skills, curated for code review (single-shot
  prompt budget ~26k tokens for SKILL.md bodies):
  - `xwiki-review` — the multi-angle review protocol (§3 routing, §5 skeptic
    bar, §6b output-safety constraint). Adapted for single-shot in
    `../XWIKI-REVIEW-CONTEXT.md`.
  - `xwiki-knowledge` — how to read/extend the OKF.
  - `xwiki-test-guidelines` — test-writing rules (pointer file: terminal rules
    are on dev.xwiki.org testing pages it links).
  - `xwiki-javadoc`, `xwiki-translations`, `xwiki-legacy`, `xwiki-xar-pages`,
    `xwiki-build`, `xwiki-fix-sonarqube-issue` — per-angle rule sources.
  - Excluded (non-review tasks, would burn prompt budget): `xwiki-backport`,
    `xwiki-backport-testneeded`, `xwiki-ci-check`, `xwiki-contrib-release-blog-post`,
    `xwiki-convert-tests`, `xwiki-convert-tests-docker`, `xwiki-deploy-extension`,
    `xwiki-doc-convert`, `xwiki-doc-writing`, `xwiki-increase-test-coverage`,
    `xwiki-jira`, `xwiki-openproject`, `xwiki-presentation`,
    `xwiki-release-documentation`, `xwiki-release-test-triage`, `xwiki-rest-api`,
    `xwiki-security-advisory` (note: disclosure-sensitive, keep out of public logs).
- `xwiki/okf/` — full knowledge base (46 files), preserved so `../../okf/`
  references inside skills resolve. PR-Agent only auto-inlines `*.md` inside
  each skill directory, so key OKF files are additionally injected via
  `config.repo_context_files` in `.pr_agent.toml`.
- `xwiki/instructions/xwiki-org.md` — org-wide conventions, injected via
  `config.repo_context_files`.

To refresh: re-copy from a fresh `xwiki-dev-llm` checkout, preserving the
`xwiki/skills` + `xwiki/okf` + `xwiki/instructions` layout.

Caveats (PR-Agent architecture, see `agent_skills.md` upstream):
- All enabled skills are injected into every review up to
  `skills.max_skills_tokens` — there is no progressive disclosure.
- `scripts/` and `assets/` are skipped (no tool-use loop); only `*.md` text
  is inlined. Skills depending on script execution will not work.
- `skills.paths` is host-only and cannot come from `.pr_agent.toml`; it is set
  in `.github/workflows/pr-agent-xwiki.yml` env pointing at
  `$GITHUB_WORKSPACE/.pr-agent-context/xwiki/skills`.
