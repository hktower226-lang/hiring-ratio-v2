# Working on this repository

- This is a static mobile dashboard. Serve the repository root over HTTP; no build or dependency installation is needed.
- `data/sheet1.json` is the only source of application data. Do not embed a second data copy in `index.html`. The page fetches JSON on load, manual refresh, tab return, and every minute while visible.
- Preserve source totals and notes. Affiliation sums may differ from equipment totals; do not silently rebalance them.
- Rental company grouping uses `rental`, not `contractor`. Company labels use the exact, unrounded Han-no ratio: at least 40% is 우호업체, at most 30% is 비협조업체, and the interval between has no label.
- Validate relevant interactions, calculations, and small mobile widths after UI changes. See `data/README.md` for the data schema and `DEPLOY.md` for hosting.

## Git in this cloud environment

The platform Git proxy works when the full HTTPS repository URL is passed explicitly. Remote aliases such as `origin` can fail with a username/authentication prompt even when access is available. Use the existing platform authentication; no token needs to be copied into files or URLs.

```sh
git ls-remote https://github.com/hktower226-lang/hiring-ratio-v2.git HEAD refs/heads/main
git fetch https://github.com/hktower226-lang/hiring-ratio-v2.git main
# After checking the remote state, when an upload is authorized:
git push https://github.com/hktower226-lang/hiring-ratio-v2.git HEAD:refs/heads/main
```

Use the existing checkout. Do not create worktrees unless the user requests one. Preserve concurrent remote changes and avoid force-pushing. GitHub API access is separate from Git access; a blocked API request does not establish a Git authentication failure.
