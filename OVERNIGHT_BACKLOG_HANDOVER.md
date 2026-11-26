# AnyPS5 Overnight Handover Report (`NobleBrains/AnyPS5` -> `boykopovar/AnyPS5`)

**Stand:** 2026-11-26 21:25 CET  
**Zweck:** Vollständige Übergabe an eine zweite Antigravity-Instanz auf macOS für den Über-Nacht-Betrieb (`/goal` oder `/schedule`).

---

## 1. Setup, Remotes & harte Regeln

- **Upstream-Repo:** `boykopovar/AnyPS5` (Remote `origin`)
- **Fork-Repo:** `NobleBrains/AnyPS5` (Remote `fork`, `gh` Account = `NobleBrains`)
- **PR-Slot-Limit:** Auf `boykopovar/AnyPS5` sind maximal **20 offene PRs** gleichzeitig erlaubt.
  - **Puffer-Regel:** Halte immer **2 Slots als Puffer** frei!
  - Öffne neue PRs nur, solange `NobleBrains` **weniger als 18 offene PRs** hat (also auf maximal **18** auffüllen).
- **Pflichtregeln im Repository:**
  - Lies `AGENTS.md`, `CONVENTIONS.md` und `CONTRIBUTING.md`.
  - **Keine Kommentare in C++-Code!** (`//` oder `/* ... */` im C++-Code führt zum Fehlschlag von `check_conventions.py`).
  - **Keine direkten Guest-Pointer-Dereferenzierungen:** Verwende immer `CopyGuestIn` / `CopyGuestOut` / `GuestCopyIn_nid_no_patch` / `GuestCopyOut_nid_no_patch` / `CopyGuestString` unter Erhalt von `errno`.
  - **`docs/dev/debt/changes/*.md` Pfadregeln:**
    - Relative Links im Fragment beziehen sich auf `docs/dev/debt/` -> **`../../../core/...`** (NICHT `../../../../core/...`!).
    - Jede `## change <needle>`-Überschrift darf **keine** umschließenden Klammern um das Suchmuster haben, es sei denn, `[` und `]` stehen buchstäblich im Original-Eintrag (z. B. `[link title]`), und muss **exakt 1** Eintrag in `docs/dev/debt/<library>.md` treffen.
    - **`tools/check_conventions.py` prüft `HEAD`-Commits!** Also erst `git commit` ausführen, dann `python3 tools/check_conventions.py` laufen lassen (und bei Bedarf `git commit --amend`).
  - **`backlog-handover` selbst:** Dieser Branch ist nur die Übergabe-Doku auf `fork` — **niemals** als PR auf `boykopovar/AnyPS5` einreichen!

---

## 2. Aktueller Stand der 18 offenen PRs auf `boykopovar/AnyPS5`

Alle 18 PRs sind aktuell `OPEN` (keine offenen To-Dos von unserer Seite — auch `#27513` und `#27550` sind bereits beantwortet):

| PR | Branch (`NobleBrains:...`) | Status / Letztes Review (`MarcZX10`) | Hinweis |
|---|---|---|---|
| `#27510` | `fix/libkernel-mapper-guest-copy` | `APPROVED` | Merge-ready |
| `#27511` | `fix/libc-exp-guest-copy` | `COMMENTED` (keine Blocker offen) | Warten auf Review/Merge |
| `#27512` | `fix/libc-jumpbuffer-guest-copy` | `COMMENTED` (keine Blocker offen) | Warten auf Review/Merge |
| `#27513` | `test/movbe-rsp-store-execution` | `CHANGES_REQUESTED` | Bereits per Kommentar beantwortet/geklärt |
| `#27514` | `fix/guest-pointer-audit-broken-pipe` | `APPROVED` | Merge-ready |
| `#27515` | `fix/debt-report-broken-pipe` | `COMMENTED` (keine Blocker offen) | Warten auf Review/Merge |
| `#27516` | `docs/trim-debt-entries-batch` | `APPROVED` | Merge-ready |
| `#27544` | `fix/regmgr-string-pointer-probing` | `APPROVED` | Merge-ready |
| `#27545` | `fix/posixwebkit-freeaddrinfo-probing` | `COMMENTED` (keine Blocker offen) | Warten auf Review/Merge |
| `#27546` | `fix/freetype-size-transform-probing` | `COMMENTED` (keine Blocker offen) | Warten auf Review/Merge |
| `#27549` | `fix/jxr-pointer-probing` | `COMMENTED` (`no blocking findings`) | Merge-ready |
| `#27550` | `fix/font-character-pointer-probing` | `CHANGES_REQUESTED` | False Positive bzgl. `startCopy`-Sync bei `break` in `Text.cpp:507`; bereits per Kommentar widerlegt |
| `#27551` | `fix/npmanager-value-class-probing` | `COMMENTED` (`no blocking findings`) | Merge-ready |
| `#27553` | `fix/libkernel-eventflag-guest-copy` | `APPROVED` | Merge-ready |
| `#27554` | `fix/libkernel-sem-guest-copy` | `COMMENTED` (keine Blocker offen) | Warten auf Review/Merge |
| `#27555` | `fix/libkernel-pthread-attr-guest-copy` | `COMMENTED` (keine Blocker offen) | Merge-ready |
| `#27557` | `docs/trim-debt-entries-batch-10` | `APPROVED` | Merge-ready |
| `#27558` | `docs/trim-debt-entries-batch-12` | `COMMENTED` (`no blocking findings`) | Merge-ready |

### Laufendes PR-Monitoring (z. B. alle 15 Minuten)

1. Prüfe Anzahl offener PRs von `NobleBrains`:
   ```bash
   gh pr list --repo boykopovar/AnyPS5 --author NobleBrains --state open --json number,title,headRefName,updatedAt
   ```
2. Prüfe bei den offenen PRs auf neue Reviews/Kommentare (von `MarcZX10`, `coderabbitai` oder Maintainer) sowie auf neue `MERGED`/`CLOSED`-Events.
   - Bei berechtigten Code-/Doku-Findings: Branch auschecken, fixen, lokal bauen + testen + `check_conventions.py` prüfen, auf `fork` pushen und kurz auf dem PR antworten.
   - Bei False Positives: Faktenbasiert mit Zeilennummern antworten (wie bei `#27550`).
   - Bei **Merged** oder **Closed**: Es wird ein Slot frei -> sofort Abschnitt 4 abarbeiten!

---

## 3. Vollständiger Backlog auf `fork` (`23` fertige Branches)

Alle 23 Branches basieren aktuell auf `origin/main` und wurden unter Linux bereits gebaut, getestet und mit `check_conventions.py` + `debt_fold.py --check` / `guest_pointer_audit.py` verifiziert.

### Priorität A: Code- & Guest-Pointer-Fixes (9 Branches — zuerst PRen!)

1. **`fix/audiopropagation-probe-descriptor`** *(Issue `#26882`)*
   - Library: `libSceAudioPropagation` (`requireDescriptor` über `copyGuestIn`)
   - Test-Target: `guest_audio_propagation` (`GuestAudioPropagation.cpp`)
   - PR-Body-Hinweis: `Closes #26882`
2. **`fix/json2-probe-string-text`** *(Issue `#26881`)*
   - Library: `libSceJson2` (`GuestTextLength` mit `errno`-Erhalt, `GuestBufferReadable`)
   - Test-Targets: `guest_json_string_search`, `guest_json_string_text` (`GuestJsonStringText.cpp`)
   - PR-Body-Hinweis: `Closes #26881`
3. **`fix/libkernel-check-reachability-guest-copy`**
   - Library: `libkernel` (`sceKernelCheckReachability` in `File/src/Stdio.cpp`)
   - Test-Target: `guest_filesystem`
4. **`fix/libkernel-posix-attr-schedparam-guest-copy`**
   - Library: `libkernel` (`pthread_attr_getschedparam_nid_postfix` in `Pthread/Posix/Attr.cpp`)
   - Test-Target: `guest_kernel_sched_priority_affinity_bounds`
5. **`fix/webbrowserdialog-getresult-guest-copy`**
   - Library: `libSceWebBrowserDialog` (`sceWebBrowserDialogGetResult` `offsetof` `CopyOut`)
   - Test-Target: `guest_web_browser_dialog`
6. **`fix/savedata-mount-guest-copy`**
   - Library: `libSceSaveData.native` (`sceSaveDataMount` `GuestCopyIn_nid_no_patch`)
   - Test-Target: `guest_savedata_ps4`
7. **`fix/voice-init-guest-copy`**
   - Library: `libSceVoice` (`sceVoiceInit` `ReadGuest`)
   - Test-Target: `guest_voice`
8. **`fix/depth2-query-memory-guest-copy`**
   - Library: `libSceDepth2` (`sceDepth2QueryMemory` `GuestCopyIn`/`GuestCopyOut`)
   - Test-Targets: `guest_depth2`, `guest_depth2_query_failure`
9. **`fix/vdeccore-decode-output-guest-copy`**
   - Library: `libSceVdecCore` (`sceVdecCoreGetDecodeOutput` `CopyGuestIn`/`CopyGuestOut`)
   - Test-Target: `guest_vdec_core`

### Priorität B: Debt-Trim-Batches `<1000 Zeichen` (14 Branches, 270 Einträge)

10. `docs/trim-debt-entries-batch-15` (15 Einträge)
11. `docs/trim-debt-entries-batch-16` (20 Einträge)
12. `docs/trim-debt-entries-batch-17` (20 Einträge)
13. `docs/trim-debt-entries-batch-18` (20 Einträge)
14. `docs/trim-debt-entries-batch-19` (20 Einträge)
15. `docs/trim-debt-entries-batch-20` (20 Einträge)
16. `docs/trim-debt-entries-batch-21` (20 Einträge)
17. `docs/trim-debt-entries-batch-22` (20 Einträge)
18. `docs/trim-debt-entries-batch-23` (20 Einträge)
19. `docs/trim-debt-entries-batch-24` (20 Einträge)
20. `docs/trim-debt-entries-batch-25` (19 Einträge)
21. `docs/trim-debt-entries-batch-26` (20 Einträge)
22. `docs/trim-debt-entries-batch-27` (19 Einträge)
23. `docs/trim-debt-entries-batch-28` (17 Einträge)

---

## 4. Verbindlicher Ablauf vor JEDEM neuen PR (inkl. Duplikat- & Merge-Check)

Sobald `NobleBrains` **< 18 offene PRs** auf `boykopovar/AnyPS5` hat, nimm den nächsten Branch `<branch>` aus Priorität A (danach B):

### Schritt 1: Mit Upstream synchronisieren & rebasen
```bash
git fetch origin && git fetch fork
git checkout -B <branch> fork/<branch>
git rebase origin/main
```
*(Bei Rebase-Konflikten: Konflikt sauber auflösen oder bei vollständig schon gemergtem Inhalt zu Schritt 2 -> Branch verwerfen.)*

### Schritt 2: Prüfen, ob schon von jemand anderem gemerged oder als PR offen!
```bash
git log --oneline -n 40 origin/main
gh pr list --repo boykopovar/AnyPS5 --state open --limit 100 --json number,title,headRefName
gh issue view <issue_nr> --repo boykopovar/AnyPS5 --json state   # falls Branch zu einem Issue gehört
```
- **Für `fix/*`-Branches:**
  - Prüfe `git diff origin/main`. Ist der Diff leer, wurde er bereits auf `origin/main` gemerged -> **keinen PR öffnen**, Branch auf `fork` löschen (`git push fork --delete <branch>`), weiter zum nächsten Branch.
  - Führe `python3 tools/guest_pointer_audit.py <library>` aus. Wenn die betroffenen Funktionen auf `origin/main` bereits **0 Findings** haben oder bereits ein offener/gemergter PR genau dieselbe Library/Funktion fixt -> **keinen Duplikat-PR öffnen**, Branch verwerfen.
  - Wenn noch relevant: Auf macOS das zugehörige Test-Target bauen und per `ctest` ausführen, `guest_pointer_audit.py <library>` prüfen (`0` neue Findings) und `python3 tools/check_conventions.py` ausführen.
- **Für `docs/trim-debt-entries-batch-*`-Branches:**
  - Führe aus:
    ```bash
    python3 tools/debt_fold.py --check
    python3 tools/check_conventions.py
    python3 -m unittest tools/tests/test_debt_report.py tools/tests/test_debt_fold.py
    ```
  - **Wichtig:** Falls inzwischen jemand anderes einzelne Debt-Einträge auf `origin/main` gekürzt/entfernt oder ein Fragment in `docs/dev/debt/changes/*.md` angelegt hat, meldet `debt_fold.py --check` entweder `"matches 0 entries"` oder `"both target the same ... entry; merge them into one fragment"`.
    - In diesem Fall **nur diese bereits erledigten/kollidierenden Abschnitte** aus `docs/dev/debt/changes/docs-trim-debt-entries-batch-*.md` entfernen, `git commit --amend`, erneut alle 3 Checks laufen lassen!
    - Bleiben 0 Einträge übrig, Branch verwerfen (`git push fork --delete <branch>`).
  - Prüfe auch gegen **andere noch offene `docs/trim-debt-entries-*` PRs** (falls fremde PRs dazukommen), dass keine doppelten Einträge eingereicht werden.

### Schritt 3: Pushen & PR öffnen
```bash
git push --force-with-lease fork <branch>
gh pr create --repo boykopovar/AnyPS5 --base main --head NobleBrains:<branch> --title "<commit-titel>" --body "<kurze Zusammenfassung + lokal verifizierte Checks>"
```
Danach Open-PR-Count erneut prüfen — nie über **18** offene PRs gehen!

---

## 5. macOS (Apple Silicon M4) & selbstständiges Weiterarbeiten

- Diese Instanz läuft auf einem **Mac Mini M4** und darf Dependencies bei Bedarf selbst nachinstallieren (z. B. `cmake`, `ninja`, `python3`, `gh`, Vulkan/MoltenVK-Abhängigkeiten via Homebrew, falls für macOS-Builds nötig).
- **Wichtig für macOS-Builds:** Manche Tests/Libraries (z. B. `libSceAgcDriver` / Metal / Linux-only-Features) verhalten sich auf macOS anders als unter Linux. Prüfe für jeden Code-Fix-Branch zuerst das zugehörige Test-Target lokal auf macOS; falls ein Target auf macOS nicht gebaut werden kann, prüfe `CMakeLists.txt` bzw. `AGENTS.md` oder konzentriere dich zuerst auf die plattformunabhängigen PRX-/Tool-/Debt-Branches.
- **Wenn der 23-Branch-Backlog leer ist** und weiterhin `< 18` offene PRs vorhanden sind, arbeite selbstständig weiter:
  1. `python3 tools/guest_pointer_audit.py` — die nächsten kleinen Bibliotheken (3–6 Treffer) sauber auf 0 bringen (`libSceCesCs`, `libfreetype`, `libSceFreeTypeOl`, `libSceNgs2.native`, `libSceAudioOut`, `libSceImeBackend`, `libSceAjm.native`, `libSceNet`).
  2. Ältere offene Issues prüfen, die sich auf macOS/plattformübergreifend gut lösen und testen lassen, z. B. `#23391` (*separate fault helper for progress.py*) oder `#23480` (*agc_driver_packed_integer_native fails on Apple M1/Apple Silicon*).

