# Plán: Pracovní listy ke všem 27 kapitolám

## TL;DR

> **Quick Summary**: Vytvořit pracovní listy (worksheets) pro všech 27 kapitol SkriptaOS. Každý list obsahuje Zadání, Řešení (foldable), Kontrolu funkčnosti a Sebehodnocení 1-5.
>
> **Deliverables**:
> - 27× `pracovni-listy/NN-nazev/README.md` (jeden list na kapitolu)
> - 1× `pracovni-listy/README.md` (master index)
> - Aktualizace `README.md` (přidat sloupec "Cvičení" do tabulky)
>
> **Estimated Effort**: XL (27 worksheet files + 2 README changes)
> **Parallel Execution**: YES — 6 waves (foundation + 5 worksheet waves + README)
> **Critical Path**: Wave 1 → Waves 2-6 (parallel) → Wave 7 (README) → Final Wave

---

## Context

### Original Request
Vytvořit "Pracovní listy" pro každou kapitolu (00-26) s obsahem: úkol, řešení, kontrola funkčnosti, hodnocení 1-5. 1 varianta na kapitolu.

### Interview Summary
- **Formát**: `pracovni-listy/NN-zkraceny-nazev/README.md` (GitHub auto-render)
- **Počet**: 1 varianta na kapitolu, celkem 27 listů
- **Struktura**: Volná dle typu kapitoly — Zadání → Řešení → Kontrola → Sebehodnocení
- **Jazyk**: Čeština + anglické commandy
- **README**: Přidat sloupec "Cvičení" do tabulky
- **Řešení**: Schované pod `<details>` tagem (foldable, student nejdříve zkusí sám)
- **Chybějící kapitoly**: Ne — komplet všech 27 (00–26)

### Metis Review (self-performed)
**Identified gaps**:
- **Master index**: Potřeba `pracovni-listy/README.md` s přehledem všech listů → resolved (vytvořit)
- **Foldable řešení**: Aby student neviděl řešení hned → resolved (použít `<details>` tag)
- **Windows kapitoly (03, 05)**: Jiné commandy, PowerShell místo bash → resolved (upravit formát)
- **Teoretické kapitoly (00)**: Žádné commandy pro kontrolu → resolved (kontrola formou Q&A)

---

## Work Objectives

### Core Objective
Vytvořit sadu 27 praktických pracovních listů, jeden ke každé kapitole SkriptaOS. Každý list umožní studentovi ověřit si pochopení látky formou samostatného úkolu s možností kontroly.

### Concrete Deliverables
- `pracovni-listy/README.md` — master index s přehledem všech listů
- `pracovni-listy/00-zaklady-os/README.md` až `pracovni-listy/26-web-server/README.md` — 27 worksheetů
- Úprava `README.md` — přidat sloupec "Cvičení" do tabulky

### Definition of Done
- [ ] Všech 27 adresářů existuje v `pracovni-listy/`
- [ ] Každý adresář obsahuje `README.md` se 4 sekcemi (Zadání, Řešení, Kontrola, Sebehodnocení)
- [ ] Master index `pracovni-listy/README.md` odkazuje na všechny listy
- [ ] Hlavní `README.md` má sloupec "Cvičení" s odkazy
- [ ] Každý list má zpětný odkaz na kapitolu a hlavní README

### Must Have
- Všechny kapitoly 00–26 mají svůj pracovní list
- Každý list má `<details>` foldable sekci pro řešení
- Každý list má škálu sebehodnocení 1-5
- Každý list odkazuje zpět na svoji kapitolu a hlavní přehled
- README.md tabulka má nový sloupec "Cvičení"

### Must NOT Have (Guardrails)
- Žádné automatické testy/hodnocení (pouze sebehodnocení studenta)
- Žádné HTML stránky nebo interaktivní verze — pouze Markdown
- Žádné duplicitní kopírování celé kapitoly — jen úkoly
- Žádné oddělené řešení (answer key) — řešení je v `<details>` v tomtéž souboru
- Maximálně 1 soubor na adresář (pouze README.md)

---

## Verification Strategy

> **ZERO HUMAN INTERVENTION** — ALL verification is agent-executed.

### QA Policy
- **File existence**: `test -f pracovni-listy/NN-nazev/README.md` pro každou kapitolu
- **Content structure**: grep na `## Zadání`, `## Řešení`, `## Kontrola funkčnosti`, `## Sebehodnocení`
- **Foldable řešení**: grep na `<details>` a `</details>` v každém listu
- **Škála 1-5**: grep na `— 1`, `— 5` (přítomnost celé škály)
- **Zpětné odkazy**: grep na zpětný odkaz na kapitolu a README
- **README tabulka**: grep na `Cvičení` v hlavičce tabulky

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (Foundation — prepares directories):
├── Task 1: Create directory structure + master index
└── Task 2: Create template example file (optional)

Waves 2-6 (Worksheets — ALL INDEPENDENT, max parallel):
├── Wave 2: 00, 01, 02, 03, 04, 05 (6 files)
├── Wave 3: 06, 07, 08, 09, 10, 11 (6 files)
├── Wave 4: 12, 13, 14, 15, 16, 17 (6 files)
├── Wave 5: 18, 19, 20, 21, 22, 23 (6 files)
└── Wave 6: 24, 25, 26 (3 files)

Wave 7 (Integration):
├── Task: Update README.md table (new column)

Wave FINAL (Verification):
├── F1: File existence + content audit
├── F2: Cross-reference check
├── F3: Content quality review
└── F4: Scope fidelity
```

---

## Template (for all worksheets)

```markdown
# Pracovní list: {Název kapitoly}

> {One-line description}

---

## Zadání

### Úkol 1: {title}
{description of task}

### Úkol 2: {title}
{description of task}

### Úkol 3: {title}
{description of task}

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení
{correct answer / procedure}

### Úkol 2 — Řešení
{correct answer / procedure}

### Úkol 3 — Řešení
{correct answer / procedure}

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] {verification step 1}
- [ ] {verification step 2}
- [ ] {verification step 3}

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../NN-soubor.md) · [Zpět na přehled](../../README.md)
```

---

## TODOs

### Wave 1: Foundation

- [x] 1. **Create directory structure + master index**

  **What to do**:
  - `mkdir -p pracovni-listy/` and all 27 subdirectories
  - Create `pracovni-listy/README.md` as master index listing all chapters with links
  - Master index structure: table with #, Kapitola, Odkaz na list

  **Template for master index**:
  ```markdown
  # Pracovní listy — SkriptaOS

  Přehled pracovních listů ke všem kapitolám. Každý list obsahuje zadání,
  řešení, kontrolu funkčnosti a sebehodnocení.

  | # | Kapitola | Pracovní list |
  |---|----------|---------------|
  | 00 | [Základy OS](../00-zaklady-os.md) | [Cvičení](./00-zaklady-os/) |
  | 01 | [Základní Linux příkazy](../01-linux-prikazy-zakladni.md) | [Cvičení](./01-linux-prikazy-zakladni/) |
  ...
  ```

  **Must NOT do**:
  - Nezahrnovat řešení nebo obsah listů do master indexu
  - Nepoužívat absolutní cesty (musí fungovat relative z `pracovni-listy/`)

  **Recommended Agent Profile**:
  > Category: `quick` — triviální mkdir + šablonovitý soubor

  **Can Run In Parallel**: NO (foundation, blocks nothing)
  **Blocks**: All worksheet tasks (directories must exist)
  **Blocked By**: None

  **Acceptance Criteria**:
  - [ ] `ls -d pracovni-listy/*/` → 27 directories
  - [ ] `head -1 pracovni-listy/README.md` → "# Pracovní listy"
  - [ ] `grep -c '| [0-9][0-9] |' pracovni-listy/README.md` → 27

---

- [x] 2. **Update main README.md — add Cvičení column**

  **What to do**:
  - Transform table in `/home/petrg/Project/SkriptaOS/README.md` from 3 columns to 4
  - Add `| Cvičení |` to header: `| # | Sekce | Popis | Cvičení |`
  - Add `|---|` separator
  - For each row: add `| [Cvičení](pracovni-listy/NN-nazev/) |`
  - Example: `| 00 | [Základy OS](00-zaklady-os.md) | Teoretický základ... | [Cvičení](pracovni-listy/00-zaklady-os/) |`

  **Must NOT do**:
  - Neměnit obsah stávajících sloupců (Sekce, Popis)
  - Zachovat stejné pořadí řádků

  **Recommended Agent Profile**:
  > Category: `quick` — jedna tabulková transformace

  **Can Run In Parallel**: NO (must exist after worksheets are done, or can be done anytime since links are relative)
  **Blocks**: Nothing (link works even before directories exist on GitHub)
  **Blocked By**: None (can run immediately)

  **Acceptance Criteria**:
  - [ ] `head -15 README.md | grep 'Cvičení'` → exists in header row
  - [ ] `grep -c '\[Cvičení\]' README.md` → 27 (one per chapter)
  - [ ] `grep '| 26 |' README.md | grep 'Cvičení'` → row 26 has the column

---

### Wave 2: Kapitoly 00–05 (teoretická + command-based)

- [x] 3. **Create worksheet: 00-zaklady-os**
- [x] 4. **Create worksheet: 01-linux-prikazy-zakladni**
- [x] 5. **Create worksheet: 02-linux-prikazy-rozirene**
- [x] 6. **Create worksheet: 03-windows-prikazy**
- [x] 7. **Create worksheet: 04-bash-skriptovani**
- [x] 8. **Create worksheet: 05-powershell-skriptovani**
- [x] 9. **Create worksheet: 06-sprava-procesu**
- [x] 10. **Create worksheet: 07-bezpecna-architektura**
- [x] 11. **Create worksheet: 08-ovladani-soukromosti**
- [x] 12. **Create worksheet: 09-optimalizace-vykony**
- [x] 13. **Create worksheet: 10-aktualizace-systemu**
- [x] 14. **Create worksheet: 11-sprava-uzivatelu**
- [x] 15. **Create worksheet: 12-prava-souboru**
- [x] 16. **Create worksheet: 13-sprava-filesystemu**
- [x] 17. **Create worksheet: 14-zalohovani**
- [x] 18. **Create worksheet: 15-apparmor**
- [x] 19. **Create worksheet: 16-netplan**
- [x] 20. **Create worksheet: 17-raid-redundance-disku**
- [x] 21. **Create worksheet: 18-proxmox-raid**
- [x] 22. **Create worksheet: 19-prace-s-ssh**
- [x] 23. **Create worksheet: 20-systemd**
- [x] 24. **Create worksheet: 21-monitoring-systemu**
- [x] 25. **Create worksheet: 22-reseni-problemu**
- [x] 26. **Create worksheet: 23-selinux**
- [x] 27. **Create worksheet: 24-firewall**
- [x] 28. **Create worksheet: 25-bezpecnostni-audit**
- [x] 29. **Create worksheet: 26-web-server**

  **Chapter type**: System config (Apache, Nginx, PHP-FPM, gunicorn, PM2, ACME, logování)
  **Tasks**: Nginx server block config, PHP-FPM pool setup, log analysis, reverse proxy test
  **Backlink**: `../../26-web-server.md`

  **Parallelization**: Can run parallel
  **Blocked By**: Task 1

---

## Final Verification Wave

- [x] F1. **File Existence Audit** — `bash` (27/27 PASS)
- [x] F2. **Content Structure Audit** — `bash` (ALL sections, foldable, scale, backlinks PASS)
- [x] F3. **README Integration Check** — `bash` (Column OK, Links 27/27, Master index 27/27 PASS)
- [x] F4. **Scope Fidelity Check** — `deep` (SCOPE CLEAN PASS)
  No extra files beyond README.md in worksheet directories. No content beyond spec.
  Output: `Scope [CLEAN/ISSUES] | VERDICT`

---

## Commit Strategy

- **All files**: `docs: add worksheets for chapters 00-26`

---

## Success Criteria

- [x] 27 worksheet README.md files exist
- [x] All have Zadání + Řešení + Kontrola + Sebehodnocení
- [x] All have `<details>` foldable sections
- [x] All have škála 1-5
- [x] All have zpětné odkazy
- [x] README.md má sloupec Cvičení
- [x] Master index `pracovni-listy/README.md` existuje
