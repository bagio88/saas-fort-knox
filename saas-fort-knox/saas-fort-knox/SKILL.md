---
name: saas-fort-knox
description: Concevoir, renforcer ou auditer une architecture SaaS microservices sécurisée et résiliente, avec Zero Trust, protection des données, DevSecOps et réponse aux incidents. Utiliser pour une demande d'architecture SaaS haute sécurité ou de profil Fort Knox numérique.
---

# SaaS Fort Knox

Transformer les besoins métier en architecture vérifiable, plan d'implémentation ou audit. Répondre en français sauf préférence contraire. Lire [le référentiel de contrôles](references/controles.md) pour sélectionner les mesures pertinentes.

## Cadrage et choix du profil

Identifier le livrable demandé : architecture, implémentation, audit ou playbook. Examiner le système existant avant de proposer des changements. Recueillir ou expliciter les hypothèses sur les tenants, données sensibles, régions, fournisseur cloud, trafic, équipe, budget, contraintes réglementaires et objectifs SLO/RTO/RPO. Ne bloquer que sur les informations indispensables ; avancer avec des hypothèses identifiées pour le reste.

Choisir un profil et justifier les écarts :
- **Socle sécurisé** : les six domaines du référentiel, dimensionnés pour le produit.
- **Fort Knox** : ajouter isolation forte, identités de workloads, mTLS, secrets dynamiques, accès administrateur résistants au phishing, chaîne de livraison vérifiable et capacité d'investigation.
- **Déception défensive** : ajouter leurres isolés et réponses automatiques bornées lorsque cela répond au modèle de menace.

Si les microservices sont exigés, respecter ce choix et expliciter leurs coûts opérationnels. Sinon, comparer brièvement leur intérêt à celui d'un monolithe modulaire. Ne pas imposer toutes les technologies proposées : sélectionner une option par besoin et expliquer les arbitrages.

## Méthode

1. Définir actifs, frontières de confiance et scénarios de menace : accès inter-tenant, compromission de compte ou service, fuite de secrets, dépendance compromise, DDoS, perte de région et suppression de sauvegardes.
2. Dessiner les flux utilisateur, administration, inter-services, données et livraison. Montrer les points d'authentification, d'autorisation, de chiffrement et de journalisation.
3. Sélectionner les contrôles du référentiel. Pour chacun, préciser le risque couvert, le composant responsable, les dépendances et une preuve de validation.
4. Établir les objectifs de disponibilité et de reprise. Calculer le budget d'erreur pour la période choisie ; une cible de 99,99 % ne constitue pas une garantie.
5. Produire une feuille de route priorisée : prérequis, socle, renforcement, exercices de reprise et amélioration continue. Relier chaque tâche à un risque et à un critère d'acceptation.
6. Pour une implémentation, réaliser les changements autorisés et vérifier les flux affectés. Pour un audit, distinguer contrôles observés, absents et non vérifiés ; ne jamais présenter une recommandation comme déjà déployée.

## Invariants

- Vérifier l'autorisation et le périmètre tenant côté backend à chaque opération sensible, y compris tâches asynchrones, caches, exports et stockage objet.
- Ne pas placer de secrets dans le code, les images, les journaux ou les exemples. Préférer identités de workloads et secrets éphémères délivrés au runtime ; pour le profil Fort Knox, éviter les secrets en variables d'environnement et utiliser un montage temporaire ou un agent de secrets.
- Vérifier dans les sources officielles les capacités, versions, configurations et exigences légales avant de les utiliser dans une implémentation. Donner les références utiles dans le livrable ; ne pas inventer une certification ou une compatibilité.
- Ne pas assimiler WAF, chiffrement ou conformité documentaire à une sécurité démontrée. Tester les contrôles et leurs modes de défaillance.
- Ne pas déclencher un test de charge, de chaos, d'intrusion ou une action de confinement en production sans périmètre et autorisation adaptés à cette action.
- Toute réponse automatique doit avoir un seuil, une durée, une possibilité de retour arrière et des garde-fous contre les faux positifs. Préserver les preuves et éviter une panne provoquée par la défense.

## Livrable attendu

Adapter la longueur à la demande et inclure les éléments utiles :
- Hypothèses, profil retenu et décisions avec alternatives.
- Schéma Mermaid des composants et frontières de confiance.
- Matrice « risque / contrôle / composant / preuve / priorité ».
- Stratégie données et secrets ; modèle d'accès utilisateurs, administrateurs et workloads.
- SLO, RTO/RPO, sauvegardes, failover et scénario de restauration testé.
- Pipeline CI/CD et règles de promotion ou rollback.
- Playbooks incidents et feuille de route avec critères d'acceptation.

Pour un audit, ajouter sévérité, preuve disponible et correction pour chaque constat. Pour du code ou de l'IaC, fournir les fichiers utilisables, les vérifications réalisées et les limites restantes. Ne pas générer une infrastructure complète lorsque l'utilisateur demande seulement une conception.
