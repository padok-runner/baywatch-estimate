# Service Levels Reference

## Plages horaires

| Plage    | Horaires                           | Immobilisation mutualisé | Immobilisation semi-dédié | Immobilisation dédié | Prix horaire HNO |
| -------- | ---------------------------------- | ------------------------ | ------------------------- | -------------------- | ---------------- |
| Standard | Lundi-Vendredi, 09h30-18h30        | 500€                     | 0€                        | 0€                   | NA               |
| Etendue  | Lundi-Samedi, 08h-22h              | 750€                     | 1 000€                    | 2 000€               | TJM x 1.5        |
| Complète | 7j/7, 24h/24h, jours fériés inclus | 1 000€                   | 2 500€                    | 5 000€               | TJM x 3          |

## Niveaux de SLA

| Coefficient MCO        | SLA     |
| ---------------------- | ------- |
| 1                      | Bronze  |
| 1.05 par environnement | Silver  |
| 1.10 par environnement | Gold    |
| 1.20 par environnement | Platine |

## Règles par défaut

- Les environnements **hors production** sont par défaut **Bronze / Standard**, sauf demande explicite du client.
- Les environnements de **production** sont généralement **Gold**
- Toujours partir du niveau le plus faible (Bronze / Standard) et monter UNIQUEMENT si nécessaire.
- Un client ne peut avoir que **2 groupes de plages horaires** maximum.

## Guide de choix — Plage horaire

| Profil d'utilisation                                              | Plage recommandée |
| ----------------------------------------------------------------- | ----------------- |
| Application interne, utilisateurs en France métropolitaine        | Standard          |
| Application B2B avec utilisateurs dans plusieurs fuseaux horaires | Etendue           |
| Application B2C critique, e-commerce, santé, services d'urgence   | Complète          |

## Guide de choix — SLA

| Criticité                                                   | SLA recommandé |
| ----------------------------------------------------------- | -------------- |
| Environnement de dev/staging etc, application non critique  | Bronze         |
| Application interne avec impact modéré si indisponible      | Silver         |
| Application client-facing avec impact business significatif | Gold           |
| Application critique (infrastructure vitale)                | Platine        |

## Capability floor (v3)

Pour les engagements multi-tenants (`tenancy_count ≥ 5`), un plancher capacitaire en j/h/mois s'applique au-dessus de la somme marginale des buckets MCO. Le floor dépend de **plage horaire × régulation × tenancy_count** et reflète l'équipe minimum viable pour honorer la prestation.

Voir `pricing-rules.md` → "Modificateur 1 — capability_floor" pour la table complète. Les clients single-tenant (`T = 1` ou `T < 5`) ne sont pas affectés (floor = 0).
