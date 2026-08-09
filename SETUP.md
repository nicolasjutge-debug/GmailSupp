# Vider Gmail — version web (iOS / Safari)

Version web du nettoyage de stockage Gmail, pensée pour Safari sur iPhone (ajoutable à l'écran d'accueil comme une app). Tout tourne dans le navigateur : pas d'installation, pas de fichier de connexion à gérer comme avec la version Python.

Deux fichiers, `index.html` et `icon.png` (icône pour l'écran d'accueil) — à héberger ensemble, dans le même dossier. Mêmes fonctionnalités que la version Python : recherche par taille, transfert vers une autre adresse, corbeille ou suppression définitive — avec sélection des emails et une jauge d'espace.

## Configuration (à faire une seule fois)

### 1. Créer un Client ID OAuth "Web application"

Cette version a besoin de son propre identifiant Google, différent de celui de la version Python (qui était de type "Desktop app"). Le même projet Google Cloud peut être réutilisé si l'étape a déjà été faite pour le script Python.

1. [console.cloud.google.com](https://console.cloud.google.com/) → ouvrir le projet (ou en créer un) ; l'API Gmail doit être activée (**APIs & Services > Library** → "Gmail API" → **Enable**).
2. L'écran de config OAuth s'appelle maintenant **Google Auth Platform**, en 4 sections (menu **APIs & Services > OAuth consent screen**, ou taper "OAuth" dans la barre de recherche en haut) :
   - **Audience** : vérifier que l'adresse Gmail concernée est dans **Test users**
   - **Data Access** : ajouter les scopes `.../auth/gmail.modify` et `.../auth/gmail.send` (ajouter aussi `https://mail.google.com/` pour la suppression définitive)
3. Onglet **Clients > Create Client** :
   - Type d'application : **Web application**
   - Dans **Authorized JavaScript origins**, ajouter l'URL exacte où le fichier sera hébergé (ex. `https://simon.jutge.free.fr` ou `https://pseudo.github.io`), sans slash final
   - Laisser **Authorized redirect URIs** vide — pas utilisé par cette app
   - Créer : une fenêtre affiche deux valeurs, **copier uniquement le Client ID** (se termine par `.apps.googleusercontent.com`), pas le Client Secret (`GOCSPX-...`, inutile ici et à ne jamais partager)
   - Pour retrouver le Client ID plus tard : onglet **Clients** → cliquer sur le nom du client → il est affiché en haut de la page

### 2. Héberger les fichiers

Le navigateur doit charger la page depuis une vraie URL https (pas juste ouvrir le fichier localement) pour que la connexion Google fonctionne — c'est une exigence de Google. Déposer `index.html` **et** `icon.png` ensemble, dans le même dossier. Options simples :
- Un hébergement existant
- [GitHub Pages](https://pages.github.com/) (gratuit) : déposer les deux fichiers dans un dépôt, activer Pages
- Netlify ou Vercel (glisser-déposer le dossier, gratuit)

L'URL choisie doit correspondre exactement à ce qui a été mis dans **Authorized JavaScript origins** à l'étape 1.

### 3. Configurer le Client ID dans l'app

Ouvrir l'URL : l'app demande de coller le Client ID obtenu à l'étape 1, une seule fois — il reste ensuite mémorisé sur cet appareil/navigateur (rien à éditer dans le fichier). Un lien "Changer le Client ID" permet de le modifier plus tard si besoin.

Cet écran affiche aussi l'**origine exacte de la page** (avec bouton copier) : c'est cette valeur précise qui doit être dans **Authorized JavaScript origins** sur Google Cloud Console. Si la page est ouverte en local (`file://`), un avertissement le signale — dans ce cas la connexion Google ne peut jamais fonctionner, quelle que soit la config (voir étape 2).

### 4. Utiliser depuis l'iPhone

Ouvrir l'URL dans **Safari**. Pour une expérience type app :
- Bouton Partager → **Sur l'écran d'accueil**
- L'icône (fournie par `icon.png`) ouvre ensuite la page en plein écran, sans barre Safari

Au premier clic sur "Se connecter avec Google", un écran d'avertissement "application non validée" apparaît (normal, l'app n'est pas publique) : **Paramètres avancés** → **Accéder à [nom] (non sécurisé)**.

### Erreur "client ID introuvable" (ou équivalent côté Google)

- Vérifier que c'est bien le **Client ID** qui a été collé (finit par `.apps.googleusercontent.com`), pas le **Client Secret** (`GOCSPX-...`) — les deux s'affichent côte à côte à la création
- Pas d'espace collé avant/après la valeur
- Le client existe toujours dans le même projet que l'API Gmail activée (onglet **Clients** de Google Auth Platform)
- L'URL d'hébergement correspond exactement (même https, sans slash final) à ce qui est dans **Authorized JavaScript origins**

### Erreur 401 "invalid_client" / "no registered origin"

L'app tourne bien, mais Google refuse la connexion : l'origine de la page n'est pas dans la liste autorisée du Client ID. Sur l'écran de Configuration de l'app, copier la valeur affichée sous "Origine de cette page" et l'ajouter (ou la corriger) dans **Authorized JavaScript origins** sur Google Cloud Console → **Clients** → cliquer sur le client → **Save**. Un changement peut prendre quelques minutes à se propager.

## Différences avec la version Python

- Pas de `credentials.json` ni de jeton à gérer : la connexion se refait à chaque session (jeton valable ~1h). Adapté à un usage ponctuel plutôt qu'à une automatisation en arrière-plan.
- Recherche limitée à 50 résultats par lancement ; relancer une recherche après traitement pour voir les suivants.
- Mêmes règles de sécurité : corbeille par défaut (espace libéré seulement après vidage, automatique à 30 jours ou manuel), suppression définitive optionnelle et irréversible (confirmation demandée avant d'agir).

## Sécurité

Le Client ID n'est pas un secret : personne ne peut l'utiliser pour accéder au compte sans passer par sa propre connexion Google. Il est stocké dans le stockage local du navigateur sur l'appareil utilisé — normal et sans risque pour un usage personnel.
