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

The page uses `fetch` with `cache: no-store` only on initial load, manual 데이터 새로고침, and an explicit retry after a load failure. Do not poll on an interval or refetch on focus/visibility changes: the user requested less mobile data usage. After publishing JSON edits, users open/reload the page or press 데이터 새로고침 to see the new data. All three responses are validated before replacing the current dataset together. A failed refresh keeps the last valid screen/data and reports the failure. Keep this behavior when changing data loading. Serve the repository root over HTTP for testing; opening the HTML as a local file cannot fetch the JSON.

The current user request explicitly authorizes separation, understandable filenames, and uploading the completed changes to GitHub. It supersedes the earlier temporary instruction to keep combined previews local. Do not reimpose that obsolete restriction. For subsequent work, follow the user's requested delivery and existing session authorization; a new request to change visibility or other repository administration is a separate action.

## 채용비율 데이터 수정 지침

### 1. 기본 파일과 작업 순서

- 실제 데이터 수정 전에는 GitHub `main`의 최신 JSON부터 확인한다. 전체 HTTPS URL로 fetch하고 현재 로컬 변경을 보존하여, 이전 파일로 최신 내용을 덮어쓰지 않는다.
- 사용자가 **반도체 현장/2번 JSON** 또는 **전체비율/3번 JSON**을 명시하지 않으면 기본 수정 대상은 **1번 `data/01_전체_월별_지부별_임대사별.json`**이다. 현장 추가·수정·대수 변경을 이 파일에서 처리한다.
- “전체 수치도 변경해줘”는 해당 현장 변경에 따른 화면의 합계·비율 재계산을 뜻한다. 이 표현만으로 독립된 3번 전체비율 JSON을 수정하지 않는다.
- 요청한 현장만 추가·수정·이동하고 관련 없는 현장과 다른 JSON은 보존한다. 합계·비율·그래프는 JSON으로부터 자동 계산하므로 일반적인 현장 변경 때문에 HTML에 수치를 직접 적지 않는다.

### 2. 일반 현장 입력 내용과 등록 월 — 1번 JSON

- 입력의 지부 → `branch`, 타워임대사/임대사 → `rental`, 원청사 → `contractor`, 현장명 → `site`, 총대수 → `total`, 소속별 대수 → `counts`로 기록한다.
- 입사일자가 있으면 해당 날짜의 월을 `month`로 사용한다. 정확한 날짜는 `hireDate`에 `YYYY-MM-DD` 형식으로 보관한다. 예를 들어 2026년 09월 29일은 9월 현장이다.
- 1번 JSON에 새로운 월을 추가하면 최상위 `months`에도 해당 월을 중복 없이 넣고 월 순서를 유지한다.
- 날짜가 없는 기존 현장의 부분 수정은 기존 월을 유지한다. 신규 현장의 월을 알 수 없으면 임의로 정하지 말고 필요한 월만 확인한다.
- 신규 현장은 `한노`, `민노`, `섬유`, `연합`, `직원` 중 입력에 없는 소속을 0으로 기록한다. 기존 현장의 부분 수정에서는 언급하지 않은 소속·비고·기타 필드를 보존한다. 새 현장의 `id`는 기존 값과 겹치지 않게 부여한다.
- 완성된 현장 정보를 다시 주면 대수는 **최종값으로 교체**한다. 기존 대수에 다시 더하지 않는다. “한노 1대 추가/감소”처럼 증감을 명시했을 때만 그 증감으로 처리한다. 총대수와 소속 합계가 다르다고 소속 항목을 만들어 차이를 채우거나 수치를 보정하지 않는다.

### 3. 일반 현장 중복과 월 이동 — 1번 JSON

- 동일 현장의 판별 기준은 **임대사 + 원청사 + 현장명** 세 항목이다. 월과 지부는 중복 판별 키에 넣지 않는다. 공백 등 단순 표기 차이를 확인하되 비슷한 현장명만으로 서로 다른 현장을 합치지 않는다.
- 수정 대상 파일의 모든 월에서 이 세 항목을 먼저 검색한다. 같은 현장이 같은 달에 이미 있으면 기존 행을 갱신하고 **한 건만 유지**한다. 내용까지 같으면 추가·가산하지 않는다. 같은 달의 재요청에 실제 대수 변경이 있으면 그 변경은 반영한다.
- 같은 현장이 다른 달에 있고 새 입사일자/등록 월을 주면 **이전 월 기록을 없애고 새 월로 이동**한다. 가능하면 기존 `id`와 요청하지 않은 필드를 유지하고, 새 요청의 지부·날짜·대수 등 명시한 값으로 갱신한다. 9월 현장을 10월로 다시 주면 최종적으로 10월에만 한 건이 남아야 한다.
- 해당 중복 현장이 여러 행으로 들어가 있다면 요청한 현장에 한해서 한 건으로 정리한다. 관련 없는 기존 현장을 일괄 삭제하거나 합치지 않는다.

### 4. 반도체 현장과 전체비율

- **사용자가 반도체 현장 수정을 명시한 경우에만 2번 JSON을 갱신한다.** 일반 현장 요청으로 반도체 데이터를 자동 수정하지 않는다. 전체비율/3번 JSON 역시 해당 메뉴의 수정을 명시한 경우에만 갱신한다.
- 반도체 현장을 1번 JSON에 복사하거나 일반 현장 통계에 중복 집계하지 않는다. 같은 현장이 두 파일에 걸쳐 발견되면 자동으로 두 파일을 함께 수정·합산하지 말고 분류와 중복 집계 여부를 확인한다.
- 1번과 2번은 `categories`와 모든 `counts`에 **`한노`, `민노`, `섬유`, `연합`, `직원` 다섯 소속만** 같은 순서로 보관한다. 사용자 요청으로 미정 항목을 JSON과 화면에서 완전히 삭제했다. `records`뿐 아니라 반도체 `assignedRecords`에도 미정 키를 다시 만들지 않는다.
- 2026-10-25 기존 미정 대수는 각 현장 `total`에서 이미 한 번 차감했다. 현재 총대수는 1번 218대, 반도체 진행중 96대, 배정 완료 28대이며 미정 키 삭제로 다시 차감하지 않는다. 원본 엑셀이나 과거 입력 예시의 미정을 복구하지 않는다. 3번은 이미 미정 제외 기준이므로 이 작업으로 변경하지 않는다.
- 이후 1번·2번 입력에 미정이 있더라도 저장·표시 항목에 추가하지 않는다. 총대수가 미정을 포함함이 명확하고 미정 대수도 함께 제공되면 `total`에는 미정 대수를 한 번 뺀 값을 기록한다. 이미 미정 제외 총대수라고 명시한 값은 그대로 사용한다. 총대수의 포함·제외 기준이 불명확하면 그 기준만 확인한 뒤 처리한다. 미정 대수가 없으면 총대수와 소속 합계의 차이로 미정을 추정하거나 차감하지 않는다.
- 기존 전체 화면의 **‘반도체 현장포함’ 비교 통계**는 유지한다. 이 통계에서만 1번 현장과 2번의 현재 배정 진행중 `records`를 의도적으로 합산하고, `assignedRecords`는 제외한다. 월별·지부별·임대사별 일반 현장 통계에는 반도체를 넣지 않는다.

### 5. 검증과 처리 결과

- 수정 후 JSON 형식, 대상 현장의 중복 여부·등록 월·대수, 영향받은 전체·월별·지부별·임대사별 합계와 비율을 확인한다. 월 이동은 이전 달에서 빠지고 새 달에 한 번만 들어갔는지도 확인한다.
- 사용자가 요청한 전달 방식과 기존 업로드 승인을 따른다. GitHub에 반영할 때는 원격 변경을 보존하고, 반영 여부를 확인한 뒤 보고한다.
- 결과에는 수정한 JSON 번호, 추가/수정/이동 내용, 변경된 합계, 업로드/배포 여부를 알기 쉽게 정리한다.

### 입력 형식 예시 — 반복 등록하지 않기

아래는 사용자가 입력 형식을 설명한 뒤 별도로 실제 등록을 요청한 사례이다. **이 예시는 자동 실행할 등록 요청이 아니다.** 실제 등록 여부·현재 월·대수는 항상 GitHub의 최신 1번 JSON에서 확인하며, 지침을 읽었다는 이유로 다시 추가하지 않는다.

> 남부지부 / 입사일자 2026년 09월 29일 / 임대사 (주)국영타워랜드 / 원청사 지에스건설(주) / 현장명 북오산자이리버블시티 1구역 / 총 6대, 한노 1대, 민노 2대, 미정 3대

이 과거 입력 형식은 1번 JSON의 남부지부·9월 기록으로 해석한다. 현재 기준으로 위 입력을 새로 처리하면 미정 포함 6대에서 명시된 미정 3대를 한 번 제외한 `total: 3`, 한노 1·민노 2·섬유 0·연합 0·직원 0으로 저장하며 미정 키는 만들지 않는다. 이미 등록된 현재 기록에서 다시 3대를 빼지 않는다. 동일한 세 항목의 현장이 있는지 먼저 확인하고 추가 또는 수정·이동한다.

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
