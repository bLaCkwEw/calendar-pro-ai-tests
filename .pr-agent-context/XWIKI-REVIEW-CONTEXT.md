# XWiki review context for PR-Agent (single-shot adaptation)

PR-Agent runs single-shot model calls: every skill listed in `skills.paths` is
injected into every review prompt (no progressive disclosure), and there is no
tool-use loop. This file distills how to apply the vendored XWiki skills in
that setting. The full rules live in `xwiki/skills/` and `xwiki/okf/` next to
this file — cite the terminal source, not this distillation.

## What is vendored here

- `xwiki/skills/`: 9 review-relevant skills (see README.md for the list).
- `xwiki/okf/`: the declarative knowledge base (conventions, architecture,
  testing, sonarqube rule correctness, servers, processes). Map: `xwiki/okf/index.md`.
- `xwiki/instructions/xwiki-org.md`: org-wide conventions injected in dev sessions.

Skill paths use `../../okf/` relative to the skill directory, which resolves
inside this layout (e.g. `xwiki/skills/xwiki-review/` → `xwiki/okf/`).

## How to review XWiki code in one pass

1. **Be XWiki-aware, not generic.** Every finding must trace to a rule this
   project actually holds — an `xwiki/okf/` file, a vendored skill, or the
   repo's `CLAUDE.md`/`AGENTS.md`. Never flag generic best-practice folklore.
2. **Cite the terminal source.** Some skills are pointers (e.g.
   `xwiki-test-guidelines` points at dev.xwiki.org testing pages). If the words
   are not in the cited file, follow one hop (the linked page, the `verify:`
   recipe) and cite that page. Quote any clause that limits the rule's scope
   (file types, excluded modules, "unless"). A truncated quote is a false positive.
3. **Route by changed files** (from `xwiki-review` §3):
   - any `.java` → architecture (`component-system.md`), backward compatibility
     (`backward-compatibility.md` + `versioning.md`), conventions (`code-style.md`,
     `code-comments.md`, `logging.md`), performance (`performance.md`), security
     mechanisms (`security.md`).
   - tests touched or missing → `okf/testing/strategy.md` + `xwiki-test-guidelines`.
   - user-facing strings, `.properties`, `.vm`, `.xml` sheets → `xwiki-translations`.
   - new/changed public API → `xwiki-javadoc` + `okf/conventions/documentation.md`.
   - XAR pages (`src/main/resources/` `*.xml`) → `xwiki-xar-pages`.
   - deprecated API touched → `xwiki-legacy` (it belongs in `-legacy`, not main).
   - Sonar-shaped nit → check `xwiki/okf/sonarqube/index.md` first, then only the
     one family file for that rule; many mechanical fixes are wrong in XWiki.
4. **Hard XWiki rules to check first:**
   - Lines ≤ 120 chars; LGPL headers present; `jakarta.*` not `javax.*` in new code.
   - Components via `@Component`/`@Inject`/`@Role`, not context-passing.
   - `@since` / `@Deprecated(since=…)` = next dev version `<X.Y.0>RC1` read from
     the root `pom.xml`, never from memory or a stale `CLAUDE.md`.
   - Public/protected signature changes need backward-compat handling (default
     methods, `@Unstable` lifecycle); Revapi covers main artifacts, not `-legacy`.
   - Stream user-sized data, never buffer (`byte[]`/`String`/`ByteArrayOutputStream`).
   - Tests: no stdout/stderr, page-object boundary (no `getDriver()` in test class),
     reuse existing `*IT` fixtures instead of rebuilding, lightest base
     (`@ComponentTest` over `@OldcoreTest`), JUnit5/Mockito for new tests.
   - Translations: externalized, escaped (`$escapetool.xml` etc.), word-order safe.
   - Javadoc on new public API explains the contract, not the signature.
   - Commit subject = `KEY-12345: <JIRA title verbatim>` (`XWIKI`/`XCOMMONS`/
     `XRENDERING`/contrib key); `[Misc]` only for trivial changes. Tracker is
     jira.xwiki.org, NOT GitHub Issues.
5. **Finding bar:** report nothing rather than padding. Zero findings is a good
   outcome. Never report what Checkstyle/Revapi/compiler already catch,
   pre-existing issues on untouched lines, or taste with no rule behind it.
   Be direct, no praise section, no PR summary, no emojis in findings.

## Output safety (hard constraint, from `xwiki-review` §6b)

GitHub Actions logs on a public repo are public. There is no safe place in this
pipeline to say something is a vulnerability — not in the comment, not in the
reasoning, not in the logs.

- **Write:** "line 42 writes `$doc.title` without `$escapetool.xml`";
  "this state-changing endpoint lacks the CSRF form token";
  "this query concatenates a parameter instead of binding it".
- **Never write:** vulnerability, exploit, attack, attacker, XSS, CSRF-as-a-flaw,
  injection, traversal, escalation, "unsafe", "an attacker could", "this allows",
  CVE, CVSS, or severity justified by impact. No `security` label or framing.
- Check mechanisms as a linter would (escaper present? check present?
  untrusted value treated as untrusted?) — do not assess exploitability.
- This covers conventions only; it does not replace a human security review
  and must never be described as one.
