---
name: Code Whisperer
description: Intelligently analyzes code quality, identifies patterns, and provides gentle guidance for improvements
version: 1.2.0
author: OpenClaw Team
tags: [code, quality, analysis, patterns, refactoring, suggestions]
maintainer: kode-team@openclaw.io
homepage: https://docs.openclaw.io/skills/code-whisperer
dependencies:
  - clang-tidy >= 15.0
  - black >= 23.0 (Python)
  - prettier >= 3.0 (JS/TS/JSON)
  - shellcheck >= 0.9.0 (Bash)
  - yamllint >= 1.30 (YAML)
  - ruby -c (Ruby syntax)
  - golangci-lint >= 1.50 (Go)
  - hadolint >= 3.0 (Docker)
  - markdownlint >= 0.28 (Markdown)
required_env:
  - OPENCLAW_WORKSPACE (path)
  - OPENCLAW_SKILLS_DIR (path)
optional_env:
  - CODE_WHISPERER_STRICT_MODE=1 (enable strict checks)
  - CODE_WHISPERER_AUTO_FIX=1 (auto-apply safe fixes)
  - CODE_WHISPERER_VERBOSE=1 (detailed output)
capabilities:
  - analyze
  - suggest
  - refactor-patterns
  - detect-anti-patterns
  - complexity-calc
  - security-scan
  - style-enforce
  - dead-code
---

# Code Whisperer

Intelligent code analysis and quality improvement engine that learns from your codebase and provides contextual, non-intrusive suggestions for cleaner, more maintainable code.

## Purpose

Real use cases:
- **Detect God Objects**: Find classes/modules with >500 lines and >15 methods, suggest splitting by responsibility
- **Spot Feature Envy**: Identify methods accessing another object's data >3x more than its own, recommend move method refactoring
- **Uncover Duplication**: Detect copy-paste code blocks with >70% similarity across files, suggest extraction
- **Flag Complex Conditionals**: Find nested if/else depth >3, propose guard clauses or strategy pattern
- **Highlight Incidental Complexity**: Locate comment-heavy code (comments:code ratio >0.5), indicate need for self-documenting code
- **Find Data Clumps**: Track parameter lists >4 items passed together, suggest parameter object
- **Expose Shotgun Surgery**: Trace single change affecting >5 files, recommend facade or observer
- **Reveal Speculative Generality**: Identify unused abstract classes/interfaces, recommend removal
- **Detect Primitive Obsession**: Find string/number fields that should be value objects
- **Catch Contrived Hierarchy**: Spot inheritance used where composition would be simpler

## Scope

Exact commands:

### `code-whisperer analyze <path> [--file=<ext>] [--pattern=<type>] [--fix]`

Analyzes files/directories for code smells.  
**Real patterns**: `god-object`, `feature-envy`, `duplication`, `complex-conditional`, `data-clump`, `shotgun-surgery`, `speculative-generality`, `primitive-obsession`, `contrived-hierarchy`, `middle-man`, `lazy-class`, `comment-code`

**Example**:  
`code-whisperer analyze src/ --file=.py --pattern=duplication --fix`  
`code-whisperer analyze app/controllers --file=.js --pattern=feature-envy`

### `code-whisperer complexity <file> --metric=cyclomatic`

Calculates complexity metrics with thresholds.  
**Metrics**: `cyclomatic` (>10 warnings), `cognitive` (>15), `npath` (>200), `maintainability-index` (<20)

**Example**:  
`code-whisperer complexity src/utils/parser.py --metric=cyclomatic`

### `code-whisperer dead-code <path>`

Finds unused functions, variables, imports, classes.  
Modes: `all` (all unused), `exports` (only exported but unused), `params` (unused parameters)

**Example**:  
`code-whisperer dead-code src/ --mode=exports`

### `code-whisperer style <path> --language=<lang>`

Enforces language-specific style guides.  
Languages: `python` (Black + isort), `javascript` (Prettier + ESLint airbnb), `typescript` (Prettier + @typescript-eslint), `bash` (ShellCheck strict), `ruby` (standardrb), `go` (golangci-lint), `docker` (Hadolint), `yaml` (yamllint), `markdown` (markdownlint)

**Example**:  
`code-whisperer style src/ --language=python`

### `code-whisperer refactor <file> --suggestion=<id>`

Applies specific refactoring based on detected pattern.  
**Suggestion IDs**: `extract-method`, `inline-variable`, ` introduce-parameter-object`, `split-class`, `replace-conditional-with-polymorphism`, `remove-unused-imports`, `simplify-boolean`, `consolidate-duplicate`

**Example**:  
`code-whisperer refactor src/auth.py --suggestion=split-class --dry-run`

### `code-whisperer history <file>`

Shows evolution of code quality metrics over git history.  
Outputs: `complexity-trend`, `duplication-trend`, `smells-added`, `smells-removed`

**Example**:  
`code-whisperer history src/models/user.py`

### `code-whisperer baseline <path>`

Creates quality baseline snapshot (`.openclaw/code-quality-baseline.json`) for future comparison.  
Stores: metrics per file, detected smells, complexity scores, duplication blocks

**Example**:  
`code-whisperer baseline src/ --include-tests`

### `code-whisperer diff <commit> [--against=<commit>]`

Compares code quality between commits/refs.  
Shows: new smells, fixed smells, metric delta, regression hotspots

**Example**:  
`code-whisperer diff HEAD~5..HEAD --against=main`

### `code-whisperer quick-fix <file>`

Applies ALL safe, automatic fixes (unused import removal, simple formatting, trivial simplifications)  
**Never**: changes logic, removes functionality, modifies public API

**Example**:  
`code-whisperer quick-fix src/main.py`

### `code-whisperer whitelist <file> --add=<smell>`

Marks specific smell in specific file as acceptable (stored in `.openclaw/code-whisperer-whitelist.json`).  
Use when smell is intentional/contextual.

**Example**:  
`code-whisperer whitelist src/legacy.py --add=god-object`

## Work Process

Real workflow:

1. **Initial analysis**: `code-whisperer analyze . --file=.py,.js,.ts --pattern=all`  
   Generates report: `code-quality-report.json` + `code-quality-summary.md`

2. **Review suggestions**: Open `code-quality-summary.md` - lists each issue with:
   - File:line
   - Pattern type
   - Severity (low/medium/high/critical)
   - Why it matters (specific to pattern)
   - Suggested refactoring (concrete, file/line specific)
   - Effort estimate (1=minutes, 3=hours, 5=days)

3. **Apply selective fixes**:  
   Manual: edit files per suggestions  
   Auto: `code-whisperer refactor <file> --suggestion=<id> --dry-run` then remove `--dry-run`  
   Bulk safe: `code-whisperer quick-fix src/`

4. **Verify**: `code-whisperer analyze . --pattern=all` again  
   Expect: reduced count, no new high-severity, metric improvements

5. **Re-run style checks**: `code-whisperer style . --language=<all>`  
   Fix formatting: `code-whisperer style . --language=<lang> --fix`

6. **Update baseline**: `code-whisperer baseline . --include-tests`  
   Commits: `git add .openclaw/code-quality-baseline.json && git commit -m "fix: improve code quality"`

7. **CI integration**: Add to `.github/workflows/quality.yml`:
   ```yaml
   - name: Code Whisperer Scan
     run: |
       code-whisperer analyze src/ --pattern=all --fail-on=high
       code-whisperer style src/ --language=all
       code-whisperer dead-code src/
   ```

## Golden Rules

1. **Never enable `CODE_WHISPERER_AUTO_FIX=1` for refactorings** - only style/unused-imports. Refactorings need human review.
2. **Always use `--dry-run` first** for any refactor command. Review diff with `git diff` before applying.
3. **Whitelist intentionally complex code** (performance hotspots, algorithmic clarity) - don't "fix" what's already optimal.
4. **Run analysis per-language** (separate Python, JS, TS runs) - cross-language pattern detection is unreliable.
5. **Exclude generated code** (protobuf, swagger, migrations, build outputs) via `--exclude=**/*_pb2.py,**/generated/`.
6. **Baseline BEFORE major refactor** - `code-whisperer baseline .` so you can measure improvement.
7. **Treat `god-object` >2K lines as critical** - schedule immediate split; <500 lines may be acceptable.
8. **Feature envy**: only refactor if target class exists and makes sense; otherwise consider extract class.
9. **Duplication threshold**: 70% similarity, 10+ lines minimum - smaller coincidences are false positives.
10. **Complexity**: cyclomatic >15 requires immediate attention; 10-14 schedule for next sprint.
11. **Never refactor in hotfix branches** - quality improvements separate from bug fixes.
12. **Document whitelist decisions** in file comments: `# code-whisperer-whitelist: god-object - this module orchestrates complex workflow, split would reduce readability`
13. **Dead code**: run `--mode=exports` first; avoid removing truly dynamic imports (`__import__`, `require()` with variable).
14. **Comment-code ratio >0.5** usually indicates need for better naming/structuring, not comment removal.
15. **Security scan**: `code-whisperer analyze --pattern=security` (separate pattern set: hardcoded-secrets, sql-injection-risk, shell-injection-risk, weak-crypto)

## Examples

### Example 1: Finding God Object

**User input**:  
`code-whisperer analyze src/controllers/ --pattern=god-object`

**Output** (real):
```
Analyzing 12 Python files...
Detected 2 god-object smells:

1. src/controllers/admin_controller.py:1
   Class AdminController (847 lines, 28 methods)
   Metrics: cyclomatic=24, methods-per-class=28, lines-per-method=avg 30
   Suggestion: Split by responsibility:
     - Authentication (login, logout, sessions)
     - User Management (create, delete, roles)
     - Audit Logging (all audit_* methods)
     - Reporting (export_*, generate_*)
   Effort: 3 (days)
   Severity: high

2. src/controllers/report_controller.py:1
   Class ReportController (612 lines, 22 methods)
   Suggestion: Extract ReportGenerator, ReportExporter, ReportScheduler classes
   Effort: 2
   Severity: medium
```

### Example 2: Feature Envy Detection

**User input**:  
`code-whisperer analyze src/models/ --pattern=feature-envy`

**Output**:
```
Feature Envy detected:

File: src/models/order.py, line 156
Method: Order.calculate_discount()
Accesses Order.customer (4 times) and Customer.tier (2 times)
vs accesses Order fields (1 time: self.status)
Recommendation: Move calculate_discount() to Customer class
  - Customer has discount_rules, tier, purchase_history
  - Order only has order_items, status, total
Effort: 1 (hours to move + update callers)
```

### Example 3: Complexity Analysis

**User input**:  
`code-whisperer complexity src/utils/validation.py --metric=cyclomatic`

**Output**:
```
File: src/utils/validation.py
Method: validate_user_input: cyclomatic complexity = 14
  - if user_type == 'admin': (1)
  - if permissions & 0x4: (2)
  - for role in roles: (3)
  - if not is_valid_email(email): (4)
  - if phone and not phone.match(PATTERN): (5)
  - if country not in COUNTRIES: (6)
  - try: json.loads(data) (7)
  - except: (8)
  - if metadata: (9)
  - for key in metadata: (10)
  - if key in RESERVED: (11)
  - if len(data) > MAX: (12)
  - if rate_limit_exceeded(): (13)
  - log_error(): (14)
Threshold: 10 (FAIL)
Suggest: Extract validate_email, validate_phone, validate_metadata, check_rate_limit
```

### Example 4: Duplication Detection

**User input**:  
`code-whisperer analyze src/ --pattern=duplication`

**Output**:
```
Duplicate code blocks found:

Block 1: src/services/user_service.py:45-58
Block 2: src/services/product_service.py:32-45
Similarity: 73% (15/20 lines identical except variable names)
Pattern:
  try:
    db.begin()
    # ... specific operations
    db.commit()
  except:
    db.rollback()
    raise
Suggestion: Extract TransactionManager context manager to utils/transaction.py
  @contextmanager
  def transactional(session):
      try:
          yield session
          session.commit()
      except:
          session.rollback()
          raise
Effort: 1 (create utils, update 2 callers)
```

### Example 5: Dead Code Export Scan

**User input**:  
`code-whisperer dead-code src/ --mode=exports`

**Output**:
```
Unused exported symbols:

src/api/__init__.py:
  - export 'LegacyV1Handler' (imported nowhere)
  - export 'calculate_legacy_checksum' (not used)
  - export 'DEPRECATED_ERROR_CODES' (unreferenced)

src/utils/format.py:
  - function format_currency_old() (replaced by format_currency_v2)

src/models/constants.py:
  - constant OLD_STATUS_MAP (new code uses STATUS_ENUM)

Total: 5 unused exports. Review before removal (some may be public API).
```

### Example 6: Quick-Fix Safe Changes

**User input**:  
`code-whisperer quick-fix src/`

**Output**:
```
Applying safe automatic fixes...

src/utils/helpers.py:
  - Removed unused import 'os' (line 3)
  - Removed unused variable 'temp_path' (line 12)
  - Consolidated duplicate 'if x is None' checks (lines 15, 22)

src/models/user.py:
  - Removed unused parameter 'debug=False' from __init__
  - Fixed trailing whitespace (3 occurrences)

src/api/endpoints.py:
  - Removed dead code: commented-out block 45-52

Summary: 8 fixes applied. Review with: git diff
WARNING: No logic changes made. Always run tests.
```

### Example 7: Refactor with Dry-Run

**User input**:  
`code-whisperer refactor src/parser.py --suggestion=extract-method --dry-run`

**Output**:
```
Dry run: extract-method suggestion #42
Target: src/parser.py, lines 120-156: "Complex parsing logic with nested conditionals"

Proposed extraction:
  New method: _parse_metadata_block(self, tokens: List[str]) -> Metadata
  Lines: 120-156 (37 lines) -> becomes 5-line call + new method
  Complexity reduction: from cyclomatic 12 to 4 (main) + 6 (new)
  Variables extracted: metadata_dict, validation_errors, block_type
  No new dependencies introduced.
  
To apply: code-whisperer refactor src/parser.py --suggestion=42
```

### Example 8: Creating Baseline

**User input**:  
`code-whisperer baseline src/ --include-tests`

**Output**:
```
Baseline created: .openclaw/code-quality-baseline.json
Scanned: 47 files (42 source, 5 test)
Metrics captured:
  - Cyclomatic complexity avg: 4.2, max: 18 (src/controllers/payment.py:87)
  - Duplication blocks: 3 (affecting 6 files)
  - God objects: 1 (src/controllers/admin_controller.py)
  - Feature envy: 4
  - Comment-code ratio: 0.34
  - Dead exports: 12

Next time: code-whisperer analyze . --compare-baseline
Will show: +3 new smells, -1 fixed, cyclomatic +0.2 delta
```

## Rollback Commands

### Undo specific refactoring
`git checkout -- src/<file>` (if not committed)  
or `git revert <commit-hash>` (if committed)

### Undo `quick-fix` batch changes
`git diff -- . ':!*.md' | grep -E '^[-+]' | cut -c2- | xargs -I{} git checkout -- "{}"`

Better: `git checkout .` (resets all non-committed changes) after reviewing what quick-fix did

### Remove whitelist entry
`jq 'del(.whitelist["src/legacy.py"])' .openclaw/code-whisperer-whitelist.json > tmp && mv tmp .openclaw/code-whisperer-whitelist.json`

### Revert baseline to previous state
1. Find old baseline in git: `git log -- .openclaw/code-quality-baseline.json`
2. Restore: `git checkout <old-hash> -- .openclaw/code-quality-baseline.json`

### Un-apply suggestion that caused regression
If you applied suggestion ID 42:
- Check which commit: `git log --oneline --grep="suggestion 42"`
- Revert that commit: `git revert <commit>`
- Or if not committed: `git checkout -- src/<file>`

### Restore accidentally removed dead code
Check git log for file: `git log -p -- src/<file>`  
Find deleted function, cherry-pick that commit segment or:  
`git checkout <commit-before-removal>^ -- src/<file>`

### Disable auto-fix mode if enabled
`unset CODE_WHISPERER_AUTO_FIX`  
or remove from shell profile: `sed -i '/CODE_WHISPERER_AUTO_FIX/d' ~/.bashrc ~/.zshrc`

### Clear generated report files
`rm -f code-quality-*.json code-quality-*.md .openclaw/code-whisperer-whitelist.json`
(whitelist caution: preserves intentional decisions)

### Full system reset (last resort)
`rm -rf .openclaw/ && code-whisperer baseline .`  
(WARNING: loses all history, whitelist, previous baselines)

---

## Troubleshooting

| Issue | Command |
|-------|---------|
| "Pattern X not found" | Ensure language plugin installed (e.g., `pip install code-whisperer-python`) |
| Slow analysis on large codebase | Use `--exclude=tests/,**/node_modules/,**/venv/` |
| False positive on duplication | Adjust threshold: `--similarity=80` (needs 80% not 70%) |
| Missing language support | Check dependencies: `code-whisperer --list-languages` |
| CI failing on high severity | Temporarily whitelist: `code-whisperer whitelist <file> --add=<smell>` |
| Auto-fix breaking code | Disable: `unset CODE_WHISPERER_AUTO_FIX`, revert changes |
| Complexity not reducing after refactor | Some complexity is inherent (e.g., parsers). Document in whitelist. |
| Dead code removal breaks dynamic imports | Use `--mode=exports` (only catches explicit imports). Review manually. |

## CLI Reference Summary

```bash
# Analyze all patterns
code-whisperer analyze . --pattern=all

# Fix style only
code-whisperer style . --fix

# Find unused exports
code-whisperer dead-code src/ --mode=exports

# See complexity hotspots
code-whisperer complexity . --metric=cyclomatic

# Apply one specific refactoring
code-whisperer refactor <file> --suggestion=<id>

# Create baseline before refactor
code-whisperer baseline .

# Compare against baseline
code-whisperer analyze . --compare-baseline

# Language-specific style with auto-fix
code-whisperer style src/ --language=python --fix
```
```