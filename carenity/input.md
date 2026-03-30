## Contexte

### Inventaire

Equipe Dev/IT
2 développeurs seniors
1 product owner
1 CTO
Stack de developpement
LAMP + Opensearch + Redis
4 Multiple Page Applications legacy avec framework PHP CodeIgniter (dont une très legacy en PHP 5/MySQL 5)
2 Single Page Applications avec framework frontend VueJS et framework backend PHP Slim
2 applications "infrastructure" qui centralisent la gestion de jobqueues et la gestion des authentifications SSO de nos applications internes via notre provider Microsoft
Infrastructure AWS
Prod
5 instances EC2
2 instances RDS
1 instance Opensearch
1 instance Elasticache Redis
1 instance MySQL 5 self-hosted
Recette
3 instances EC
1 instances RDS
1 instance Opensearch
1 instance Elasticache Redis
1 instance MySQL 5 self-hosted
Autre
1 instance EC2 bastion
1 instance EC2 pour la gestion de nos packets/librairies internes
1 instance EC2 pour le déploiement de nouvelles releases
Méthodologies
Dev agile SCRUM
Déploiements de nouvelles versions de code lancés manuellement via une instance EC2 dédiée mais avec procédures automatisées
Gestion et déploiements de changements d'infrastructures automatisés via Terraform et GitHub Actions

plage horaire : 24/7
SLA : gold en prod, bronze hors prod

### Q&A 
Quand allez-vous upgrader les mysql ?
- La version 5 self-hosted ne sera pas upgradée, elle restera disponible tant que l'application legacy l'utilisant ne sera pas entièrement décommissionnée. Il n'y a pas de deadline actuellement. A noter néanmoins que dans la phase suivante à discuter ensemble de limitation du périmètre au strict besoin HDS, cette application ainsi que MySQL 5 seront enlevés
Quel profil de charge (en rps) et volume de donnés dans les bases (en go) ?
- La metric d'utilisation de base de données utilisée par AWS est le Average Active Session, sur cette metric, nous sommes entre 1 et 4 en moyenne avec parfois 1 ou 2 pics par semaine à presque 10 pendant quelques minutes.
- L'ensemble de nos bases de données de prod sur RDS font 60 Go et la plus grosse base de données parmi elles fait 27 Go. L'ensemble de nos bases de données de recette ne font pas plus de 15 Go
Combien d'incidents sur l’année dernière ET de tickets en général ?
- Incidents : 1 seul qui a mené à l'indisponibililité d'un de nos services utilisant PHP suite à une affluence plus importante qui a nécessité d'augmenter le nombre de processus PHP utilisables par le service PHP-FPM
- Tickets : Principalement des tickets d'évolution d'infrastructure (Opensearch), 1 ticket d'adaptation d'un profil AIM utilisé pour nos déploiements d'infrastructure via Terraform car nous avons changé d'organisation Github hébergeant la configuration
Nous aurons peut-être les sujets suivants d'évolution en 2026 (mais roadmap interne pas encore validée) : 
- Migration de bases de données RDS MySQL 8.0 vers 8.4 (recette et prod)
- Migration Elasticache Redis 6.2 vers 8.4 (recette et prod)
- Migration d'OS Debian 11 vers 13 (instances EC2 recette et prod)
