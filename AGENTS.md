# Repository Guidelines

## Project Structure & Module Organization
Course content lives at the repository root as standalone Markdown lessons (for example `lesson1.md`, `lesson2.md`) and practice assets (`exercise1.md`, `example1.md`). Keep filenames descriptive, lowercase, and numbered sequentially so the syllabus order stays obvious. Store internal automation notes inside `.claude/`; avoid mixing contributor guidance with learner-facing files. When introducing a new module, link to it from the relevant lesson to maintain navigability.

## Build, Test, and Development Commands
The project has no compile step, but please lint before committing:
- `npx markdownlint-cli2 "**/*.md"` — catch heading hierarchy, spacing, and list issues.
- `npx prettier --check "**/*.md"` — confirm consistent wrapping and fenced-code formatting.
Preview lessons locally with the Markdown viewer of your editor or `npx serve .` when you want a quick browser render.

## Coding Style & Naming Conventions
Write in an instructional, second-person voice and keep paragraphs short. Use ATX headings (`#`, `##`) with a single blank line before and after. Favor fenced code blocks with language tags such as ```ts``` or ```html``` so Angular snippets get proper highlighting. TypeScript samples should follow Angular defaults (two-space indentation, camelCase for variables, PascalCase for classes). Call out file paths using inline code, e.g., `src/app/shared/shared.module.ts`.

## Testing Guidelines
Run new or updated code snippets in an Angular CLI sandbox (`npx -p @angular/cli ng new tmp-sandbox`) before publishing to ensure they compile and behave as described. When documenting tests, prefer `ng test` examples and mention expected outcomes. Update exercises with self-check bullet points so readers can validate their work.

## Commit & Pull Request Guidelines
Craft commit messages in the imperative mood with concise context (`add signals lesson outline`). Reference related issues in the body when applicable. Pull requests should include: a summary of the lesson or exercise change, screenshots for rendered Markdown when layout shifts, and a checklist confirming lint commands ran. Request review from another contributor whenever the change affects multiple lesson files.

## Content Review Checklist
Before merging, confirm terminology matches current Angular releases, update any version numbers, and ensure internal links resolve. Verify diagrams or ASCII trees stay under 80 characters in width for readability. Keep learner takeaways actionable and end each lesson with a short recap or next steps section.

Use Angular CLI MCP to fetch all the updated docs you need for Angular