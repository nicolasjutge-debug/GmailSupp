# Vider Gmail & Drive — version web (iOS / Safari)

Version web du nettoyage de stockage Google, pensée pour Safari sur iPhone (ajoutable à l'écran d'accueil comme une app). Tout tourne dans le navigateur : pas d'installation, pas de fichier de connexion à gérer comme avec la version Python.

Quatre fichiers à héberger ensemble, dans le même dossier : `index.html`, `icon-192.png`, `icon-512.png` et `manifest.json` (installation PC). Un sélecteur en haut de l'écran de connexion bascule entre **Gmail** et **Drive** : recherche par taille, corbeille ou suppression définitive, sélection des éléments et jauge d'espace, dans les deux cas.

**Installation** : sur PC (Chrome/Edge), un bandeau avec bouton "Installer" apparaît automatiquement quand le navigateur détecte que l'app est installable (aucune manip manuelle). Sur iPhone, Safari ne permet pas ce bouton automatique — le bandeau affiche donc l'instruction (Partager → Sur l'écran d'accueil) à la place. L'app détecte laquelle des deux capacités est disponible, elle ne devine pas via le nom du navigateur.

**Différence entre les deux modes :** un email peut être transféré sans rien demander au destinataire ; un fichier Drive, non — changer son propriétaire nécessite que l'autre compte accepte explicitement, ce n'est pas automatisable. Le mode Drive **télécharge donc le fichier sur l'appareil** avant suppression, au lieu de l'envoyer à une autre adresse.

**Google Photos n'est pas proposé.** Depuis mars 2025, une app tierce ne peut accéder qu'aux photos qu'elle a elle-même envoyées à Google Photos, pas à la bibliothèque existante de l'utilisateur — et l'API n'a jamais permis de supprimer des photos, même avant cette restriction. Il n'existe aucune combinaison de permissions qui rendrait cette fonction possible actuellement.

## Configuration (à faire une seule fois)

### 1. Créer un Client ID OAuth "Web application"

Cette version a besoin de son propre identifiant Google, différent de celui de la version Python (qui était de type "Desktop app"). Le même projet Google Cloud peut être réutilisé si l'étape a déjà été faite pour le script Python.

1. [console.cloud.google.com](https://console.cloud.google.com/) → ouvrir le projet (ou en créer un) ; activer les APIs nécessaires (**APIs & Services > Library**) : chercher et **Enable** pour "Gmail API", et de même pour "Google Drive API" si le mode Drive doit être utilisé.
2. L'écran de config OAuth s'appelle maintenant **Google Auth Platform**, en 4 sections (menu **APIs & Services > OAuth consent screen**, ou taper "OAuth" dans la barre de recherche en haut) :
   - **Audience** : vérifier que l'adresse Gmail concernée est dans **Test users**
   - **Data Access** : ajouter les scopes `.../auth/gmail.modify` et `.../auth/gmail.send` (ajouter aussi `https://mail.google.com/` pour la suppression définitive Gmail) ; pour Drive, ajouter `.../auth/drive`
3. Onglet **Clients > Create Client** :
   - Type d'application : **Web application**
   - Dans **Authorized JavaScript origins**, ajouter l'URL exacte où le fichier sera hébergé (ex. `https://simon.jutge.free.fr` ou `https://pseudo.github.io`), sans slash final
   - Laisser **Authorized redirect URIs** vide — pas utilisé par cette app
   - Créer : une fenêtre affiche deux valeurs, **copier uniquement le Client ID** (se termine par `.apps.googleusercontent.com`), pas le Client Secret (`GOCSPX-...`, inutile ici et à ne jamais partager)
   - Pour retrouver le Client ID plus tard : onglet **Clients** → cliquer sur le nom du client → il est affiché en haut de la page

Le même Client ID sert pour Gmail et Drive — pas besoin d'en créer un second.

### 2. Héberger les fichiers

Le navigateur doit charger la page depuis une vraie URL https (pas juste ouvrir le fichier localement) pour que la connexion Google fonctionne — c'est une exigence de Google. Déposer `index.html` **et** les icônes (`icon-192.png`, `icon-512.png`) ensemble, dans le même dossier. Options simples :
- Un hébergement existant
- [GitHub Pages](https://pages.github.com/) (gratuit) : déposer les fichiers dans un dépôt, activer Pages
- Netlify ou Vercel (glisser-déposer le dossier, gratuit)

L'URL choisie doit correspondre exactement à ce qui a été mis dans **Authorized JavaScript origins** à l'étape 1.

### 3. Configurer le Client ID dans l'app

Ouvrir l'URL : l'app demande de coller le Client ID obtenu à l'étape 1, une seule fois — il reste ensuite mémorisé sur cet appareil/navigateur (rien à éditer dans le fichier). Un lien "Changer le Client ID" permet de le modifier plus tard si besoin.

Cet écran affiche aussi l'**origine exacte de la page** (avec bouton copier) : c'est cette valeur précise qui doit être dans **Authorized JavaScript origins** sur Google Cloud Console. Si la page est ouverte en local (`file://`), un avertissement le signale — dans ce cas la connexion Google ne peut jamais fonctionner, quelle que soit la config (voir étape 2).

### 4. Utiliser depuis l'iPhone

Ouvrir l'URL dans **Safari**. Pour une expérience type app :
- Bouton Partager → **Sur l'écran d'accueil**
- L'icône (fournie par `icon-192.png`) ouvre ensuite la page en plein écran, sans barre Safari

Au premier clic sur "Se connecter avec Google", un écran d'avertissement "application non validée" apparaît (normal, l'app n'est pas publique) : **Paramètres avancés** → **Accéder à [nom] (non sécurisé)**.

### Erreur "client ID introuvable" (ou équivalent côté Google)

- Vérifier que c'est bien le **Client ID** qui a été collé (finit par `.apps.googleusercontent.com`), pas le **Client Secret** (`GOCSPX-...`) — les deux s'affichent côte à côte à la création
- Pas d'espace collé avant/après la valeur
- Le client existe toujours dans le même projet que l'API Gmail activée (onglet **Clients** de Google Auth Platform)
- L'URL d'hébergement correspond exactement (même https, sans slash final) à ce qui est dans **Authorized JavaScript origins**

### Erreur 401 "invalid_client" / "no registered origin"

L'app tourne bien, mais Google refuse la connexion : l'origine de la page n'est pas dans la liste autorisée du Client ID. Sur l'écran de Configuration de l'app, copier la valeur affichée sous "Origine de cette page" et l'ajouter (ou la corriger) dans **Authorized JavaScript origins** sur Google Cloud Console → **Clients** → cliquer sur le client → **Save**. Un changement peut prendre quelques minutes à se propager.

### Erreur 403 "API has not been used in project ... or it is disabled"

Différent des erreurs ci-dessus : ici la connexion Google a réussi, mais l'API elle-même (Gmail ou Drive selon le mode utilisé) n'est pas activée sur le projet Google Cloud. Le message d'erreur contient un lien direct vers la bonne page pour l'activer (**Enable**) — vérifier aussi qu'il s'agit bien du **même projet** que celui où le Client ID a été créé, en cas de doute sur le numéro de projet affiché dans l'erreur.

### Le transfert Gmail ne faisait rien, quelle que soit la taille sélectionnée (corrigé)

Deux bugs cumulés faisaient échouer systématiquement le transfert, y compris sur un tout petit lot :

- L'app envoyait le message encodé en JSON/base64 vers le point d'entrée standard de l'API Gmail, limité à 1 Mo par requête — dépassé dès qu'un email dépasse quelques centaines de Ko, donc pratiquement toujours vu le seuil de recherche par défaut (10 Mo). Elle utilise maintenant le point d'entrée dédié aux messages volumineux (envoi du message brut, sans encodage JSON/base64), qui accepte jusqu'à ~35 Mo par message.
- L'en-tête `From` du message d'origine (l'expéditeur du mail reçu) était conservé tel quel lors du renvoi, ce que l'API Gmail refuse : le `From` doit correspondre au compte connecté. Cet en-tête est maintenant retiré ; Gmail le renseigne automatiquement avec le compte connecté.

Un email individuel qui dépasse malgré tout ~35 Mo est désormais téléchargé sur l'appareil (fichier `.eml`) au lieu d'être transféré par email — Gmail ne peut de toute façon envoyer aucun message au-delà de cette taille, quelle que soit la méthode (API ou app Gmail elle-même) : c'est une limite d'émission de Gmail, pas seulement de ce point d'entrée. La suppression Gmail se fait ensuite normalement, comme pour les autres éléments. Plus largement, l'écran final indique maintenant le détail de la dernière erreur rencontrée en cas d'échec.

### Le traitement échoue en bloc sur un gros lot ("Load failed" ou erreur réseau similaire)

Cause la plus probable : l'app a été mise en arrière-plan pendant le traitement (écran verrouillé automatiquement, changement d'appli) — Safari coupe alors les requêtes réseau en cours, ce qui peut faire échouer plusieurs éléments d'affilée avec une erreur générique de ce type plutôt qu'un message Gmail explicite. L'app attend maintenant que la page redevienne visible avant de lancer l'élément suivant (message "En pause" affiché entre-temps), ce qui limite fortement le problème pour la suite du lot — mais un appel déjà en cours au moment exact de la mise en arrière-plan peut encore échouer.

Point important : dans ce cas, il est possible qu'un transfert ait en réalité réussi côté serveur Gmail sans que l'app ait reçu la confirmation à temps (la coupure réseau intervient parfois après l'envoi effectif). Par prudence, l'app ne supprime alors PAS l'original (comportement sûr, plutôt qu'un risque de perte de données) et ne retente pas non plus automatiquement l'envoi (pour éviter de créer un doublon si l'envoi précédent avait en fait abouti) — l'élément réapparaîtra simplement dans la recherche suivante. Si un doublon a malgré tout été créé côté boîte de destination, un tri manuel ponctuel peut être nécessaire. Pour l'éviter au maximum : garder l'app ouverte et l'écran allumé jusqu'à la fin du traitement, et traiter par lots de taille raisonnable (une vingtaine d'éléments plutôt que cinquante d'un coup sur une sélection très volumineuse).

### La page se ferme / revient en arrière pendant un traitement volumineux

Cause probable : mémoire saturée. Transférer un email ou télécharger un fichier Drive passe par la mémoire du navigateur (pas de flux direct vers le disque, limitation de Safari) — un lot de plusieurs centaines de Mo peut dépasser ce que Safari autorise sur mobile, et la page se recharge ou revient en arrière sans message d'erreur explicite. L'app affiche un avertissement (jauge + confirmation) au-delà de 300 Mo sélectionnés ; en cas de blocage malgré tout, décoche une partie de la sélection et traite par groupes plus petits.

### Un élément traité réapparaît dans une recherche suivante

Deux causes possibles, toutes deux inoffensives pour les données d'origine (rien n'est perdu) :
- **Échec partiel** : le transfert/téléchargement a réussi mais la suppression a échoué ensuite (API, réseau). L'app retente automatiquement une fois avant d'abandonner, et l'écran final signale distinctement ce cas ("transféré mais pas supprimé") plutôt que de le compter comme une simple erreur générique.
- **Délai d'indexation Gmail** : après une mise à la corbeille, l'index de recherche Gmail peut mettre quelques instants à se mettre à jour ; une recherche relancée immédiatement peut encore montrer l'élément brièvement.

## Différences avec la version Python

- Pas de `credentials.json` ni de jeton à gérer : la connexion se refait à chaque session (jeton valable ~1h). Adapté à un usage ponctuel plutôt qu'à une automatisation en arrière-plan.
- Recherche limitée à 50 résultats par lancement ; relancer une recherche après traitement pour voir les suivants.
- Mêmes règles de sécurité : corbeille par défaut (espace libéré seulement après vidage, automatique à 30 jours ou manuel, identique sur Gmail et Drive), suppression définitive optionnelle et irréversible (confirmation demandée avant d'agir).
- La version Python ne couvre que Gmail ; le mode Drive n'existe que dans cette version web.

## Sécurité

Le Client ID n'est pas un secret : personne ne peut l'utiliser pour accéder au compte sans passer par sa propre connexion Google. Il est stocké dans le stockage local du navigateur sur l'appareil utilisé — normal et sans risque pour un usage personnel.

Le scope Drive utilisé (`.../auth/drive`) donne un accès complet au Drive, nécessaire pour retrouver et supprimer n'importe quel fichier volumineux existant (un accès limité aux seuls fichiers créés par l'app, plus restreint, ne permettrait pas de voir les fichiers déjà présents). Comme pour la suppression définitive Gmail, Google affiche l'écran "application non validée" à la connexion — sans risque pour un usage personnel avec son propre compte en Test user.
