# Issues & Findings - Final Verification Wave

## F1: Structural Consistency — FAILED ❌

### H1 title number prefixes (4 files need fixing):
| File | Current | Should be |
|------|---------|-----------|
| 06-sprava-procesu.md | `# 06. Správa procesů v OS` | `# Správa procesů v OS` |
| 08-ovladani-soukromosti.md | `# 8. Ovládání soukromosti v OS` | `# Ovládání soukromosti v OS` |
| 09-optimalizace-vykony.md | `# 09: Optimalizace výkonu OS (Linux Kernel Tuning)` | `# Optimalizace výkonu OS (Linux Kernel Tuning)` |
| 10-aktualizace-a-aktualizace.md | `# 10. Aktualizace systému` | `# Aktualizace systému` |

### Missing ## Úvod sections (4 files):
| File | Current start |
|------|---------------|
| 06 | `## 1. Vytvoření procesu` |
| 08 | `## 8.1 Souborový systém a práva` |
| 09 | `## CPU` |
| 10 | `## 1. Správa balíčků` |

### Numbered ## Shrnutí (3 files):
| File | Current | Should be |
|------|---------|-----------|
| 06 | `## 8. Shrnutí` | `## Shrnutí` |
| 08 | `## 8.8 Shrnutí` | `## Shrnutí` |
| 10 | `## 6. Shrnutí` | `## Shrnutí` |

### Inconsistent table header (1 file):
- 06 line 11: `| Koncept \| Popis \| Příklad \| Možný výstup \|` → should be `Výstup`

### What PASSES:
✅ Files 01-05, 07 have proper H1 (no number prefix)
✅ Files 01-05, 07 have proper ## Úvod sections
✅ Files 01-05, 07, 09 have proper ## Shrnutí sections
✅ Table 4-column format present in all files

## F2: Examples with Outputs — PASSED ✅ (minor)
✅ Most tables have Výstup column with actual content
⚠️ File 06 line 11 has "Možný výstup" instead of "Výstup" (covered in F1 fix)

## F3: Bilingual Formatting — PASSED ✅
✅ All files: Czech explanations + English command examples

## F4: Navigation — FAILED ❌
✅ README links to all 10 files
❌ No backlinks from any content file to README (0 of 10 files)
