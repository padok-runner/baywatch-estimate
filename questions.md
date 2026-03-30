# Questions Client — Qualification Infogérance Cloud

## 1. Contexte et conformité

1. Quelle est l'activité de votre entreprise ?
2. Avez-vous des exigences de conformité ou certifications (HDS, SecNumCloud, ISO 27001, RGPD...) ?
3. Depuis combien de temps votre infrastructure est-elle en production ?

## 2. Équipe

4. Combien de personnes composent votre équipe dev/IT (développeurs, PO, CTO, ops...) ?
5. Comment déployez-vous votre code et votre infrastructure aujourd'hui (CI/CD, manuel, outils utilisés) ?

## 3. Inventaire des ressources

6. Pouvez-vous lister toutes vos ressources d'infrastructure par environnement (prod, recette, dev, shared...) avec pour chacune : le type (VM, cluster K8s, BDD, app...), le cloud provider, et la taille (CPU, RAM, volume de données) ?
7. Combien d'applications hébergez-vous et de quel type sont-elles (legacy, custom, SPA, API...) ?
8. Avez-vous des ressources d'infra partagée (bastion, CI/CD, monitoring, gestion de paquets...) ?

## 4. Niveaux de service

9. Sur quelles plages horaires vos utilisateurs accèdent-ils à vos services (heures de bureau, étendu, 24/7) ?
10. Quel est le niveau de criticité de votre application en production (impact business si indisponible) ?
11. Avez-vous des besoins différents entre vos environnements de production et de non-production ?

## 5. Données empiriques _(si infra en prod depuis +6 mois)_

12. Combien de tickets de support/ops avez-vous traités sur les 6 à 12 derniers mois (incidents, demandes, changements, problèmes) ?
13. Quel est le temps moyen de résolution d'un incident ?
14. Y a-t-il des sujets récurrents ou particulièrement chronophages ?
15. Combien de personnes maintiennent la plateforme aujourd'hui, et comment se répartit leur temps entre le run (incidents, monitoring, patching), le build (évolutions) et la gouvernance (réunions, audits) ?
16. Utilisez-vous un outil d'IaC (Terraform, Pulumi, CloudFormation...) et pouvons-nous y avoir accès ?

## 6. Évolutions prévues

17. Quelles évolutions d'infrastructure sont prévues dans les 6 prochains mois (migrations, changements d'architecture, nouveaux services...) ?
18. Cette roadmap est-elle validée ou encore prévisionnelle ?

## 7. Contraintes complémentaires

19. Avez-vous des contraintes de localisation géographique pour vos données ?
20. Avez-vous des engagements de SLA envers vos propres clients ?
21. Quelle durée d'engagement envisagez-vous (1 an, 2 ans, 3+) ?
