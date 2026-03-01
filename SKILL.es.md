name: Code Whisperer
description: Analiza inteligentemente la calidad del código, identifica patrones y provide orientación suave para mejoras
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
```

# Code Whisperer

Motor inteligente de análisis y mejora de calidad de código que aprende de tu codebase y proporciona sugerencias contextuales y no intrusivas para un código más limpio y mantenible.

## Propósito

Casos de uso reales:
- **Detectar God Objects**: Encontrar clases/módulos con >500 líneas y >15 métodos, sugerir división por responsabilidad
- **Detectar Feature Envy**: Identificar métodos que acceden a datos de otro objeto >3x más que a los propios, recomendar refactorización move method
- **Descubrir Duplicación**: Detectar bloques de código copia-pega con >70% de similitud entre archivos, sugerir extracción
- **Marcar Condicionales Complejos**: Encontrar anidación if/else depth >3, proponer guard clauses o strategy pattern
- **Resaltar Complejidad Incidental**: Localizar código con exceso de comentarios (ratio comentarios:código >0.5), indicar necesidad de código autodocumentado
- **Encontrar Data Clumps**: Rastrear listas de parámetros >4 ítems pasados juntos, sugerir parameter object
- **Exponer Shotgun Surgery**: Rastrear cambios simples que afectan >5 archivos, recomendar facade u observer
- **Revelar Speculative Generality**: Identificar clases/interfaces abstractas no utilizadas, recomendar eliminación
- **Detectar Primitive Obsession**: Encontrar campos string/number que deberían ser value objects
- **Cazar Contrived Hierarchy**: Detectar herencia usada donde composition sería más simple

## Alcance

Comandos exactos:

### `code-whisperer analyze <path> [--file=<ext>] [--pattern=<type>] [--fix]`

Analiza archivos/directorios en busca de code smells.  
**Patrones reales**: `god-object`, `feature-envy`, `duplication`, `complex-conditional`, `data-clump`, `shotgun-surgery`, `speculative-generality`, `primitive-obsession`, `contrived-hierarchy`, `middle-man`, `lazy-class`, `comment-code`

**Ejemplo**:  
`code-whisperer analyze src/ --file=.py --pattern=duplication --fix`  
`code-whisperer analyze app/controllers --file=.js --pattern=feature-envy`

### `code-whisperer complexity <file> --metric=cyclomatic`

Calcula métricas de complejidad con umbrales.  
**Métricas**: `cyclomatic` (>10 advertencias), `cognitive` (>15), `npath` (>200), `maintainability-index` (<20)

**Ejemplo**:  
`code-whisperer complexity src/utils/parser.py --metric=cyclomatic`

### `code-whisperer dead-code <path>`

Encuentra funciones, variables, imports, clases no utilizadas.  
Modos: `all` (todo no usado), `exports` (solo exportados pero no usados), `params` (parámetros no usados)

**Ejemplo**:  
`code-whisperer dead-code src/ --mode=exports`

### `code-whisperer style <path> --language=<lang>`

Aplica guías de estilo específicas por lenguaje.  
Lenguajes: `python` (Black + isort), `javascript` (Prettier + ESLint airbnb), `typescript` (Prettier + @typescript-eslint), `bash` (ShellCheck strict), `ruby` (standardrb), `go` (golangci-lint), `docker` (Hadolint), `yaml` (yamllint), `markdown` (markdownlint)

**Ejemplo**:  
`code-whisperer style src/ --language=python`

### `code-whisperer refactor <file> --suggestion=<id>`

Aplica refactorización específica basada en patrón detectado.  
**IDs de sugerencia**: `extract-method`, `inline-variable`, ` introduce-parameter-object`, `split-class`, `replace-conditional-with-polymorphism`, `remove-unused-imports`, `simplify-boolean`, `consolidate-duplicate`

**Ejemplo**:  
`code-whisperer refactor src/auth.py --suggestion=split-class --dry-run`

### `code-whisperer history <file>`

Muestra evolución de métricas de calidad de código a través del historial de git.  
Salidas: `complexity-trend`, `duplication-trend`, `smells-added`, `smells-removed`

**Ejemplo**:  
`code-whisperer history src/models/user.py`

### `code-whisperer baseline <path>`

Crea snapshot de línea base de calidad (`.openclaw/code-quality-baseline.json`) para comparación futura.  
Almacena: métricas por archivo, smells detectados, puntajes de complejidad, bloques de duplicación

**Ejemplo**:  
`code-whisperer baseline src/ --include-tests`

### `code-whisperer diff <commit> [--against=<commit>]`

Compara calidad de código entre commits/refs.  
Muestra: nuevos smells, smells arreglados, delta de métricas, hotspots de regresión

**Ejemplo**:  
`code-whisperer diff HEAD~5..HEAD --against=main`

### `code-whisperer quick-fix <file>`

Aplica TODOS los fixes automáticos seguros (eliminación de imports no usados, formato simple, simplificaciones triviales)  
**Nunca**: cambia lógica, elimina funcionalidad, modifica API pública

**Ejemplo**:  
`code-whisperer quick-fix src/main.py`

### `code-whisperer whitelist <file> --add=<smell>`

Marca smell específico en archivo específico como aceptable (almacenado en `.openclaw/code-whisperer-whitelist.json`).  
Usar cuando el smell es intencional/contextual.

**Ejemplo**:  
`code-whisperer whitelist src/legacy.py --add=god-object`

## Proceso de Trabajo

Flujo de trabajo real:

1. **Análisis inicial**: `code-whisperer analyze . --file=.py,.js,.ts --pattern=all`  
   Genera reporte: `code-quality-report.json` + `code-quality-summary.md`

2. **Revisar sugerencias**: Abrir `code-quality-summary.md` - lista cada problema con:
   - Archivo:línea
   - Tipo de patrón
   - Severidad (baja/media/alta/crítica)
   - Por qué importa (específico al patrón)
   - Refactorización sugerida (concreto, archivo/línea específico)
   - Estimación de esfuerzo (1=minutos, 3=horas, 5=días)

3. **Aplicar fixes selectivos**:  
   Manual: editar archivos según sugerencias  
   Auto: `code-whisperer refactor <file> --suggestion=<id> --dry-run` luego quitar `--dry-run`  
   Bulk seguro: `code-whisperer quick-fix src/`

4. **Verificar**: `code-whisperer analyze . --pattern=all` de nuevo  
   Esperar: reducción en conteo, sin nuevas severidades altas, mejoras en métricas

5. **Re-ejecutar checks de estilo**: `code-whisperer style . --language=<all>`  
   Formato fix: `code-whisperer style . --language=<lang> --fix`

6. **Actualizar baseline**: `code-whisperer baseline . --include-tests`  
   Commits: `git add .openclaw/code-quality-baseline.json && git commit -m "fix: improve code quality"`

7. **Integración CI**: Añadir a `.github/workflows/quality.yml`:
   ```yaml
   - name: Code Whisperer Scan
     run: |
       code-whisperer analyze src/ --pattern=all --fail-on=high
       code-whisperer style src/ --language=all
       code-whisperer dead-code src/
   ```

## Reglas de Oro

1. **Nunca habilitar `CODE_WHISPERER_AUTO_FIX=1` para refactorizaciones** - solo estilo/imports no usados. Refactorizaciones necesitan revisión humana.
2. **Siempre usar `--dry-run` primero** para cualquier comando refactor. Revisar diff con `git diff` antes de aplicar.
3. **Whitelist código intencionalmente complejo** (hotspots de performance, claridad algorítmica) - no "fix" lo que ya es óptimo.
4. **Ejecutar análisis por-lenguaje** (corridas separadas Python, JS, TS) - detección de patrones cross-language es poco fiable.
5. **Excluir código generado** (protobuf, swagger, migrations, build outputs) via `--exclude=**/*_pb2.py,**/generated/`.
6. **Baseline ANTES de refactor mayor** - `code-whisperer baseline .` para poder medir mejora.
7. **Tratar `god-object` >2K líneas como crítico** - programar split inmediato; <500 líneas puede ser aceptable.
8. **Feature envy**: solo refactorizar si la clase destino existe y tiene sentido; si no, considerar extract class.
9. **Umbral de duplicación**: 70% similitud, 10+ líneas mínimo - coincidencias más pequeñas son falsos positivos.
10. **Complejidad**: cyclomatic >15 requiere atención inmediata; 10-14 programar para próximo sprint.
11. **Nunca refactorizar en branches hotfix** - mejoras de calidad separadas de fixes de bugs.
12. **Documentar decisiones de whitelist** en comentarios del archivo: `# code-whisperer-whitelist: god-object - este módulo orquesta flujo complejo, split reduciría legibilidad`
13. **Dead code**: ejecutar `--mode=exports` primero; evitar eliminar imports verdaderamente dinámicos (`__import__`, `require()` con variable).
14. **Ratio comentarios-código >0.5** usualmente indica necesidad de mejor naming/estructuración, no eliminación de comentarios.
15. **Security scan**: `code-whisperer analyze --pattern=security` (conjunto de patrones separado: hardcoded-secrets, sql-injection-risk, shell-injection-risk, weak-crypto)

## Ejemplos

### Ejemplo 1: Encontrar God Object

**Input del usuario**:  
`code-whisperer analyze src/controllers/ --pattern=god-object`

**Salida** (real):
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

### Ejemplo 2: Detección de Feature Envy

**Input del usuario**:  
`code-whisperer analyze src/models/ --pattern=feature-envy`

**Salida**:
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

### Ejemplo 3: Análisis de Complejidad

**Input del usuario**:  
`code-whisperer complexity src/utils/validation.py --metric=cyclomatic`

**Salida**:
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

### Ejemplo 4: Detección de Duplicación

**Input del usuario**:  
`code-whisperer analyze src/ --pattern=duplication`

**Salida**:
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

### Ejemplo 5: Escaneo de Código Muerto en Exports

**Input del usuario**:  
`code-whisperer dead-code src/ --mode=exports`

**Salida**:
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

### Ejemplo 6: Quick-Fix Cambios Seguros

**Input del usuario**:  
`code-whisperer quick-fix src/`

**Salida**:
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

### Ejemplo 7: Refactor con Dry-Run

**Input del usuario**:  
`code-whisperer refactor src/parser.py --suggestion=extract-method --dry-run`

**Salida**:
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

### Ejemplo 8: Crear Baseline

**Input del usuario**:  
`code-whisperer baseline src/ --include-tests`

**Salida**:
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

## Comandos de Rollback

### Deshacer refactorización específica
`git checkout -- src/<file>` (si no está committeado)  
o `git revert <commit-hash>` (si está committeado)

### Deshacer cambios batch `quick-fix`
`git diff -- . ':!*.md' | grep -E '^[-+]' | cut -c2- | xargs -I{} git checkout -- \"{}\"`

Mejor: `git checkout .` (resetea todos los cambios no committeados) después de revisar lo que quick-fix hizo

### Remover entrada de whitelist
`jq 'del(.whitelist[\"src/legacy.py\"])' .openclaw/code-whisperer-whitelist.json > tmp && mv tmp .openclaw/code-whisperer-whitelist.json`

### Revertir baseline a estado anterior
1. Encontrar baseline antiguo en git: `git log -- .openclaw/code-quality-baseline.json`
2. Restaurar: `git checkout <old-hash> -- .openclaw/code-quality-baseline.json`

### Des-aplicar sugerencia que causó regresión
Si aplicaste sugerencia ID 42:
- Check commit: `git log --oneline --grep="suggestion 42"`
- Revert ese commit: `git revert <commit>`
- O si no está committeado: `git checkout -- src/<file>`

### Restaurar código muerto eliminado accidentalmente
Check git log para archivo: `git log -p -- src/<file>`  
Encontrar función eliminada, cherry-pick ese commit segmento o:  
`git checkout <commit-before-removal>^ -- src/<file>`

### Deshabilitar modo auto-fix si está habilitado
`unset CODE_WHISPERER_AUTO_FIX`  
o remover de shell profile: `sed -i '/CODE_WHISPERER_AUTO_FIX/d' ~/.bashrc ~/.zshrc`

### Limpiar archivos de reporte generados
`rm -f code-quality-*.json code-quality-*.md .openclaw/code-whisperer-whitelist.json`
(cuidado whitelist: preserva decisiones intencionales)

### Reset completo del sistema (último recurso)
`rm -rf .openclaw/ && code-whisperer baseline .`  
(WARNING: pierde todo historial, whitelist, baselines previas)

---

## Solución de Problemas

| Issue | Command |
|-------|---------|
| "Pattern X not found" | Asegurar plugin de lenguaje instalado (ej., `pip install code-whisperer-python`) |
| Análisis lento en codebase grande | Usar `--exclude=tests/,**/node_modules/,**/venv/` |
| Falso positivo en duplicación | Ajustar umbral: `--similarity=80` (necesita 80% no 70%) |
| Soporte de lenguaje faltante | Check dependencias: `code-whisperer --list-languages` |
| CI fallando en severidad alta | Temporalmente whitelist: `code-whisperer whitelist <file> --add=<smell>` |
| Auto-fix rompiendo código | Deshabilitar: `unset CODE_WHISPERER_AUTO_FIX`, revertir cambios |
| Complejidad no reduciéndose después de refactor | Alguna complejidad es inherente (ej., parsers). Documentar en whitelist. |
| Dead code removal rompiendo imports dinámicos | Usar `--mode=exports` (solo captura imports explícitos). Revisar manualmente. |

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