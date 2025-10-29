# Guide d’installation — Recettes Symfony Flex et bundle Api Keycloak Authenticator

Ce document explique, étape par étape, comment configurer votre projet Symfony pour consommer ce dépôt de recettes Flex, puis installer le bundle `inter-invest/api-keycloak-authenticator-bundle` et finaliser sa configuration.


## 1) Prérequis
- Un projet Symfony existant (5.x/6.x/7.x) avec Composer.
- Accès réseau au dépôt de recettes (public GitHub/GitLab ou hébergement interne) — l’URL de `index.json` doit être accessible en HTTP(S).
- Le package `inter-invest/api-keycloak-authenticator-bundle` disponible (Packagist privé/public) exposant la classe `InterInvest\ApiKeycloakAuthenticatorBundle\ApiKeycloakAuthenticatorBundle`.


## 2) Ajouter ce dépôt de recettes à Symfony Flex
Dans votre projet Symfony qui consommera la recette, ajoutez l’endpoint Flex personnalisé en conservant le registre par défaut. Remplacez `<ORG>` et l’URL par celle de votre dépôt si besoin.

```bash
composer config --json extra.symfony.endpoint "[\n  \"https://raw.githubusercontent.com/<ORG>/recipes/main/index.json\",\n  \"flex://defaults\"\n]"
```

Vérification rapide:
```bash
composer config extra.symfony.endpoint
```
Vous devez voir à la fois votre endpoint personnalisé et `flex://defaults`.


## 3) Installer le bundle (déclenchement de la recette)
Dans le projet Symfony consommateur, lancez:
```bash
composer req inter-invest/api-keycloak-authenticator-bundle:^2.0
```

Ce paquet installe le bundle et déclenche la recette `2.0.5` référencée dans ce dépôt. La recette:
- enregistre le bundle pour tous les environnements,
- copie le fichier de configuration suivant dans votre projet:
  - `config/packages/api_keycloak_authenticator.yaml`
- affiche un message post-install avec les prochaines étapes.


## 4) Configurer les variables d’environnement
Ajoutez les variables suivantes dans votre `.env.local` (ou votre gestionnaire de secrets/CI):

```
API_KEYCLOAK_URL=https://keycloak.example.com
KEYCLOAK_REALM_INTERINVEST_APPLICATION_CLIENT_ID=XXXXXXXX
KEYCLOAK_REALM_INTERINVEST_APPLICATION_CLIENT_SECRET=XXXXXXXX
KEYCLOAK_REALM_INTERINVEST_APPLICATION_GRANT_TYPE=client_credentials
```

Notes:
- `API_KEYCLOAK_URL` doit pointer vers votre serveur Keycloak.
- `*_CLIENT_SECRET` doit rester secret (utilisez des variables de CI/CD ou `vault` en prod).
- La recette désactive par défaut la vérification SSL en `dev` via `when@dev` (dans le fichier de conf ci-dessous). En prod, la vérification reste activée.


## 5) Vérifier/ajuster la configuration copiée
La recette a copié `config/packages/api_keycloak_authenticator.yaml` dans votre projet. Exemple de contenu installé:

```yaml
api_keycloak_authenticator:
  baseUrl: '%env(resolve:API_KEYCLOAK_URL)%'
  verifyPeer: true
  verifyHost: true
  claim: 'sub'
  realms:
    interinvest_applications_internes:
      client_id: '%env(resolve:KEYCLOAK_REALM_INTERINVEST_APPLICATION_CLIENT_ID)%'
      client_secret: '%env(resolve:KEYCLOAK_REALM_INTERINVEST_APPLICATION_CLIENT_SECRET)%'
      grant_type: '%env(resolve:KEYCLOAK_REALM_INTERINVEST_APPLICATION_GRANT_TYPE)%'

when@dev:
  api_keycloak_authenticator:
    verifyPeer: false
    verifyHost: false
```

Adaptez si nécessaire:
- `claim`: le nom de la réclamation JWT utilisée pour identifier l’utilisateur (ex: `sub`).
- `realms`: ajoutez/renommez vos realms et identifiants clients.

Important: Ce dépôt contient aussi un exemple `config/packages/api_keycloak.yaml` qui n’est PAS copié par la recette. Vous pouvez le supprimer du dépôt si vous maintenez ce repo, afin d’éviter toute confusion.


## 6) Tests rapides de bon fonctionnement
- Démarrez votre application et vérifiez qu’aucune erreur de configuration n’apparaît.
- Si le bundle expose des commandes/points d’intégration (middleware d’auth, listeners…), testez un flux d’authentification contre Keycloak.
- En `dev`, la vérification SSL est désactivée; en `prod`, veillez à fournir des certificats valides et laissez `verifyPeer/verifyHost` à `true`.


## 7) Dépannage
- Erreur « environment variable not found »: vérifiez `.env.local`, vos variables de CI/CD et les noms exacts des clés:
  - `API_KEYCLOAK_URL`
  - `KEYCLOAK_REALM_INTERINVEST_APPLICATION_CLIENT_ID`
  - `KEYCLOAK_REALM_INTERINVEST_APPLICATION_CLIENT_SECRET`
  - `KEYCLOAK_REALM_INTERINVEST_APPLICATION_GRANT_TYPE`
- Conflits de recettes: assurez-vous que votre endpoint Flex personnalisé figure AVANT `flex://defaults` (comme dans la commande ci-dessus) si vous souhaitez que vos recettes privées priment.
- SSL en dev/prod: si vous voyez des erreurs de certificat en dev, confirmez que la section `when@dev` est bien présente dans `api_keycloak_authenticator.yaml`. N’activez pas ces assouplissements en prod.


## 8) Mise à jour du bundle/recette
- Pour mettre à jour le bundle: `composer up inter-invest/api-keycloak-authenticator-bundle`.
- Si une nouvelle version de recette est publiée dans ce dépôt, mettez à jour l’endpoint (si l’URL change) puis relancez `composer up` afin que Flex considère la nouvelle recette en fonction de la version du bundle.


## 9) Désinstallation
Pour retirer le bundle et éventuellement supprimer la configuration:
```bash
composer rem inter-invest/api-keycloak-authenticator-bundle
```
Ensuite, supprimez manuellement `config/packages/api_keycloak_authenticator.yaml` si vous n’en avez plus l’usage.


## 10) Références
- Recette référencée dans `index.json` → version `2.0.5` (`inter-invest/api-keycloak-authenticator-bundle/2.0.5/manifest.json`).
- Documentation du bundle: https://github.com/inter-invest/api-keycloak-authenticator-bundle
