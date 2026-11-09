HEALTHY LIFE 2.1 — version préparée pour hébergement
====================================================

ARCHITECTURE
- Front-end : HTML/CSS/JavaScript dans /public.
- API : Node.js 20+ / Express.
- Base : SQLite (better-sqlite3).
- Authentification : bcryptjs + JWT.
- Le même service Node sert le site et l'API : le front-end appelle /api sur le même domaine.
- Les données locales du navigateur sont conservées dans localStorage et peuvent être synchronisées après connexion.
- Langues : français, anglais, allemand, japonais, chinois, coréen et arabe.

DÉMARRAGE LOCAL
1. Installer Node.js 20 ou supérieur (Node 22 recommandé).
2. Dans le dossier du projet : npm install
3. Copier .env.example vers .env.
4. Générer un secret : node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
5. Remplacer JWT_SECRET dans .env par le secret généré.
6. Exécuter : npm start
7. Ouvrir http://localhost:3000 dans le navigateur.

Le front-end est servi directement par Express. Pour travailler avec Live Server (port 5500),
le script détecte l'aperçu local et appelle http://localhost:3000/api. L'origine doit rester
présente dans ALLOWED_ORIGINS.

DÉPLOIEMENT RENDER (un seul service)
1. Créer un dépôt Git privé et y pousser le contenu du projet.
2. Vérifier que .env, node_modules, data/ et les fichiers SQLite ne sont pas suivis par Git.
3. Dans Render, choisir New > Blueprint et connecter le dépôt contenant render.yaml.
4. Examiner le Blueprint : il crée le service web healthy-life et un disque persistant de 1 Go.
5. Render génère JWT_SECRET automatiquement. Ne pas le remplacer par un secret public.
6. Après création, vérifier l'URL publique attribuée par Render.
7. Dans Environment, remplacer ALLOWED_ORIGINS par l'URL exacte du service (https://...onrender.com).
8. Vérifier DB_PATH=/var/data/healthy-life.sqlite et le montage du disque sur /var/data.
9. Déployer puis tester https://VOTRE-DOMAINE/api/health (réponse attendue : {"status":"ok"}).
10. Tester l'inscription, la connexion, la synchronisation, puis redémarrer le service et vérifier la persistance.

La configuration render.yaml utilise le plan Starter et un disque persistant : ce n'est pas un
hébergement entièrement gratuit. Vérifier le tarif courant dans le tableau de bord Render avant
validation du Blueprint. Aucun compte ni service n'a été créé par ce projet.

VARIABLES D'ENVIRONNEMENT
- NODE_ENV=production
- PORT : fourni automatiquement par l'hébergeur.
- JWT_SECRET : secret aléatoire long, à conserver uniquement dans le gestionnaire de secrets.
- DB_PATH=/var/data/healthy-life.sqlite (Render avec le disque défini dans render.yaml).
- ALLOWED_ORIGINS=https://VOTRE-DOMAINE (plusieurs origines séparées par des virgules si nécessaire).
- JWT_EXPIRES_IN=7d (optionnel).

SAUVEGARDE
- Une commande de sauvegarde SQLite est disponible : npm run backup.
- Elle écrit une copie cohérente dans ./backups (ou BACKUP_DIR si défini).
- Cette commande n'installe pas de sauvegarde cloud automatique. En production, prévoir une
  tâche planifiée et une copie chiffrée hors du serveur, puis tester périodiquement la restauration.
- Ne pas committer les sauvegardes ni la base SQLite.

VÉRIFICATIONS AVANT OUVERTURE
- GET /api/health répond avec status=ok.
- Créer un compte avec une adresse de test et un mot de passe unique de 8 à 128 caractères.
- Vérifier qu'une seconde inscription avec le même e-mail est refusée.
- Se déconnecter puis se reconnecter : les données distantes doivent être rechargées.
- Modifier une habitude, attendre la synchronisation, recharger et vérifier la persistance.
- Vérifier le comportement après redémarrage/redéploiement.
- Parcourir les sept langues et vérifier l'arabe en RTL ainsi que les caractères asiatiques.
- Tester sur téléphone et ordinateur, et vérifier la console du navigateur.
- Vérifier qu'une origine non autorisée ne peut pas appeler l'API depuis un navigateur.

SÉCURITÉ ET LIMITES
- Cette application reste un prototype de suivi et d'information générale, pas un dispositif médical.
- Les mots de passe sont hachés avec bcryptjs; les routes d'authentification sont limitées en débit;
  les en-têtes HTTP sont renforcés avec Helmet; SQLite utilise un chemin configurable.
- Le JWT est actuellement stocké dans localStorage : une faille XSS pourrait exposer le jeton.
  Avant une utilisation plus large, envisager des cookies HttpOnly/Secure/SameSite et une protection CSRF.
- Pas de vérification d'e-mail, récupération de mot de passe, suppression de compte self-service,
  gestion multi-rôles réelle, audit complet ni sauvegarde externe automatique.
- Le rate limiting est en mémoire et doit être adapté si plusieurs instances sont utilisées.
- Ne pas utiliser pour stocker des dossiers médicaux, diagnostics, résultats d'analyses ou données
  de santé sensibles. Une revue de sécurité, une analyse juridique/confidentialité, un chiffrement
  des sauvegardes et des contrôles d'accès adaptés sont nécessaires avant ce type d'usage.
- L'assistant est local et prédéfini, pas une IA générative et ne pose aucun diagnostic.
- Les rappels ne fonctionnent que lorsque la page est ouverte; notifications en arrière-plan
  nécessitent une PWA/service worker et une configuration dédiée.
- L'espace professionnel reste une maquette sans partage ni consultation de dossiers d'autres personnes.

FICHIERS IMPORTANTS
- server.js : API, authentification, sécurité, SQLite et service du front-end.
- public/ : site web publié.
- scripts/backup.js : sauvegarde SQLite locale.
- render.yaml : configuration de déploiement Render.
- .env.example : modèle de configuration locale sans secret réel.
