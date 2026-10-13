# CLAUDE.md

github-mate is a desktop app that supports GitHub-centred development. It is at the specification stage; there is no application code yet.

## Specification

- **Every file Claude reads is written in English** (the human, 2026-10-10): this file, `docs/SPEC.md`, and any later design or process document. The conversation with the human and the app's UI text are in Japanese.
- The specification is `docs/SPEC.md`. Update it whenever something is decided in conversation. Keep what the human stated, what Claude assumed, and what is still open in separate sections.
- **The specification describes the finished product.** Do not mix development steps into it, and do not ask the human what goes into "the first version" (the human, 2026-10-10: 「仕様を考えるのと開発ステップを考えるのをごっちゃにしている」).
- **Development steps are Claude's to cut**, by what is easy to develop and easy to verify. The human has no preference about which step delivers what, and will start using the app only after the final version is built.

## Screen design

- **Screen layout and appearance are discussed with the human through HTML mocks** (the human, 2026-10-10). Do not try to settle them with text descriptions or ASCII sketches.
- Mocks live in `mock/` as self-contained HTML files (no external resources) that open directly in a browser.
- When a screen question comes up, build or update a mock, show it, and write what the human decides into `docs/SPEC.md`.
