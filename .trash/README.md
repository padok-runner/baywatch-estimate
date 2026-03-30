Pour améliorer le SKILL

## flow complet

1. les sales envoient les questionnaire de qualification au client
2. le sales colent les réponses et tous les éléments dans `<client-name>/`, clonen le code etc
3. le sales lance l'agent
   1. l'agent fail, le sales renvois des questions complémentaire au client
   2. l'agent réussi; le sales fais challenger a un SA ou TL
4. le sales envois le prix

- add audit and reversibility prices
- add monitoring prices ?
- si il y a pas assez d'info, l'agent fail et demande plus d'infos ou propose du temps homme
- différencier les prix au "forfait" et au "temps passé" dans le tableau de synthèse

## Later

- lister clairement les hypothèses de design to cost
- have a clean `questions.md`, a questionaire that sales send to the client (including asking fot the "inventaire", the export of tickets of the last 6 months, ETP estiamtes, etc)
  - l'inventire complet des resrouces (on peu fournir les scripts aws, k8s etc)
  - le code IaC et l'extract des tickets de 6 derniers mois
  - les question de niveau de service, compliance etc
  - ajouter de récupérer le budget du client -> pour information seulement, ca ne doit pas influencer le sizing mais ca doit nous dire si on fait du design to cost
- instead of "MCO" use the terme "support" ?
- insted of "evolutions" use the terme "change" ?
- utiliser "consommé" plutôt que "temps passé"
- francais ou anglais ????
