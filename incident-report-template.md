# Rapport d’incident et post‑mortem

## Informations générales

- **Date et heure de l’incident** : 29 juillet 2025, 14:35 CET
- **Services impactés** : API principale exposée sur localhost:5000
- **Gravité** : faible

## Résumé

Au cours d’un test de résilience, un appel volontaire à la route /error de l’API a entraîné une erreur 500, générée suite à une division par zéro.
L’incident a bien été détecté au travers des dashboards Grafana, sur lequel on peut voir l’injection brutale d’erreurs, grâce aux métriques remontées par Prometheus.
Cet incident n’a eu aucun impact sur des utilisateurs finaux car il a été effectué en milieu de test, mais a permis de valider la bonne fonction de chaîne de supervision et d’alerte.

## Timeline

| Heure (CET) | Événement 									|
|-------------|---------------------------------------------------------------------------------|
|14:35	      |Appel volontaire à la route /error entraînant une erreur 500			|
|14:36	      |Détection de la montée du taux d’erreurs dans Grafana				|
|14:38	      |Analyse des logs et identification de la cause (division par zéro)		|
|14:40	      |Rétablissement : arrêt des requêtes vers /error et redéploiement de la stack	|
|-------------|---------------------------------------------------------------------------------|

## Détection et diagnostic

Détection immédiate grâce au dashboard Grafana affichant le taux d’erreurs HTTP 5xx.


La métrique clé utilisée était http_requests_total{status=”500”}, scrappée par Prometheus.


Les logs applicatifs ont confirmé un ZeroDivisionError provenant de la route /error.

## Cause racine

Implémentation volontaire dans app.py d’une route /error contenante une division par zéro, dédiée à tester la supervision.


Facteurs contributifs :


Pas de middleware global pour intercepter et traiter proprement les exceptions.


Route de test exposée sans restriction, même dans des environnements non productifs.

## Actions correctives et rétablissement

Correctives actions and recovery
Immediate stop of requests to /error.


Redeployment of the Docker Compose stack to restart from a stable state.


Post-fix checks :


Return of the rate of HTTP 500 errors to zero in Grafana.


Logs free of new errors.


Manual check of the other routes of the API to confirm their proper functioning.

## Leçons apprises et actions de prévention

L’efficacité de la supervision (Prometheus + Grafana) a permis une détection rapide des anomalies Le respect strict dans l’environnement de production de la désactivation des routes de test permettrait d’éviter des incidents similaires à l’avenir ;
Améliorations envisagées :
Ajout d’un middleware global afin de capturer et logger correctement toutes les exceptions ;
Mise en œuvre d’une configuration environnementale à bascule pour activer/désactiver les routes de test ;
Mise en place d’alertes Prometheus plus granularisées distinguant les erreurs applicatives des erreurs de test ;
Documentation des routes particulières dans la documentation technique ;
Processus/outils :
Vérification systématique avant mise en production de l’absence de routes ou endpoints dédiés au test ;
Intensification de la revue de code pour se protéger de ce type de risque ;
