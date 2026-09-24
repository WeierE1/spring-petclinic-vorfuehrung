# pr-100 — katalog nachtragen: Zielzweig aus dem Ereignis, Ruleset-Fall eindeutig

| | |
|---|---|
| PR | [WeierE1/spring-petclinic-vorfuehrung#100](https://github.com/WeierE1/spring-petclinic-vorfuehrung/pull/100) |
| Branch | `fix/katalog-nachtragen-basiszweig` → `1.5.x` |
| Merged | 2026-09-24 11:01 UTC |
| Size | +144 / −8 über 4 Dateien |
| Issues | — |
| Review | offen (Mensch) |

## 1. Why

Der Job `nachtragen` checkte fest `main` aus und pushte nach `main`. Diese
Kopie hat kein `main` (Standardzweig `1.5.x`, kein Ruleset) — gemessen
24.09.2026, Lauf 35985858831: „A branch or tag with the name 'main' could
not be found". Die Marken von pr-099 blieben deshalb stehen.

Entscheidung des Menschen vom 24.09.2026: der Job wird in allen drei Forks
repariert, die Rulesets der beiden anderen Forks bekommen **keine** Ausnahme
für GitHub Actions (§32/16 bleibt ohne Ausnahme). Die Datei ist in allen
drei Forks byte-gleich, deshalb trägt sie auch hier die Ruleset-Behandlung,
obwohl hier keines liegt.

## 2. What changed

`.github/workflows/katalog.yml`, nur Job `nachtragen` (Job `pruefen` und
`permissions` unverändert):

- Zielzweig aus dem Ereignis statt Literal `main`: `env: BASIS:
  ${{ github.event.pull_request.base.ref }}` auf Jobebene;
  `actions/checkout` mit `ref: ${{ github.event.pull_request.base.ref }}`;
  Push als `git push origin "HEAD:refs/heads/$BASIS"`, Rebase gegen
  `"$BASIS"`. In `run:` steht der Zweigname nur als Umgebungsvariable, nie
  als Ausdruck (Skript-Injektion über Zweignamen).
- Ruleset-Fall: enthält die Push-Ausgabe `GH013` oder `Repository rule
  violations`, wird nicht erneut versucht. Meldung in `$GITHUB_STEP_SUMMARY`
  und stderr („Ruleset verlangt einen Pull Request auf <Zweig> -- Marken
  dieses PR im naechsten PR von Hand fuellen: bash
  infra/katalog-nachtragen.sh <nr> …") und Exit 1. Andere Push-Fehler
  behalten Rebase und bis zu drei Versuche.
- PR-Nummer für die Meldung über `env: NUMMER` des Commit-Schritts.

`infra/katalog-nachtragen.sh` enthält kein fest verdrahtetes `main` und ist
unverändert.

Offene Marken:

- pr-099 → Merged `2026-09-24 10:12 UTC`, Size `+61 / −1 über 3 Dateien`
  (`GH_REPO=WeierE1/spring-petclinic-vorfuehrung bash infra/katalog-nachtragen.sh 99`).
- pr-023 **nicht** gefüllt: der Eintrag ist die Kopie des Pilot-Eintrags
  (verlinkt `WeierE1/spring-petclinic#23`), und #23 dieses Repositorys ist
  ein geschlossener Renovate-PR (`fix(deps): update dependency
  org.webjars:jquery to v4`). Das Skript hätte ihn abgewiesen; ein Wert aus
  dem Piloten wäre eine Deutung.

## 3. Files

| Path | Change |
|---|---|
| `.github/workflows/katalog.yml` | Job `nachtragen`: Zielzweig, Ruleset-Fall |
| `docs/changes/pr-099-renovate-preset-umzug.md` | Marken gefüllt |
| `docs/changes/README.md` | Indexzeile pr-099 gefüllt, Indexzeile pr-100 |
| `docs/changes/pr-100-katalog-nachtragen-basiszweig.md` | neu |

## 4. Verification

- Commit-Schritt aus der Datei ausgeschnitten und mit einem Stub für `git`
  gefahren (`BASIS=1.5.x`, `NUMMER=104`): GH013 → Meldung auf stderr und in
  der Zusammenfassung, **ein** Push-Versuch, Exit 1; Ablehnung ohne GH013 →
  drei Rebase-Versuche gegen `origin 1.5.x`, dann „Push nach drei Versuchen
  nicht moeglich.", Exit 1; Erfolg → `HEAD:refs/heads/1.5.x`, Exit 0.
- `grep -n main .github/workflows/katalog.yml` trifft nur noch Kommentare.
- Mojibake-Prüfung (`grep -c` auf U+00C3 über `docs/changes/*.md`): 0.
- `bash infra/katalog-pruefen.sh 100`: Katalogpflicht erfüllt (lokal vor dem
  Push).
- `actionlint` liegt nicht vor und wurde nicht installiert. Dass GitHub die
  Datei parst, belegt der Lauf von `katalog / pruefen` an diesem PR.

## 5. Known gaps

- **Dass `nachtragen` hier nach dem Merge grün läuft, ist nicht belegt** —
  hier gibt es kein Ruleset, der Push sollte durchgehen und die Marken dieses
  PR selbst füllen. Beleg ist der erste Post-Merge-Lauf.
- pr-023: Marken offen, Entscheidung beim Menschen (mit den Werten des
  Piloten füllen und als Kopie annotieren, oder so lassen).
- `vorfuehrung-basis` zeigt auf den Merge von #99 (`08c0c5e`), nicht auf
  diesen PR. Ein Reset (`infra/vorfuehrung/zuruecksetzen.sh` im
  CRA-Repository) setzt `1.5.x` auf den Tag zurück und nimmt diese Reparatur
  wieder heraus, bis der Tag nachgezogen ist. Kein Tag angefasst — Sache des
  Menschen.
- `infra/katalog-nachtragen.sh` erkennt eine spotless-ausgerichtete
  Kopfzeile (`| Size   |`) nicht und meldet trotzdem Exit 0 (gemessen in
  WebGoat). Hier ohne Folge, weil die Tabellen nicht ausgerichtet sind; nicht
  in diesem PR behoben.

## 6. Provenance

Geschrieben am 2026-09-24, im selben Zug wie der PR, von der Arbeitssitzung,
die den Job in allen drei Forks nach der Entscheidung des Menschen repariert
hat. Die Marken-Werte kommen aus `gh pr view`, nicht abgetippt.
