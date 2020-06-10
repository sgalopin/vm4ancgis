# vm4ancgis

Machine virtuelle de développement pour AncGIS.

## INSTALLATION

### Installation de la machine virtuelle (VM)

Vagrant est utilisé pour instancier la machine virtuelle.
- Installer [VirtualBox](https://www.virtualbox.org/wiki/Downloads),
- Installer [Vagrant](https://www.vagrantup.com/downloads.html),
- Installer [Git](https://git-scm.com/downloads),
- Cloner le dépôt sgalopin/ancgis:
    - Ouvrir un Git Bash:
        - Se placer dans le répertoire dans lequel vous souhaitez installer le projet,
        - Cliquer sur le bouton droit de la souris,
        - Cliquer sur 'Git Bash',
        - Taper la ligne de commande suivante:
        ```
        git clone https://github.com/sgalopin/vm4ancgis.git
        ```

**Note**: Sur certaines machines, il peut être nécessaire d'activer la virtualisation via votre BIOS. Les procédures sont propres à chaque machine et ne peuvent donc être décrites ici.

**Cas particulier (Derrière un proxy)**:

- Ajouter le plugin **vagrant-proxyconf**:

  Se placer dans le répertoire du projet et lancer la commande suivante:
  ```
  VAGRANT_DISABLE_STRICT_DEPENDENCY_ENFORCEMENT=1 vagrant plugin install vagrant-proxyconf
  ```

- Créer le fichier **~/.vagrant.d/Vagrantfile** avec le contenu suivant:
  ```
  Vagrant.configure("2") do |config|

    puts "Setting the local configuration."
    if Vagrant.has_plugin?("vagrant-proxyconf")
          config.proxy.http     = ENV['http_proxy'] || ""
          config.proxy.https    = ENV['https_proxy'] || ENV['http_proxy'] || ""
          config.proxy.no_proxy = ENV['no_proxy'] || "localhost,127.0.0.1,.dev.net"
          config.proxy.enabled  = ENV.fetch('http_proxy', false)
          if ENV.has_key?('http_proxy')
            puts "Setting of the environment proxy configuration."
            puts "http_proxy = #{config.proxy.http}"
          else
            puts "No environment proxy configuration found."
          end
      end

  end
  ```
  **Note**:
  - Le répertoire **.vagrant.d** est un répertoire caché. Il peut être nécessaire d'afficher les fichiers cachés.
  - Pour comprendre comment Vagrant utilise les Vagrantfiles, notamment concernant l'ordre de priorisation et la fusion, il peut être utile de consulter la [documentation](https://www.vagrantup.com/docs/vagrantfile/#load-order-and-merging).
  - Le caractère '~' correspond au répertoire par défaut de l'utilisateur courant ([Home directory](https://en.wikipedia.org/wiki/Home_directory)). Par exemple:
    - **Sous windows (vista, 7, 8 et 10)** créer le fichier dans: ```<root>\Users\<username>\.vagrant.d```.
    - **Sous linux (Ubuntu)** créer le fichier dans: ```/home/<username>/.vagrant.d```.


- Ajouter la variable d'environnement du proxy dans le Bash qui servira à lancer le serveur:
  ```
  http_proxy="http://<proxyhostorip>:<proxyport>"
  ```
  **Note**: Il est également possible de mettre la variable directement en dur dans le fichier de configuration si l'on travaille toujours derrière le même proxy (Évite d'avoir à remettre les variables à chaque lancement).

- Configurer les navigateurs:
  - Chrome **sous ubuntu**:

    Il peut être nécessaire selon les cas de rajouter les paramètres suivants au lancement de chrome:
    - **proxy-server** : pour indiquer l'adresse du proxyconf,
    - **proxy-bypass-list** : pour indiquer les adresses locales.
    ```
    /usr/bin/google-chrome-stable %U --proxy-server="<proxyhostorip>:<proxyport>" --proxy-bypass-list="127.0.0.1, localhost, *.dev.net"
    ```

### Ajout de la résolution de l'hôte en local
  Editer le fichier **hosts** du système d'exploitation et ajouter la ligne suivante:
  ```
  192.168.50.11 ancgis.dev.net
  ```
  **Note**:
  - **Sous windows** éditer le fichier: ```<root>\Windows\System32\drivers\etc\hosts```,
  - **Sous linux (Ubuntu)** éditer le fichier: ```/etc/hosts```.

### Paramétrer une connexion SSH vers la VM en activant le X11 Forwarding (**Optionnel**)
#### **Sous linux (Ubuntu)**
  Editer le fichier **.ssh/config** du compte courant sur votre système d'exploitation et ajouter la configuration suivante:
  ```
  Host ancgis.dev.net
    User vagrant
    Port 22
    UserKnownHostsFile /dev/null
    StrictHostKeyChecking no
    PasswordAuthentication no
    IdentityFile <installpath>/vm4ancgis/.vagrant/machines/default/virtualbox/private_key
    IdentitiesOnly yes
    LogLevel FATAL
    ForwardX11 yes
  ```
  La connexion ssh en X11 Forwarding vers la VM pourra être établie avec la commande:
  ```
  ssh ancgis.dev.net
  ```

#### **Sous windows**
 - Installer un manager de fenêtre X (X window manager),
 - Installer PuTTY,
 - Importer la clé privée de la VM dans PuTTYgen et créer une nouvelle clé privée au format PuTTY,
 - Configurer une connexion SSH vers la VM.


### Lancer la VM et le serveur (NodeJS)

Se placer dans le répertoire dans lequel vous avez installé le projet:
- Ouvrir une console ('clic droit' puis 'Git Bash Here' sous windows),
- Entrer les variables d'environnement propres au proxy si nécessaire,
- Taper la commande suivante pour lancer la VM:
```
vagrant up
```
- Une fois la VM prête, taper la commande suivante pour ouvrir une connexion ssh vers la VM:
```
vagrant ssh
```
ou (sous linux avec X11 Forwarding):
```
ssh ancgis.dev.net
```
ou avec PuTTY (sous windows avec X11 Forwarding).
- Une fois la connexion établie, taper la commande suivante pour lancer le serveur:
```
cd /var/www/ancgis/sources && npm run start:dev
```

### Modification des variables d'environnement du projet

Après la création de la VM, le projet contient deux fichiers **.env** qui permettent la définition de variables privées globales:

- /var/www/ancgis/sources/.env
- /var/www/ancdb/sources/shell/.env

**Note**: Dans un premier temps, il n'est pas nécessaire de les éditer.

### Lancer le manager de base de données (**Optionnel**)

- Ouvrir une connexion ssh vers la VM,
- Une fois la connexion établie, taper la commande suivante pour lancer le serveur:
```
cd /var/www/adminMongo && npm start
```

### Accès à l'application et à la base de données
Lancer votre navigateur et entrer les adresses suivantes:
- Pour accéder à l'application : https://ancgis.dev.net/
- Pour accéder au manager de la base de données: http://ancgis.dev.net:1234

**Note**: Testé uniquement avec Chrome pour l'instant.

### Ajouter le certificat ssl de développement
Afin de pouvoir activer les services workers (Gestion du cache navigateur pour le mode déconnecté), il est nécessaire de travailler en https et donc de disposer d'un certificat. Un certificat auto-signé à donc été créé pour les besoins du développement. Toutefois celui-ci n'est pas accepté par défaut par les navigateurs. Il faut donc ajouter le certificat en local sur la machine de développement.
Le mode opératoire dépend du navigateur et du système d'exploitation utilisés.

Voici quelques exemples de mode opératoire:
- [Chrome sur Ubuntu](https://leehblue.com/add-self-signed-ssl-google-chrome-ubuntu-16-04/)
- [Chrome sur Windows](https://www.digital-dynamics.fr/FR/tutos/ajouter-exception-site-certificat-non-fiable-google-chrome.html)

### Modifier le referer (Optionnel)

Les cartes IGN utilisées par le projet nécessitent la possession d'une clé de sécurité rattachée à un referer. Le referer de la clé (à accès limité) fournie pour le développement est **ancgis.dev.net**. Si vous souhaitez utiliser une autre adresse pour vos développements, deux choix s'offrent à vous:
- Demander une autre clé sur le site de l'[IGN](http://professionnels.ign.fr) et la modifier dans le code,
- Télécharger un plugin de modification du referer, comme par exemple [Referer Control](https://chrome.google.com/webstore/detail/referer-control/hnkcfpcejkafcihlgbojoidoihckciin).

## DÉVELOPPEMENT

### Configurer les utilisateurs Git

Dans chacun des répertoires "vm4ancgis", "ancgis" et "ancdb":
- Après les avoir modifiées, lancer les commandes suivantes:
```
git config user.name "YourUserName"
git config user.email "YourEmailAddress"
```

### Lancer Atom (IDE) en X11 Forwarding

Pour faciliter les développements l'éditeur Atom peut être installé sur la VM.
- Ouvrir une console et taper la commande suivante pour lancer l'installation des outils de développement sur la VM:
```
vagrant provision --provision-with install-devtools
```
- Ouvrir une connexion ssh avec X11 Forwarding vers la VM,
- Une fois la connexion établie, taper la commande suivante pour lancer Atom:
```
cd /var/www/ancgis/sources && atom
```

**Note**: Si votre installation a été correctement faite, l'interface graphique d'Atom doit s'ouvrir (X11 Forwarding).

### Configuration de l'outil de développement de Chrome
Afin d'éviter tout problème de maj du code, il est **indispensable** de désactiver le cache du navigateur:
- **Désactiver le cache**: Dans l'onglet "Network" cocher la case "disable cache".

- **Désactiver le service worker de cache**: Dans l'onglet "application", dans le menu "Service Workers", cocher "Update on reload" et "Bypass for network".

**Note**: ***Attention***, certaines options ne sont actives que si l'outil de développement reste ouvert...

### Commandes vagrant
- **$ vagrant halt**: Stope la VM.
- **$ vagrant destroy**: Supprime la VM.
- **$ vagrant provision --provision-with npm-install**: Lance l'installation des modules node requis par l'application.
- **$ vagrant provision --provision-with populate-db**: (Re)charge la base de données (MongoDB).
- **$ vagrant provision --provision-with launch-app**: Lance (avec node) le serveur SANS la possibilité de l'arrêter et de le redémarrer.
- **$ vagrant provision --provision-with launch-dba**: Lance (avec node) le manager de base de données SANS la possibilité de l'arrêter et de le redémarrer.
- **$ vagrant provision --provision-with install-devtools**: Lance l'installation des outils de développements sur la VM.

### Commandes serveur (via console ssh)
- **vagrant@ancgis:~$ cd /var/www/ancgis/sources && npm run start**: Lance (avec node) le serveur AVEC la possibilité de l'arrêter et de le redémarrer.
- **vagrant@ancgis:~$ cd /var/www/ancgis/sources && npm run start:dev**: Idem ci-dessus mais lancement fait avec nodemon (Redémarrage automatique du serveur après modification des sources).
- **vagrant@ancgis:~$ cd /var/www/ancgis/sources && npm test**: Lance une batterie de tests UI (puppeteer, mocha, chai).
- **vagrant@ancgis:~$ cd /var/www/ancgis/sources && npm run build**: Lance la construction des bundles javascript.
- **vagrant@ancgis:~$ cd /var/www/adminMongo && npm start**: Lance (avec node) le manager de base de données AVEC la possibilité de l'arrêter et de le redémarrer.

### PostMan
- Il faut supprimer les variables d'environnement "http_proxy" et "https_proxy" afin que Postman puisse accéder au réseau privé de la machine virtuelle.
- Il faut s'authentifier via Postman afin de récupérer le jeton de connexion de passport (le cookie de session ne suffit pas):
  - Ouvrir Postman,
  - Créer une requête POST,
  - Dans l'onglet "Authorization", sélectionner "No Auth",
  - Dans l'onglet "Body", sélectionner "x-www-form-urlencoded"
  - Ajouter un paramètre "username" et renseigner un nom d'utilisateur valide,
  - Ajouter un paramètre "password" et renseigner un mot de passe valide,
  - Soumettre la requête et vérifier la réponse.

## DÉSINSTALLATION

### Déstruction de la VM
Dans Git Bash, entrer les commandes suivantes:
  - **$ vagrant halt**: Stope la VM.
  - **$ vagrant destroy**: Supprime la VM.

### Suppression des sources
Supprimer le répertoire dans lequel vous avez installer le projet.

### Suppression des logiciels
- Ouvrir votre gestionnaire de programme (exemple sous windows: "Panneau de configuration\Programmes et fonctionnalités"),
- Supprimer les logiciels précédemment installés:
  - VirtualBox,
  - Vagrant,
  - Git.
