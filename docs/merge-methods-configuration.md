# Konfiguracja merge commit, squash i wyjątku synchronizacyjnego

W repozytorium ustawiliśmy merge commit do `main`, squash do `develop` oraz możliwość synchronizacji `main → develop` przez merge commit wykonany przez administratora.

Konfiguracja składa się z ustawień globalnych i trzech aktywnych rulesetów. Bypass administratora nie sprawdza gałęzi źródłowej PR-a: wyjątek synchronizacyjny jest konwencją operatora, a nie automatycznie egzekwowanym warunkiem.

## 1. Ustawienia globalne

W **Settings → General → Pull Requests**:

| Opcja | Ustawienie | Pole API |
|---|---|---|
| Allow merge commits | Włączone | `allow_merge_commit: true` |
| Allow squash merging | Włączone | `allow_squash_merge: true` |
| Allow rebase merging | Wyłączone | `allow_rebase_merge: false` |

Te ustawienia dotyczą całego repozytorium. Metoda dozwolona przez ruleset musi być również włączona globalnie.

## 2. Trzy rulesety

W **Settings → Rules → Rulesets** utworzyliśmy trzy branch rulesets. Wszystkie mają **Enforcement status: Active**. Dla każdego wskazujemy dokładne gałęzie w **Target branches**, bez wykluczeń.

| Nazwa | Target branches | Reguły | Bypass |
|---|---|---|---|
| Shared - PR required, no deletion or force push | `main`, `develop` | Require a pull request before merging; Allowed merge methods: Merge i Squash; Restrict deletions; Block force pushes | Brak |
| Main - merge commit only | `main` | Require a pull request before merging; Allowed merge methods: Merge | Brak |
| Develop - squash only | `develop` | Require a pull request before merging; Allowed merge methods: Squash | Repository administrators → Always allow |

Opcja **Allowed merge methods** znajduje się w dodatkowych ustawieniach **Require a pull request before merging**. To ona odpowiada za wybór metod dla konkretnej gałęzi.

Odpowiedniki w API:

- Wspólny ruleset: `conditions.ref_name.include = ["refs/heads/main", "refs/heads/develop"]`; reguły `pull_request`, `deletion`, `non_fast_forward`; `allowed_merge_methods = ["merge", "squash"]`; `bypass_actors = []`.
- Main: `conditions.ref_name.include = ["refs/heads/main"]`; reguła `pull_request` z `allowed_merge_methods = ["merge"]`; `bypass_actors = []`.
- Develop: `conditions.ref_name.include = ["refs/heads/develop"]`; reguła `pull_request` z `allowed_merge_methods = ["squash"]`; `bypass_actors = [{"actor_id": 5, "actor_type": "RepositoryRole", "bypass_mode": "always"}]`.

Pełne konfiguracje: [shared.json](../rulesets/shared.json), [main.json](../rulesets/main.json), [develop.json](../rulesets/develop.json). Są to eksporty konfiguracji; zmiana pliku nie aktualizuje automatycznie ustawień GitHub.

## 3. Dlaczego rozdzieliliśmy reguły

Rulesety obowiązujące tę samą gałąź nakładają się. Wspólny ruleset dopuszcza merge i squash, natomiast ruleset gałęzi zawęża wybór do jednej metody.

Bypass pozwala ominąć cały ruleset. Dlatego administrator ma bypass tylko w `Develop - squash only`. Osobny wspólny ruleset, bez bypass, nadal wymaga PR-a i blokuje force push oraz usuwanie `main` i `develop`.

Do samego ograniczenia metod wystarczają reguły `Allowed merge methods`. Blokady usuwania i force push są dodatkowymi zabezpieczeniami. Osobny wspólny ruleset pozwala zachować je oraz wymaganie PR-a niezależnie od wyjątku dotyczącego metody na develop.

## 4. Wyjątek main → develop i wynik testu bypass

W rzeczywistym teście [PR #4](https://github.com/ZDSDD/merge-methods-playground/pull/4):

1. Bez bypass próba merge commit do `develop` została odrzucona przez REST API z HTTP 405.
2. Po dodaniu administratora z **For pull requests only** REST nadal odrzucał merge. Próba `gh pr merge --merge --admin` przez GraphQL również została odrzucona.
3. Po zmianie bypass na **Always allow**, wyłącznie w rulesecie metody dla develop, REST API zaakceptowało merge commit.

To zaobserwowane zachowanie tego eksperymentu, nie gwarancja działania każdego wariantu bypass w GitHub.

**Administrator może ominąć ograniczenie metody dla każdego PR-a do develop, również z feature brancha.** Ta konfiguracja nie egzekwuje warunku „merge tylko ze źródła main”. Takie automatyczne rozróżnienie wymaga dodatkowej automatyzacji.

## 5. Pozostałe istotne ustawienia

- **Require linear history** jest wyłączone. Włączenie go na develop kolidowałoby z synchronizacyjnymi merge commitami; na main z założoną metodą merge.
- **Required approvals** wynosi `0` w każdym rulesecie.
- Nie wymagamy review właścicieli kodu, zatwierdzenia ostatniego push ani rozwiązania wszystkich wątków review.
- Nie ustawiliśmy wymaganych status checks ani merge queue. Repo służy do testowania metod scalania, nie pełnej polityki CI i review.

## 6. Jak sprawdzić konfigurację ręcznie

- [PR #8 → main](https://github.com/ZDSDD/merge-methods-playground/pull/8): przykład do sprawdzenia ograniczenia do merge commit, również dla administratora.
- [PR #7 feature → develop](https://github.com/ZDSDD/merge-methods-playground/pull/7): dwa commity do squasha. Administrator ma bypass. Aby zobaczyć ścisłe squash-only, tymczasowo usuń bypass z `Develop - squash only` i odśwież PR.
- [PR #6 main → develop](https://github.com/ZDSDD/merge-methods-playground/pull/6): przykład synchronizacji. Przywróć administratorowi **Always allow** w rulesecie metody, aby użyć merge commit.

PR-y pozostawiono otwarte po utworzeniu eksperymentu; ich stan może zmienić się po ręcznych testach. Pierwsze wykonane testy to [#1 — squash do develop](https://github.com/ZDSDD/merge-methods-playground/pull/1) i [#2 — merge do main](https://github.com/ZDSDD/merge-methods-playground/pull/2). Niedozwolone metody zostały odrzucone, a dozwolone zaakceptowane.

Synchronizacja przez merge zachowuje wspólną historię. Nie oznacza identycznych SHA obu gałęzi, ponieważ powstaje nowy merge commit. Przyjęta konwencja: feature → develop przez squash; develop → main oraz main → develop przez merge commit.

[Powrót do README i tabeli wyników](../README.md) · [Ustawienia rulesets](https://github.com/ZDSDD/merge-methods-playground/settings/rules)
