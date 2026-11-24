# Installer OpenCode 2 sur Termux (Google Play) — Android ARM64

Ce guide reprend la méthode qui a fonctionné lors de nos essais sur un Samsung Galaxy Z Fold 8 sous Android 17, avec **Termux Google Play `googleplay.2026.10.09`** et une architecture **ARM64 (`aarch64`)**.

La méthode consiste à installer directement le paquet DEB natif publié par Hope2333. Elle évite les dépendances Bun et Ripgrep qui n'étaient pas disponibles dans le dépôt Termux Google Play rencontré pendant nos essais.

> **Installation vérifiée :** OpenCode `2.0.31` s'est installé et `opencode --version` a renvoyé `opencode v2.0.31`. Le serveur Web a ensuite démarré sur `127.0.0.1:4096`.

## 1. Prérequis et vérification

- Android sur appareil ARM64.
- Termux installé depuis Google Play.
- Connexion Internet.

Dans Termux :

```bash
termux-info
```

Vérifier notamment `TERMUX_VERSION=googleplay...` et `Packages CPU architecture: aarch64`.

Mettre à jour les paquets et installer les outils de téléchargement :

```bash
pkg update && pkg upgrade -y
pkg install curl ca-certificates
```

## 2. Télécharger OpenCode 2.0.31

La version utilisée pendant nos essais est publiée dans la release `Push261105` du dépôt Hope2333.

```bash
cd ~
curl -fL https://github.com/Hope2333/opencode-termux/releases/download/Push261105/opencode_2.0.31_aarch64.deb -o opencode.deb
```

Vérifier la présence et la taille du fichier :

```bash
ls -lh ~/opencode.deb
```

Lors de notre essai, le fichier faisait environ 72 Mo.

## 3. Vérifier l'intégrité SHA256

```bash
sha256sum ~/opencode.deb
```

Empreinte attendue pour le fichier testé :

```text
9dd31aa2765968bc5240e86befe46348239a2f2363980d1dd0d401031adf9df2
```

**Ne pas installer le fichier si l'empreinte ne correspond pas exactement.**

Releases : https://github.com/Hope2333/opencode-termux/releases

## 4. Installer et vérifier OpenCode

```bash
dpkg -i ~/opencode.deb
opencode --version
```

Résultat obtenu pendant nos essais :

```text
opencode v2.0.31
```

L'installateur a indiqué que la version OpenCode v2 (mainline) coexiste avec la commande `opencode1` de génération 1.

## 5. Autoriser l'accès aux fichiers Android

```bash
termux-setup-storage
```

Accepter l'autorisation Android, puis créer un dossier de projets :

```bash
mkdir -p ~/projects
```

Les raccourcis vers le stockage partagé sont ensuite accessibles dans `~/storage/`. Pour les projets de développement, `~/projects` est un emplacement simple dans l'espace privé de Termux.

## 6. Démarrer le serveur Web

La commande qui a fonctionné avec OpenCode 2.0.31 :

```bash
opencode serve --hostname 127.0.0.1 --port 4096
```

Le serveur a affiché une adresse semblable à :

```text
server listening on http://127.0.0.1:4096
```

Laisser Termux ouvert pendant l'utilisation. Dans Chrome sur le même téléphone, ouvrir :

**http://127.0.0.1:4096**

Le serveur est lié à `127.0.0.1` : il est destiné à être utilisé depuis le téléphone lui-même, pas exposé au réseau local.

Pour lancer le serveur depuis un projet précis, se placer dans son dossier avant de démarrer le serveur :

```bash
cd ~/projects/mon-projet
opencode serve --hostname 127.0.0.1 --port 4096
```

Le dossier de travail dépend du répertoire depuis lequel le serveur est lancé.

## 7. Connexion depuis OpenCode Mobile

Pour définir explicitement les identifiants du serveur, les exporter avant son démarrage :

```bash
export OPENCODE_SERVER_USERNAME=opencode
export OPENCODE_SERVER_PASSWORD='REMPLACER_PAR_UN_MOT_DE_PASSE_LONG'
opencode serve --hostname 127.0.0.1 --port 4096
```

Remplacer le texte d'exemple par un mot de passe personnel long. Ne pas le publier ni le partager.

Dans OpenCode Mobile, ajouter une connexion manuelle avec :

- **URL :** `http://127.0.0.1:4096`
- **Nom d'utilisateur :** `opencode`
- **Mot de passe :** celui défini dans `OPENCODE_SERVER_PASSWORD`

L'application mobile et Termux étant sur le même appareil, `127.0.0.1` désigne le téléphone lui-même.

## 8. Dépannage issu de nos essais

### `Unable to locate package bun` ou `ripgrep`

Dans l'environnement Termux Google Play testé, `pkg search bun` et `pkg search ripgrep` ne renvoyaient pas de paquets installables. Le paquet DEB natif OpenCode 2.0.31 ci-dessus a permis d'éviter l'installation séparée de ces dépendances.

### Le dépôt Hope2333 renvoie `404` sur `Packages`

Une tentative de configuration APT classique vers une URL de release a échoué parce que le chemin `Packages` renvoyait `404`. Le téléchargement direct du DEB depuis l'URL de release a fonctionné.

### `hope2333-mirrorlist` ne se décompacte pas

Une première méthode a téléchargé un fichier de 934 octets qui n'était pas un paquet DEB valide (`control.tar` absent). Ne pas réutiliser cette méthode pour l'installation décrite ici.

### `opencode web` ou les options `--hostname` / `--port` à la racine sont refusées

Sur la version 2.0.31 testée, la commande à utiliser est :

```bash
opencode serve --hostname 127.0.0.1 --port 4096
```

Pour vérifier les options disponibles :

```bash
opencode serve --help
```

## 9. Arrêter le serveur

Dans la session Termux où le serveur tourne, utiliser `CTRL+C` (la touche `CTRL` est accessible depuis la rangée de touches supplémentaires de Termux).

---

## Références

- Dépôt du port Android : https://github.com/Hope2333/opencode-termux
- Releases : https://github.com/Hope2333/opencode-termux/releases
- Documentation Termux : https://termux.dev/

## Notes de compatibilité

Ce guide consigne une installation effectivement réalisée avec Termux Google Play `2026.10.09`, Android 17 et ARM64, en utilisant le paquet OpenCode `2.0.31` de la release `Push261105`.

Les versions, noms d'assets et empreintes peuvent changer. Vérifier la release et l'empreinte du paquet avant de reprendre ces commandes sur une autre version.
