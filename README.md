# Merge methods playground

Publiczne repo testowe z wyłącznie sztucznymi przykładami.

## Aktywne ustawienia

- Globalnie: merge commit i squash włączone; rebase wyłączony.
- `main`: tylko merge commit, bez bypass.
- `develop`: tylko squash; administrator ma bypass `Always allow` wyłącznie w rulesecie ograniczającym metodę.
- Osobny wspólny ruleset wymaga PR, blokuje usuwanie obu gałęzi i force push. Nie ma bypass.
- Brak wymaganych approvals i CI — repo służy do testowania metod merge.
- `Require linear history` jest wyłączone, aby umożliwić synchronizacyjne merge commity.

[Ustawienia rulesets](https://github.com/ZDSDD/merge-methods-playground/settings/rules)

## Wyniki rzeczywistych testów

| PR | Próba | Wynik |
|---|---|---|
| [#1](https://github.com/ZDSDD/merge-methods-playground/pull/1) feature → develop | Merge commit bez bypass | HTTP 405, odrzucony |
| [#1](https://github.com/ZDSDD/merge-methods-playground/pull/1) feature → develop | Squash dwóch commitów | Scalony |
| [#2](https://github.com/ZDSDD/merge-methods-playground/pull/2) develop → main | Squash | HTTP 405, odrzucony |
| [#2](https://github.com/ZDSDD/merge-methods-playground/pull/2) develop → main | Merge commit | Scalony |
| [#3](https://github.com/ZDSDD/merge-methods-playground/pull/3) hotfix → main | Merge commit | Scalony |
| [#4](https://github.com/ZDSDD/merge-methods-playground/pull/4) main → develop | Merge commit bez bypass | HTTP 405, odrzucony |
| [#4](https://github.com/ZDSDD/merge-methods-playground/pull/4) main → develop | Bypass administratora `For pull requests only` | REST nadal HTTP 405; `gh pr merge --merge --admin` również odrzucony |
| [#4](https://github.com/ZDSDD/merge-methods-playground/pull/4) main → develop | Bypass administratora `Always allow` w regule metody | Merge commit scalony |

To wyniki tego eksperymentu, nie gwarancja identycznego działania wszystkich wariantów bypass.

## Ręczne testy w interfejsie

Otwórz [listę PR-ów](https://github.com/ZDSDD/merge-methods-playground/pulls).
Pozostawione przykłady obejmują feature → develop, main → develop oraz gałąź demonstracyjną → main.

- Do main: dostępny powinien być tylko merge commit.
- Do develop: zwykły współpracownik ma tylko squash. Administrator może również użyć merge dzięki bypass.
- Aby samemu zobaczyć ścisły squash-only jako administrator, usuń tymczasowo bypass z `Develop - squash only`, a następnie odśwież PR. Przywrócenie roli administratora z `Always allow` odtwarza wariant synchronizacji.

**Bypass nie jest warunkiem „źródło PR to main”.** Administrator może ominąć ograniczenie metody dla dowolnego PR do develop. Synchronizacyjny wyjątek jest zatem konwencją operatora. Automatyczne egzekwowanie zależności od gałęzi źródłowej wymaga osobnej automatyzacji.

Merge między main i develop zachowuje wspólną historię, ale tworzy nowy commit: nie oznacza identycznych SHA obu gałęzi. Zwykły feature squashujemy; stałych gałęzi przy synchronizacji nie squashujemy.

Pliki w `rulesets/` są eksportem konfiguracji z tego eksperymentu. Zmiany w tych plikach same nie aktualizują ustawień GitHub.
