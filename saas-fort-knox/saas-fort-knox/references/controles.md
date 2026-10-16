# Référentiel des contrôles

Utiliser ce référentiel comme catalogue de décisions, pas comme liste de produits à installer systématiquement. Pour chaque mesure retenue, définir une preuve de fonctionnement et un responsable.

## 1. Architecture et scalabilité

- Load balancer Nginx ou Traefik en frontal ; CDN Cloudflare ou CloudFront pour les assets statiques. Empêcher la mise en cache publique de données privées et protéger l'accès direct à l'origine.
- Docker et Kubernetes pour isoler et orchestrer les services lorsque leur coût est justifié ; workloads non privilégiés, ressources bornées, probes et autoscaling horizontal.
- Propriété des données par service avec PostgreSQL ou MySQL et identifiants d'accès distincts. Éviter les écritures directes dans la base d'un autre service ; définir migrations et cohérence inter-services.
- Redis pour le cache avec authentification, accès privé, TTL et clés incluant le tenant.
- RabbitMQ ou Kafka pour les tâches asynchrones : droits minimaux, messages validés, idempotence, retries bornés, dead-letter queue et stratégie d'évolution des schémas.
- Prometheus/Grafana pour métriques et alertes ; ELK ou équivalent pour journaux centralisés ; corrélation avec traces distribuées.

Preuves : test d'isolation tenant, charge représentative, saturation des dépendances et traitement répété d'un message sans effet métier dupliqué.

## 2. Zero Trust et contrôle d'accès

- MFA obligatoire pour les comptes humains concernés par le périmètre ; profil Fort Knox : SSO OIDC/SAML et clé matérielle FIDO2/WebAuthn pour les administrateurs, sessions courtes et réauthentification des opérations sensibles. Les workloads utilisent des identités machine, pas du MFA humain.
- RBAC fin vérifié côté backend, complété par les contraintes tenant et ressource ; moindre privilège et refus par défaut.
- API Gateway : quotas par identité et tenant, limites de taille et de durée, validation JWT de signature, algorithmes autorisés, issuer, audience et expiration. La gateway ne remplace pas l'autorisation métier.
- TLS 1.3 comme cible sur les flux supportés ; identifier les exceptions et leur plan de réduction. Profil Fort Knox : Istio ou Linkerd avec mTLS inter-services et politiques d'identité explicites.
- Segmentation réseau privée, ingress/egress restreints et NetworkPolicies. En profil renforcé, évaluer VPC ou sous-réseaux dédiés par domaine sensible ; un VPC par microservice n'est pas automatiquement nécessaire.
- WAF contre les attaques pertinentes de l'OWASP Top 10, avec validation applicative et tests complémentaires.
- Trivy/Snyk ou équivalent pour les scans continus ; suivi des exceptions avec propriétaire et échéance.

Preuves : accès refusé entre tenants, token invalide rejeté, service non autorisé bloqué et secret révoqué effectivement inutilisable.

## 3. Données et conformité

- Chiffrement au repos AES-256 lorsque supporté, clés gérées par KMS/HSM et séparation des droits ; chiffrement en transit et rotation documentée.
- TDE si disponible dans le moteur ou l'offre retenue. Vérifier cette disponibilité ; le chiffrement des volumes, TDE et le chiffrement applicatif couvrent des risques différents.
- Sauvegardes automatisées, chiffrées et stratégie 3-2-1 : trois copies, deux supports ou domaines de défaillance adaptés, une copie hors site ; ajouter immutabilité ou copie hors ligne selon le risque. Séparer les identités et tester la restauration.
- Données synthétiques ou anonymisées en test. Le masking ou la pseudonymisation ne garantissent pas l'anonymat ; vérifier le risque de réidentification.
- Audit des actions sensibles avec intégrité vérifiable, stockage append-only/WORM adapté, droits séparés, horodatage fiable et rétention définie. Exclure secrets et données inutiles des logs.
- Privacy by Design : minimisation, durées de conservation, droits des personnes, localisation, sous-traitants et analyse d'impact lorsque requise. Construire les preuves pour ISO 27001 ; ne pas déclarer une certification sur la seule base de l'architecture.
- Profil Fort Knox : chiffrement homomorphique uniquement pour des traitements précis après étude de faisabilité, latence et coût. Il ne permet pas de chiffrer arbitrairement tout traitement applicatif sans adaptation.

Preuves : restauration dans le RTO/RPO prévu, accès aux clés contrôlé, tentative d'altération d'audit détectée et données de test examinées.

## 4. Résilience et disponibilité

- Définir un SLO de 99,99 % avec indicateur, fenêtre, périmètre et exclusions explicites. Examiner dépendances et modes de panne communs.
- Multi-zones puis multi-régions selon le besoin ; multi-cloud seulement si le bénéfice justifie complexité et coûts.
- Réplication et failover des bases avec fencing, prévention du split-brain, cohérence et perte de données admissible documentées.
- Health checks distincts de startup, readiness et liveness ; autoscaling avec limites et capacité des dépendances.
- PRA automatisable, procédures de retour à la normale et exercices de bascule/restauration.
- Chaos engineering régulier dans un périmètre maîtrisé, avec hypothèse, limite d'impact et arrêt immédiat.

Preuves : perte de zone/région simulée, bascule sans double écriture, restauration mesurée et alertes sur consommation du budget d'erreur.

## 5. Code et livraison

- SAST SonarQube/Checkmarx ou équivalent ; SCA et scans des conteneurs/secrets/IaC. Prioriser selon exploitabilité et exposition, avec exceptions traçables.
- DAST et tests d'intrusion automatisables dans un environnement autorisé ; compléter par revue humaine des risques métier et d'autorisation.
- Terraform et politiques OPA ou équivalent : blocage des expositions publiques injustifiées, droits excessifs et ressources non chiffrées.
- Identités CI éphémères, branches et environnements protégés ; signature des artefacts, attestations de provenance et vérification avant déploiement.
- Blue/green ou canary selon l'application ; promotion fondée sur métriques, erreurs, latence et signaux de sécurité pertinents. Définir seuils, fenêtre et rollback.
- Migrations compatibles avec ancienne et nouvelle versions ; un rollback applicatif ne garantit pas un retour arrière des données.

Preuves : artefact altéré refusé, politique IaC bloquante testée, canary dégradé arrêté et rollback compatible avec le schéma.

## 6. Détection et incidents

- SOC minimal : collecte utile, règles de détection, triage, astreinte et responsabilités ; IDS/IPS et UEBA selon l'environnement.
- Alertes Slack/PagerDuty ou équivalent avec niveaux, déduplication et escalade. Mesurer faux positifs et délai de détection ; l'analyse ML est optionnelle et doit être évaluée.
- Traces Jaeger ou équivalent avec identifiants de corrélation, échantillonnage et contrôle des données sensibles ; ne pas promettre le traçage exhaustif sans dimensionnement.
- Playbooks fuite de données, DDoS, compte compromis et service compromis : qualification, confinement, révocation, préservation des preuves, récupération et communication.
- Profil Fort Knox : isolation automatisée bornée ; capture de logs et mémoire lorsque techniquement et juridiquement adaptée, avec accès restreint et chaîne de conservation.
- Post-mortem sans recherche de faute individuelle, causes, responsables et suivi. Sous 24 h : triage, confinement et plan correctif selon gravité ; ne pas promettre la correction de toutes les causes dans ce délai.
- Formation phishing et simulations proportionnées ; accès production en lecture seule par défaut, élévation temporaire justifiée et accès d'urgence audité.

Preuves : exercice de compromission, alerte routée au bon responsable, révocation efficace et récupération chronométrée.

## 7. Déception défensive optionnelle

- Honeypots, fausses APIs et canary tokens isolés sans données ni secrets réels, sans route exploitable vers la production.
- Fausses routes et données uniquement dans un périmètre contrôlé ; ne pas tromper les utilisateurs légitimes ni contaminer les données métier.
- Blocage automatique d'IP avec seuils, expiration, déblocage et analyse des NAT/proxies ; blocage d'ASN réservé à des situations justifiées, son impact étant plus large.
- Programme interne de signalement ou bug bounty : périmètre explicite, règles d'engagement, canal privé, triage et récompenses définies.
- Défense limitée aux systèmes autorisés ; aucun contre-piratage des infrastructures de tiers.

Preuves : leurre inaccessible aux flux métier, déclenchement détecté, blocage expirant et faux positif réversible.
