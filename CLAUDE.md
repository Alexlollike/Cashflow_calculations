# CLAUDE.md — Sandsynlighedsvægtede retrospektive depoter

## Formål

Implementer fremregning af sandsynlighedsvægtede tilstandsvise retrospektive
depoter for en markedsrentepolicy i et multi-tilstands Markov-setup.

Løs Thieles fremadrettede differentialligning simultant med Kolmogorovs
forlæns ligning via Euler-diskretisering.

## Rolle og ansvar

Denne fil definerer matematiske konventioner som Claude Code skal følge strengt.
Afvigelser fra disse formler kræver eksplicit godkendelse.

Kodeansvar: arkitektur, struktur, test, tooling.
Aktuarfagligt ansvar: Alexander. Spørg ved tvivl — gæt ikke.

## Matematisk spec

### Tilstandsproces og depot

Lad $Z(t) \in \mathcal{J}$ betegne den forsikredes tilstand til tid $t$,
og lad $X(t)$ betegne depotets størrelse til tid $t$. Starttilstand $Z(0) = i$
og startdepot $X(0) = x_0$ er kendt fra policen.

Det tilstandsvise depot defineres som den betingede middelværdi:

```
X^j(t) = E[X(t) | Z(t) = j]
```

Det sandsynlighedsvægtede depot er da:

```
X_tilde(t) = Σ_j p^j(t) X^j(t) = E[X(t)]
```

hvor `p^j(t) = P(Z(t) = j)`.

### Thiele (fremadrettet):

```
dX^j/dt = [δ(t) - β_j(t) + Σ_{k≠j} μ_jk(t)(1 + β_jk(t))] X^j(t)
           - Σ_{k≠j} μ_jk(t) X^k(t)
           - γ_j(t)
           + Σ_{k≠j} μ_jk(t) γ_jk(t)
```

### Kolmogorov (forlæns):

```
dp^j/dt = Σ_{k≠j} p^k(t) μ_kj(t) - p^j(t) μ_j(t)
```

### Begyndelsesvilkår:

```
X^j(0) = x_0  for j = i  (starttilstand)
X^j(0) = 0    for j ≠ i  (ikke observeret i anden tilstand til tid 0)
p^i(0) = 1
p^j(0) = 0    for j ≠ i
```

### Sandsynlighedsvægtet depot:

```
X_tilde(t) = Σ_j p^j(t) X^j(t)
```

## Betalingsfunktioner

Fire funktioner, alle med signatur `f(t: float) -> float`:

|Symbol    |Betydning                                    |Fortegn      |
|----------|---------------------------------------------|-------------|
|`beta_j`  |Depotafhængig løbende betalingskoefficient   |positiv = ud |
|`gamma_j` |Depotsuafhængig løbende betalingsrate        |negativ = ind|
|`beta_jk` |Depotafhængig overgangsbetalingskoefficient  |positiv = ud |
|`gamma_jk`|Depotsuafhængig overgangsbetalingved overgang|positiv = ud |

## Kodestruktur



```
thiele_retro/
├── policy.py        # Aftale- og produktdataklasser
├── solver.py        # solve(model, T, h, start_state, x0) → (t, X, p, X_tilde)
├── biometrics.py    # BiometriskModel — dødelighedsintensiteter
├── market.py        # Markedsantagelser — rentekurve
└── output.py        # DataFrame til QRT-alignet CSV
```

## Solver-kontrakt

`solve(model, T, h, start_state, x0=0.0)` returnerer numpy arrays:

- `t_grid` shape `(steps+1,)`
- `X` shape `(steps+1, n_states)` — tilstandsvise depoter `X^j(t)`
- `p` shape `(steps+1, n_states)` — tilstandssandsynligheder `p^j(t)`
- `X_tilde` shape `(steps+1,)` — sandsynlighedsvægtet depot `E[X(t)]`

Euler-skridt størrelse `h` er i år. Default `h = 1/12` (månedlig).

## Hvad Claude Code IKKE må

- Ikke ændre betalingsfunktionernes fortegnskonvention
- Ikke introducere prospektive depoter eller diskonterede cashflows
- Ikke tilføje stokastiske afkast uden eksplicit instruktion
- Ikke refaktorere `Model`-dataklassen uden at køre eksisterende tests først

## Næste skridt (efter denne session)

- [ ] Porteføljeaggregering: vægt enkeltpolice-depoter med bestandsstørrelse
- [ ] R-port af solver til endelig aktuarmodel
