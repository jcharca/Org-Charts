# Context Handoff — Regional Org Charts (MPG)

## What exists
An interactive org-chart tool (single self-contained `index.html`, data inlined as JSON) in the GitHub repo **jcharca/Org-Charts**, branch `claude/regional-org-charts-filter-nxpzdy` (also mirrored to `main`). Deployed via Cloudflare Pages at **org-charts-6nk.pages.dev** (production branch = the claude/... branch; Git connection to Cloudflare was flaky — if deploys stall, check the "project disconnected from Git" banner, or drag-and-drop upload `index.html` as a manual deployment).

## Source data
- File: `RVP_Org_Chart.xlsx` (Sheet1, 5,010 employee rows, 28 cols).
- Key columns: `RVP Flag` (region tag, only set on the 7 RVP rows), `Employee Name`, `Business Title`, `Manager` (name — this is the edge used to build the tree), `Level 1..8` (mgmt chain from Doug Wenners down), `Work City/State`, `FTE %`, `Active Status`.
- Tool filters to `Active Status = Yes` only.
- A processed `org_data.json` in the repo holds the full recursive tree per region with computed `team` = total downstream headcount.

## The 8 regions (top person, total downline)
| Region | Leader | Title | Team |
|---|---|---|---|
| Lee | Jessie Dye | VP, Regional | 470 |
| Charlotte | April Sowell | VP, Regional | 418 |
| NE Florida | Casey Jabot | VP, Regional | 387 |
| Collier | Leslie Gietano | VP, Regional | 259 |
| Tampa / Central FL | Nicole Mattia-LaRochelle | VP, Regional | 213 |
| Non-FL | Rebecca King | VP, Regional | 187 |
| IMA | Tim Pellandini | VP, Regional-IMA | 184 |
| Sarasota (Temp) | Todd Hightower | VP of Value Based Care | 515 |

- All RVPs report to Tesha Simpson (EVP/CEO — MPG); above her, Doug Wenners. Most FL RVPs route through DeLyle Manwaring (SVP, Market President).
- **Sarasota caveat**: Sarasota staff (~45 people, led by Sr. Practice Administrator Angela Halloran under Director of Ops Karrie Falcone) do NOT roll to any RVP. They roll: Karrie Falcone → **Todd Hightower (VP, Value Based Care)** → DeLyle Manwaring. As a temporary decision, the tool shows Todd's ENTIRE 515-person VBC org as region "Sarasota (Temp)". Open org-design question: keep as own region, rename to "Value Based Care", trim to Karrie's group, or reassign Sarasota under an existing RVP.
- Depth below region leader: 3 levels for most regions; 4 for Charlotte and IMA.

## Typical region shape (relevant to org design)
- RVP → mix of: Directors of Operations, Directors of Practice Administration, direct-reporting physicians/APRNs (e.g., Lee has 25 directs of which ~19 are providers), occasional Regional Coordinator / Exec Assistant.
- Directors → Practice Administrators (Senior/Associate tiers exist) → clinic staff (Medical Assistants, Patient Care Specialists, LPNs).
- Example L1 spans: Charlotte is clean (4 directs: 2 Dir Ops, 1 Dir Practice Admin, 1 coordinator); Lee is noisy (25 directs incl. many providers); Non-FL is flat (13 directs, mostly Practice Administrators reporting straight to the RVP — no director layer).
- Role taxonomy used by the tool (derived from Business Title, regex in the data-gen script): RVP / Director of Operations / Director, Practice Administration / Director (Other) / Practice Administrator / Manager / Nursing Manager / Supervisor / Physician / Advanced Provider (APRN/NP/PA) / Clinical-Medical Assistant / Administrative-Support / Other.

## Tool features (as built)
- One region shown at a time; role-type checkbox filters PER LEVEL (L1/L2/L3/L4 sections auto-generated, options sorted by count desc); deeper levels render nested under their manager, not flattened.
- Cards sorted by team size (downstream headcount) desc; "N in team" badge on each card.
- Right panel: chain-of-command list grouped by L1 role, with breadcrumb paths; "Copy roster to clipboard" (indented text).
- Header buttons hide/show the left (Filters) and right (Chain of Command) panels.
- Defaults: L1 shows leadership roles, L2 shows Practice Admin + leadership, L3+ unchecked.
- Non-RVP region roots display "Lead" instead of "RVP" (drives off group === 'RVP / Regional VP').

## Data regen pipeline
`index.html` embeds the JSON in `<script id="data" type="application/json">`. To refresh: run the Python script pattern (openpyxl over the xlsx → build tree from `Manager` edges, active only, compute team, role_group regex) → write `org_data.json` → regex-replace the script block contents in `index.html`.

## Unresolved / watch-outs
1. A second file `20260604_Roster_Sensitive_...xlsx` was uploaded but is **password-encrypted** — never opened. It may contain sensitive fields (comp?). NOTE: repo + site are PUBLIC; current tool only exposes name/title/location/manager. Decide carefully before adding anything from the sensitive roster.
2. Cloudflare Git connection has disconnected before; verify deploys actually pick up new commits.
3. Duplicate-name guard exists in tree build (cycle protection); names are the join key (no manager-ID), so name collisions are a theoretical risk.
4. FTE % column exists per-person (100/50/15 for PRN) but "team" counts are headcount, not FTE-weighted — an org-design thread may want FTE-weighted rollups.
