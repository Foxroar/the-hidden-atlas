# Development workflow

- Treat main as production. Create a separate feature branch for each task and open a pull request. Never push directly to main or merge without the owner's explicit approval.
- Preserve existing functionality and source files. Keep changes focused and reversible.
- This project currently uses a static index.html with Leaflet; do not assume npm or a framework is required. Serve over HTTP and check browser console, map loading and fallback, search, filters, layer controls, notebook persistence and responsive layouts at phone and tablet widths. Report checks actually run and any limitations.
- Review Vercel preview deployments before merging. Report the preview URL if available, changed files, test results and known issues. Verify production after an approved merge.
- Never commit secrets, tokens or .env files. Use environment-scoped configuration and avoid production database writes during testing.
- Do not change deployment access, billing, domains or database schema without explicit approval.
