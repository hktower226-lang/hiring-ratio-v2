# Working on this repository

This is a static mobile dashboard. No build or dependency installation is needed. Read `README.md`, `data/README.md`, and `DEPLOY.md` when resuming in a new conversation. Use the existing checkout; do not create a worktree unless the user requests one.

## Current source layout and editing workflow

`index.html` fetches these three independent UTF-8 JSON sources. It must not contain another embedded copy of their records.

| Request/menu | Source file |
| --- | --- |
| 1번 JSON, 전체, 월별, 지부별, 임대사별 | `data/01_전체_월별_지부별_임대사별.json` |
| 2번 JSON, 반도체 | `data/02_반도체.json` |
| 3번 JSON, 전체비율 | `data/03_전체비율.json` |

Resolve a data-edit request to the corresponding file. Change only the requested records and preserve unrelated fields. Ratios, sums, company classifications, and graphs are derived at runtime, so ordinary data updates do not require an HTML edit. Keep the three datasets separate, and never edit the uploaded source workbook. Compute totals from current records instead of hardcoding historical totals.

The page uses `fetch` with `cache: no-store` on initial load, manual 데이터 새로고침, return to the page, and every 60 seconds while visible. All three responses are validated before replacing the current dataset together. A failed refresh keeps the last valid screen/data and reports the failure. Keep this behavior when changing data loading. Serve the repository root over HTTP for testing; opening the HTML as a local file cannot fetch the JSON.

The current user request explicitly authorizes separation, understandable filenames, and uploading the completed changes to GitHub. It supersedes the earlier temporary instruction to keep combined previews local. Do not reimpose that obsolete restriction. For subsequent work, follow the user's requested delivery and existing session authorization; a new request to change visibility or other repository administration is a separate action.

## Data and display rules

- The all-menu title is `26년 신규채용비율`; its bottom navigation label remains `전체`. Month/branch/rental menus use only file 1. Rental grouping uses `rental`, not `contractor`.
- The all view shows file 1 alone and file 1 plus file 2 `records`; file 2 `assignedRecords` never enter either all-view summary. Its detail/search list is file 1 only and is labeled accordingly.
- The semiconductor view defaults to one `records` summary titled `반도체 현장(현재배정진행중)`. Its dedicated rental search filters rental names and shows two company summaries when matched: `records + assignedRecords`, then `records`. It also filters the detail list and section counts. The general search and section buttons affect the detail list, not these statistics. A company with only assigned records has a zero active summary; an unmatched company shows an empty result.
- Semiconductor corrections: P5 정수장 3기 is in `assignedRecords`. P4 복합동 and P4 대기방지 were removed only from `records`; their existing assigned records remain. Do not infer missing month, branch, or contractor values.
- File 3 is an independent monthly snapshot. Source totals already exclude undecided personnel; do not subtract unknown counts again or infer a missing unknown count. Do not sum monthly snapshots into one current total or merge them into files 1/2.
- File 3 shows every month and affiliation at once, with no month/series selection controls. Before October, fiber/union values and reported percentages are `null`, not zero. The original 기타노조 미분리 line runs through September and connects to October fiber and union points. Those branches indicate classification change, not invented pre-October individual figures. Show all months in the compact table, splitting October's 기타노조 cell into fiber/union details.
- Preserve October's total 568 versus affiliation sum 569 and the user's `note`: `한 장비에 2명이 탑승하여 총대수와 소속 합계에 1대 차이가 있습니다.` Show it under 비고. Preserve the first dataset's equipment/affiliation differences and notes too; never silently rebalance sums.
- Company labels use the exact unrounded Han-no ratio: at least 40% is 우호업체, at most 30% is 비협조업체, and the interval between has no label.
- Keep the exact current-ratio explanation requested by the user: bold `미정 인원을 제외한 현재 기준입니다`, followed by `미정 인원을 제외한 총수를 기준으로 소속별 비율을 계산합니다. 미정을 포함한 전체대수 기준보다 각 소속비율이 조금더 높게 보입니다.`

## Validation and delivery

Run the relevant browser checks after changes: actual JSON loading and refresh, calculations and affected views, searches, company boundaries if affected, empty/error cases, and mobile widths including 320/390 pixels. Do not claim that GitHub Pages has deployed a commit until checked. For future data edits, explain which numbered file changed, what aggregates changed, and whether the changes were uploaded/deployed. Screenshots are static appearances, not interactive web previews.

## Git in this cloud environment

The platform Git proxy works with the full explicit HTTPS repository URL. `origin` aliases can fail with a username/authentication prompt despite working access. Reuse platform authentication; never copy credential values into files or URLs.

```sh
git ls-remote https://github.com/hktower226-lang/hiring-ratio-v2.git HEAD refs/heads/main
git fetch https://github.com/hktower226-lang/hiring-ratio-v2.git main
# After checking remote state, when an upload is authorized:
git push https://github.com/hktower226-lang/hiring-ratio-v2.git HEAD:refs/heads/main
```

Preserve concurrent remote changes and avoid force-pushing. GitHub API access is separate from Git access; a blocked API request does not establish a Git authentication failure.
