# TopTea POS Code Audit Report

## Audit Scope

**Scanned Directories:**
- `store_html/html/pos/api/` (and subdirectories)
- `store_html/pos_backend/helpers/`
- `store_html/html/pos/assets/js/`
- `store_html/html/pos/pos/` (Duplicate directory)
- `store_html/pos_backend/core/`

**Documents Reviewed:**
- `docs/pos_known_issues_and_debts.md`: Identified `pos_invoice_payments` ghost table and `calculate_eod_totals` ghost function.
- `docs/总对齐文档（POS 视角版）_v2.txt`: Confirmed V2 pass redemption logic.
- `docs/L4_2B_POS_代码引用审计_阶段报告_20251121.txt`: Confirmed schema usage.

## Overlap Overview

**Typical Overlap Patterns Identified:**

1.  **Recursive Directory Duplication (The "Ghost" Directory):**
    -   `store_html/html/pos/pos/` is a complete recursive copy of `store_html/html/pos/`.
    -   It contains full copies of API, assets, and logic.
    -   Example: `store_html/html/pos/pos/api/pos_api_gateway.php` (Group: DUPLICATE_DIR_001).

2.  **Migration Script Versioning:**
    -   `migrate_pass_redemptions.php` (v1) vs `migrate_pass_redemptions_v2.php` (v2).
    -   V2 is the corrected version handling both batch and detail table schemas.
    -   Example: `store_html/html/pos/api/migrate_pass_redemptions_v2.php` (Group: MIGRATION_001).

3.  **Deprecated Logic within Active Files:**
    -   `calculate_eod_totals` in `pos_helper.php` is explicitly deprecated but the file itself is main line.

## Key Business Logic Analysis

### 1. Pass Purchase (Sale)
-   **Main Line (WIRED_MAIN):** `pos_registry_member_pass.php` -> `handle_pass_purchase`.
-   **Helpers:** `pos_pass_helper.php` (`create_pass_records`), `pos_repo_ext_pass.php` (`get_pass_plan_by_sku`, `allocate_vr_invoice_number`).
-   **Ghost/Old:** None found in logic files, but duplicates exist in `html/pos/pos/`.

### 2. Pass Redemption
-   **Main Line (WIRED_MAIN):** `pos_registry_ext_pass.php` -> `handle_pass_redeem`.
-   **Helpers:** `pos_pass_helper.php` (`create_redeem_records`, `validate_redeem_limits`, `calculate_redeem_allocation`).
-   **Ghost/Old:** `migrate_pass_redemptions.php` (Old migration tool).

### 3. EOD (End of Day)
-   **Main Line (WIRED_MAIN):** `pos_registry_ops_eod.php` -> `handle_eod_submit_report`.
-   **Helpers:** `pos_repo.php` -> `getInvoiceSummaryForPeriod`.
-   **Ghost/Old:** `pos_helper.php` -> `calculate_eod_totals` (Function is DEAD/DEPRECATED).

## Risks and Suggestions

1.  **High Risk: Recursive Duplicate Directory `html/pos/pos/`**
    -   **Risk:** Confusion for developers. If someone edits a file in `pos/pos/` thinking it's live, changes will be lost.
    -   **Suggestion:** Delete `store_html/html/pos/pos/` entirely.

2.  **Ghost Function `calculate_eod_totals`**
    -   **Risk:** It references a non-existent table `pos_invoice_payments`. Any accidental call will crash the system (500 Error).
    -   **Suggestion:** Remove the function body or the function entirely (it currently throws an exception, which is safe but messy).

3.  **Artifact Comment in `member.js`**
    -   `store_html/html/pos/assets/js/modules/member.js` has a comment `// store_html/html/pos/pos/assets/js/modules/member.js` at the top.
    -   **Risk:** Low, but indicates a careless copy-paste from the duplicate directory.
    -   **Suggestion:** Clean up the header comment.

## Ghost New Line Files (New but Unwired)

*None identified as "Business Logic".*
The only "New but Unwired" files are migration tools which are expected to be manually triggered:
-   `store_html/html/pos/api/migrate_pass_redemptions_v2.php` (Group: MIGRATION_001)

## Machine-Readable Overlap List

```csv
file_path | group_id | role | freshness | reason
store_html/html/pos/api/pos_api_gateway.php | GATEWAY_001 | WIRED_MAIN | LIKELY_NEW | Main entry point used by frontend.
store_html/html/pos/api/registries/pos_registry.php | REGISTRY_001 | WIRED_MAIN | LIKELY_NEW | Main registry included by gateway.
store_html/html/pos/api/registries/pos_registry_ext_pass.php | REGISTRY_PASS_001 | WIRED_MAIN | LIKELY_NEW | Extension registry for pass redemption.
store_html/html/pos/api/registries/pos_registry_member_pass.php | REGISTRY_MEMBER_001 | WIRED_MAIN | LIKELY_NEW | Registry for member and pass purchase.
store_html/html/pos/api/registries/pos_registry_ops.php | REGISTRY_OPS_001 | WIRED_MAIN | LIKELY_NEW | Registry for operations.
store_html/html/pos/api/registries/pos_registry_ops_eod.php | REGISTRY_EOD_001 | WIRED_MAIN | LIKELY_NEW | Registry for EOD logic.
store_html/html/pos/api/registries/pos_registry_ops_shift.php | REGISTRY_SHIFT_001 | WIRED_MAIN | LIKELY_NEW | Registry for Shift logic.
store_html/html/pos/api/registries/pos_registry_sales.php | REGISTRY_SALES_001 | WIRED_MAIN | LIKELY_NEW | Registry for Order/Sales logic.
store_html/html/pos/api/registries/pos_registry_diag.php | REGISTRY_DIAG_001 | WIRED_MAIN | LIKELY_NEW | Diagnostic registry included by gateway.
store_html/pos_backend/helpers/pos_helper.php | HELPER_CORE_001 | WIRED_MAIN | LIKELY_NEW | Core helper used by registries (contains ghost function calculate_eod_totals).
store_html/pos_backend/helpers/pos_repo.php | HELPER_REPO_001 | WIRED_MAIN | LIKELY_NEW | Core repository used by registries.
store_html/pos_backend/helpers/pos_pass_helper.php | HELPER_PASS_001 | WIRED_MAIN | LIKELY_NEW | Pass logic helper used by registries.
store_html/pos_backend/helpers/pos_repo_ext_pass.php | HELPER_PASS_REPO_001 | WIRED_MAIN | LIKELY_NEW | Pass DB helper used by registries.
store_html/html/pos/api/pos_login_handler.php | LOGIN_001 | WIRED_MAIN | LIKELY_NEW | Used by login.php form post.
store_html/html/pos/api/migrate_pass_redemptions.php | MIGRATION_001 | UNUSED | LIKELY_OLD | V1 migration script.
store_html/html/pos/api/migrate_pass_redemptions_v2.php | MIGRATION_001 | UNUSED | LIKELY_NEW | V2 migration script (Newer logic).
store_html/html/pos/api/diagnose_500.php | TOOL_DIAG_001 | UNUSED | LIKELY_NEW | Standalone diagnostic tool.
store_html/html/pos/api/simulate_member_search.php | TOOL_TEST_001 | UNUSED | UNCLEAR | Standalone test script.
store_html/html/pos/pos/api/pos_api_gateway.php | DUPLICATE_DIR_001 | UNUSED | LIKELY_OLD | Duplicate file in pos/pos/ directory.
store_html/html/pos/pos/api/registries/pos_registry.php | DUPLICATE_DIR_001 | UNUSED | LIKELY_OLD | Duplicate file in pos/pos/ directory.
store_html/html/pos/pos/api/registries/pos_registry_ext_pass.php | DUPLICATE_DIR_001 | UNUSED | LIKELY_OLD | Duplicate file in pos/pos/ directory.
store_html/html/pos/pos/login.php | DUPLICATE_DIR_001 | UNUSED | LIKELY_OLD | Duplicate file in pos/pos/ directory.
```
