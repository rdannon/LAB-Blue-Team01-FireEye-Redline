# 🚨 Réponse à incident avec FireEye Redline : collecte de données et de mémoire sur un poste Windows compromis

![Domaine](https://img.shields.io/badge/Domaine-R%C3%A9ponse%20%C3%A0%20incident%20%2F%20DFIR-blue)
![Outil](https://img.shields.io/badge/Outil-FireEye%20Redline%202-red)
![Cible](https://img.shields.io/badge/Cible-Windows%2010-0078D6)
![Collecte](https://img.shields.io/badge/Collecte-M%C3%A9moire%20%2B%20Disque%20%2B%20Registre-lightgrey)
![Analyse](https://img.shields.io/badge/Analyse-Session%20.mans-orange)

> Collecte **rapide et complète** d'informations sur un poste **Windows compromis** avec **FireEye Redline** : création de collecteurs portables (standard, complet), acquisition de la **mémoire**, collecte des **processus, registre, journaux d'événements, comptes utilisateurs, historique web**, puis **ouverture et exploration de la session d'analyse**.

<!-- 📸 IMAGE : capture d'ensemble de l'interface Redline (écran d'accueil « Collect Data / Analyze Data ») -->
![Vue d'ensemble de Redline](images/00-vue-ensemble.png)

---

## 📑 Sommaire

1. [Présentation du projet](#1--présentation-du-projet)
2. [Objectifs](#2--objectifs)
3. [Notions théoriques](#3--notions-théoriques)
4. [Environnement et installation](#4--environnement-et-installation)
5. [Types de collecteurs](#5--types-de-collecteurs)
6. [Création du collecteur standard](#6--création-du-collecteur-standard)
7. [Exécution de la collecte sur le système compromis](#7--exécution-de-la-collecte-sur-le-système-compromis)
8. [Collecteurs complet et par défaut](#8--collecteurs-complet-et-par-défaut)
9. [Analyse de la session collectée](#9--analyse-de-la-session-collectée)
10. [Synthèse des résultats](#10--synthèse-des-résultats)
11. [Compétences démontrées](#11--compétences-démontrées)
12. [Bonnes pratiques](#12--bonnes-pratiques)
13. [Structure du dépôt](#13--structure-du-dépôt)
14. [Ressources et crédits](#14--ressources-et-crédits)

---

## 1. 📌 Présentation du projet

Ce laboratoire fait partie de mon parcours de montée en compétences **Blue Team / DFIR** (*Digital Forensics and Incident Response*). Il aborde la **phase de collecte** après la compromission d'un poste Windows : rassembler le plus d'informations possibles, de façon structurée et rapide, afin de comprendre ce qui s'est passé.

**FireEye Redline** propose une solution « tout-en-un » : au lieu d'utiliser un outil différent pour la mémoire, le registre, les journaux ou l'historique web, un **collecteur configurable** rassemble toutes ces données en une seule exécution.

Le projet couvre :

- l'installation de Redline 2 ;
- la configuration d'un **collecteur standard** et la compréhension de ses options ;
- l'**exécution** d'un collecteur depuis un support amovible ;
- la **collecte de mémoire** (dump) ;
- la comparaison avec les collecteurs **complet** et **par défaut** ;
- l'**ouverture d'une session d'analyse** et une première exploration des données.

---

## 2. 🎯 Objectifs

| # | Objectif |
|---|----------|
| 1 | Comprendre le rôle de Redline dans une réponse à incident |
| 2 | Installer Redline 2 sur un poste d'analyse |
| 3 | Configurer un collecteur (mémoire, disque, système, réseau, navigateur) |
| 4 | Collecter les données d'un poste Windows « compromis » et acquérir sa mémoire |
| 5 | Comparer les types de collecteurs (standard, complet, par défaut, IOC) |
| 6 | Ouvrir la session collectée et explorer processus, utilisateurs, journaux et timeline |

---

## 3. 📚 Notions théoriques

### 3.1 Qu'est-ce que Redline ?

**Redline** est un **outil gratuit de sécurité des postes de travail** développé par **FireEye**, utilisé par les intervenants en réponse à incident pour **collecter un maximum d'informations** sur un système compromis. Les données collectées sont ensuite analysées pour détecter des signes d'activité malveillante grâce à l'**analyse de la mémoire et des fichiers**, et à l'élaboration d'un **profil d'évaluation de la menace**.

### 3.2 Ce que Redline collecte

- **processus et pilotes** en cours d'exécution (depuis la mémoire) ;
- **métadonnées du système de fichiers** ;
- **registre** ;
- **journaux d'événements** ;
- **informations réseau**, **services** et **tâches** ;
- **historique web** (navigateurs).

### 3.3 Fonctions d'analyse

Redline ne sert pas qu'à collecter : il permet aussi d'**analyser** les données importées, de filtrer les résultats par période grâce à sa **timeline** (avec les modules tiers *TimeWrinkle* et *TimeCrunch*), et d'effectuer une **analyse d'indicateurs de compromission (IOC)**. Ce laboratoire se concentre sur la **collecte**, avec un aperçu de l'interface d'analyse.

### 3.4 Pourquoi Windows ?

Windows détient la majorité des parts de marché des postes en entreprise : c'est le système que l'on a le plus de chances d'avoir à investiguer. Un intervenant doit donc maîtriser la **collecte d'informations sur Windows**.

### 3.5 Prérequis

Le laboratoire fait suite à la collecte de mémoire sous Linux avec **LiME** et à l'analyse de mémoire avec **Volatility** (voir mon [laboratoire Volatility](../volatility/README.md)).

> 💡 Mon retour d'expérience : pour l'**analyse** de dumps mémoire, je préfère Volatility. L'interface graphique de Redline peut dérouter à cause de sa nomenclature, mais pour la **collecte tout-en-un**, Redline est très efficace.

---

## 4. 🧪 Environnement et installation

| Élément | Détail |
|---|---|
| Système | **Windows 10** (virtualisé), utilisé aussi pour la forensique de disque |
| Outils déjà présents | Autopsy et FTK Imager |
| Outil du lab | **FireEye Redline 2** (la version 1.2 a aussi été téléchargée pour comparaison) |

### 4.1 Téléchargement

Redline est **gratuit**. Le téléchargement demande de saisir ses informations, dont une adresse e-mail, **sans vérification** de celle-ci. Deux versions sont proposées ; la **version 2** est la plus récente. Un **guide utilisateur Redline** détaille toute la chaîne, de la collecte à l'analyse.

<!-- 📸 IMAGE : page de téléchargement de Redline (version 2 et version 1.2) -->
![Téléchargement de Redline](images/01-telechargement-redline.png)

### 4.2 Installation

L'installation se fait avec un **assistant simple** (double-clic sur l'installateur de la version 2, puis suivre les étapes).

<!-- 📸 IMAGE : assistant d'installation de Redline 2 -->
![Installation de Redline](images/02-installation-redline.png)

### 4.3 Écran d'accueil

Redline propose deux grandes sections : **Collect Data** (collecter) et **Analyze Data** (analyser). Cette partie du lab utilise la première.

<!-- 📸 IMAGE : écran d'accueil de Redline avec les sections Collect Data et Analyze Data -->
![Écran d'accueil](images/03-ecran-accueil.png)

---

## 5. 🧰 Types de collecteurs

| Collecteur | Usage |
|---|---|
| **Standard** (recommandé) | Collecte les données nécessaires à une analyse centrée sur la **mémoire** ; plus rapide et meilleur du point de vue de l'analyse |
| **Complet (Comprehensive)** | Rassemble **le maximum de données** pour une analyse approfondie ; peut durer **jusqu'à une heure** |
| **IOC Search** | **Windows uniquement** ; collecte les données correspondant à des **indicateurs de compromission** définis au préalable (lié au renseignement sur les menaces) |
| **Par défaut** | Configuration minimale : très rapide, n'inclut pas le disque, le réseau, etc. |

Chaque collecteur peut être créé pour **Windows** ou pour **Linux**.

<!-- 📸 IMAGE : menu de création des collecteurs (standard, complet, IOC search) -->
![Types de collecteurs](images/04-types-collecteurs.png)

---

## 6. 🛠️ Création du collecteur standard

### 6.1 Principe de fonctionnement

Le principe est de **ne jamais exécuter Redline sur le système dont on collecte les données**. À la place :

1. l'analyste prépare le collecteur sur **son propre poste** équipé de Redline ;
2. il l'enregistre sur une **clé USB ou un disque externe** ;
3. il connecte ce support au **système compromis** ;
4. il y exécute le script de collecte (un fichier **batch**) ;
5. il **rapatrie les résultats** sur son poste d'analyse.

<!-- 📸 IMAGE : écran « Create a Standard Collector » avec le choix de la plateforme -->
![Création du collecteur standard](images/05-creation-collecteur-standard.png)

### 6.2 Option clé : acquisition de la mémoire

Pour Windows, l'option **Acquire a memory image** permet de capturer une image de la mémoire, utilisée ensuite pour analyser de façon fiable les processus et les pilotes.

<!-- 📸 IMAGE : option « Acquire a memory image » cochée -->
![Acquisition de la mémoire](images/06-acquisition-memoire.png)

### 6.3 Personnalisation de la collecte (*Edit your script*)

Redline permet de choisir très finement ce qui est collecté :

| Catégorie | Options configurées |
|---|---|
| **Mémoire** | Liste des processus (handles, sections, imports, exports, **détection de sections injectées**, **vérification des signatures numériques**, chaînes), énumération des pilotes, **détection de hooks** |
| **Disque** | Énumération des fichiers (laissée décochée car très longue), fichiers actifs, index NTFS, **analyse d'entropie** (détection de chiffrement), imports, répertoires, **anomalies de fichiers** |
| **Système** | Informations machine et OS, **comptes utilisateurs**, points de restauration, **registre**, **journaux d'événements** |
| **Réseau** | **Ports**, table **ARP**, **table de routage** (un attaquant peut modifier le routage) |
| **Navigateur** | Cookies, téléchargements, historique d'URL, contenu des pages indexées (Firefox), historique de formulaires |
| **Autres** | **Services** (avec hash MD5/SHA-256), **mécanismes de persistance** |

> 🔎 La collecte des **mécanismes de persistance** est essentielle : elle permet de déterminer si l'attaquant a mis en place un moyen de revenir sur le système.

<!-- 📸 IMAGE : fenêtre de personnalisation, onglet Memory -->
![Options Memory](images/07-options-memory.png)

<!-- 📸 IMAGE : fenêtre de personnalisation, onglets Disk et System -->
![Options Disk et System](images/08-options-disk-system.png)

<!-- 📸 IMAGE : fenêtre de personnalisation, onglets Network et Browser -->
![Options Network et Browser](images/09-options-network-browser.png)

### 6.4 Enregistrement du collecteur

Après validation, on indique l'**emplacement du collecteur** (idéalement un support externe). Redline génère alors le **collecteur portable**.

<!-- 📸 IMAGE : choix de l'emplacement d'enregistrement du collecteur -->
![Emplacement du collecteur](images/10-emplacement-collecteur.png)

<!-- 📸 IMAGE : message « Your collector package is created » -->
![Collecteur créé](images/11-collecteur-cree.png)

### 6.5 Contenu du package

Le dossier généré contient notamment :

- le script **`RedlineAudit.bat`** à exécuter sur le système cible ;
- le fichier de configuration **`MemoryAuditScript.xml`**, modifiable sur le système compromis pour ajuster les données à collecter (y compris le dump mémoire) ;
- des dossiers **x64** et **x86** contenant les composants utilisés pour la collecte.

<!-- 📸 IMAGE : contenu du dossier du collecteur portable (RedlineAudit.bat, MemoryAuditScript.xml, x64, x86) -->
![Contenu du package](images/12-contenu-package.png)

---

## 7. ▶️ Exécution de la collecte sur le système compromis

### 7.1 Lancement

Le dossier du collecteur est transféré sur le système « compromis ». Le script **`RedlineAudit.bat`** est exécuté **en tant qu'administrateur** ; une fenêtre s'ouvre et la collecte démarre selon la configuration du fichier XML.

<!-- 📸 IMAGE : lancement de RedlineAudit.bat en tant qu'administrateur -->
![Lancement du collecteur](images/13-lancement-collecteur.png)

<!-- 📸 IMAGE : fenêtre de collecte en cours -->
![Collecte en cours](images/14-collecte-en-cours.png)

### 7.2 Résultats

Les résultats sont enregistrés dans un dossier **`Sessions`**, sous-dossier **`AnalysisSession1`** ; **un nouveau dossier de session est créé à chaque exécution**. Le fichier **`AnalysisSession1.mans`** est le projet Redline à ouvrir pour l'analyse.

> ⏱️ Avec beaucoup d'options cochées, la collecte est longue : une première collecte très chargée, estimée entre **30 minutes et 1 heure**, a été interrompue, et une seconde collecte plus ciblée a été lancée. Il faut toujours **gérer le temps** disponible pendant un incident.

<!-- 📸 IMAGE : dossier Sessions\AnalysisSession1 avec le fichier .mans -->
![Dossier de session](images/15-dossier-session.png)

---

## 8. ⚖️ Collecteurs complet et par défaut

### 8.1 Collecteur complet (Comprehensive)

Toutes les options sont activées par défaut. Il fournit **toutes les données nécessaires à une analyse approfondie**, mais peut durer jusqu'à **une heure**. Il convient quand on veut vraiment comprendre ce qui s'est passé.

<!-- 📸 IMAGE : écran de création du collecteur complet (Comprehensive) -->
![Collecteur complet](images/16-collecteur-complet.png)

### 8.2 Collecteur par défaut

La configuration par défaut **n'inclut pas** la collecte disque, système étendue ni réseau. Résultat : une exécution **beaucoup plus rapide**. L'exécution du script déclenche l'invite **UAC** (droits administrateur), puis la collecte se termine en peu de temps. Dans ce cas précis, **aucun dump mémoire** n'avait été demandé.

<!-- 📸 IMAGE : options du collecteur par défaut -->
![Options par défaut](images/17-options-defaut.png)

<!-- 📸 IMAGE : invite UAC lors du lancement du script -->
![Invite UAC](images/18-invite-uac.png)

### 8.3 Collecteur avec image mémoire

Un nouveau collecteur standard a ensuite été créé avec :

- **Acquire a memory image** ;
- la collecte des **comptes utilisateurs** ;
- la collecte des **journaux d'événements**.

Il est plus long (à cause du dump mémoire) mais reste raisonnable, et **bien plus rapide** que d'enchaîner manuellement plusieurs outils pour collecter registre, journaux et mémoire.

<!-- 📸 IMAGE : collecteur standard avec image mémoire, comptes utilisateurs et journaux d'événements -->
![Collecteur avec mémoire](images/19-collecteur-avec-memoire.png)

---

## 9. 🔬 Analyse de la session collectée

Le fichier **`AnalysisSession1.mans`** est ouvert sur le poste d'analyse (double-clic). Redline importe les données et les présente dans une interface simple.

<!-- 📸 IMAGE : ouverture de la session .mans dans Redline -->
![Ouverture de la session](images/20-ouverture-session.png)

### 9.1 Navigation et modes d'investigation

La barre latérale donne accès à : **System Information**, **Processes**, **Hierarchy**, **Users**, **Event Logs**, **Timeline**, **Tags**, etc. Redline propose plusieurs points de départ : examen d'une collecte de triage **HX** (plateforme FireEye), investigation d'un hôte à partir d'un **indice externe**, revue de **l'historique web** ou recherche par **IOC**.

<!-- 📸 IMAGE : accueil de la session avec la barre latérale et les modes d'investigation -->
![Interface d'analyse](images/21-interface-analyse.png)

### 9.2 Informations système

Affiche le **nom de la machine**, la **date et l'heure**, le **fuseau horaire**, les **disques**, la **mémoire physique totale et disponible**, la **durée de fonctionnement**, le **BIOS** et la **version de Windows** (build, correctifs, architecture, répertoire système, date d'installation, propriétaire enregistré).

<!-- 📸 IMAGE : écran System Information -->
![Informations système](images/22-system-information.png)

### 9.3 Processus

La liste des processus actifs au moment de la collecte peut être **filtrée** (nom, PID, chemin, arguments, nom d'utilisateur, **SID** et type de SID) et **recherchée** (par exemple `lsass.exe`, chargé de l'authentification des utilisateurs).

Le **menu contextuel** permet de **taguer** un processus : *follow up*, *escalate*, *clean*, *include in report*, *false positive*. La **vue hiérarchique** montre les relations parent-enfant (par exemple `cmd.exe` et `conhost.exe`, qui correspondent à l'outil de collecte lui-même).

<!-- 📸 IMAGE : liste des processus avec recherche de lsass.exe -->
![Processus](images/23-processus.png)

<!-- 📸 IMAGE : menu contextuel avec les options de tag -->
![Tags](images/24-tags-processus.png)

<!-- 📸 IMAGE : vue hiérarchique des processus -->
![Hiérarchie des processus](images/25-hierarchie-processus.png)

### 9.4 Utilisateurs

Liste des **comptes utilisateurs** avec la dernière connexion, l'**âge du mot de passe**, la présence d'un mot de passe requis et le **SID** (qui permet de confirmer le compte administrateur).

> 🔎 **Point de contrôle en cas de compromission :** les comptes **Invité** et **Administrateur par défaut** devraient être **désactivés**. S'ils sont activés, c'est un signal d'alerte.

<!-- 📸 IMAGE : liste des utilisateurs -->
![Utilisateurs](images/26-utilisateurs.png)

### 9.5 Journaux d'événements

Les journaux sont très volumineux : on les filtre par **type** (avertissement, information, erreur) et, de préférence, par **identifiant d'événement**. C'est ainsi qu'on repère les événements importants.

<!-- 📸 IMAGE : journaux d'événements avec un filtre appliqué -->
![Journaux d'événements](images/27-journaux-evenements.png)

### 9.6 Timeline

La **timeline** retrace l'activité du système : fichiers **créés** ou **modifiés**, dernières connexions, etc. Elle permet notamment, en cas de **ransomware**, de déterminer **quand le système a été infecté**, quand le ransomware a été exécuté et quand les fichiers ont été chiffrés.

<!-- 📸 IMAGE : onglet Timeline avec filtre sur les fichiers créés ou modifiés -->
![Timeline](images/28-timeline.png)

---

## 10. ✅ Synthèse des résultats

| Étape | Résultat |
|---|---|
| Installation de Redline 2 | ✅ Réalisée |
| Collecteur standard personnalisé | ✅ Mémoire, disque, système, réseau, navigateur configurés |
| Exécution sur le système cible | ✅ Via `RedlineAudit.bat` en administrateur |
| Collecteur par défaut | ✅ Collecte rapide, sans dump mémoire |
| Collecteur avec image mémoire | ✅ Mémoire, comptes et journaux collectés |
| Ouverture de la session `.mans` | ✅ Données importées dans l'interface d'analyse |
| Exploration | ✅ Système, processus, utilisateurs, journaux, timeline |

**Conclusion :** Redline permet de collecter **en une seule opération** des données qui exigeraient autrement plusieurs outils (registre, journaux, mémoire, historique web). Il est idéal pour la **phase de collecte** d'une réponse à incident, à condition de bien **choisir le type de collecteur** selon le temps disponible.

---

## 11. 🎓 Compétences démontrées

- **Réponse à incident** : collecte structurée sur un poste Windows compromis
- **Acquisition de mémoire** et collecte multi-sources (registre, journaux, processus, réseau, navigateur)
- **Configuration fine d'un collecteur** selon l'objectif et le temps disponible
- **Méthodologie forensique** : exécution depuis un support amovible, sans installer d'outil sur la cible
- **Lecture d'artefacts** : processus, SID, comptes, journaux d'événements, timeline
- **Détection d'indices de compromission** : comptes activés par défaut, persistance, processus suspects
- **Annotation d'éléments** (tags) pour le suivi d'enquête
- **Comparaison d'outils** : Redline et Volatility

---

## 12. 🛡️ Bonnes pratiques

- **Ne pas installer ni exécuter Redline sur le système investigué** : préparer le collecteur sur un autre poste et l'exécuter depuis un **support amovible** ;
- Choisir le collecteur selon le **temps disponible** (standard, complet ou par défaut) ;
- Enregistrer les résultats sur un support **distinct** du système collecté ;
- Exécuter le script avec les **droits administrateur** ;
- **Taguer** les éléments à suivre pour faciliter le rapport ;
- Contrôler les **comptes par défaut** (Invité, Administrateur) et les **mécanismes de persistance** ;
- Croiser les résultats avec d'autres outils (par exemple Volatility pour la mémoire).

---

## 13. 🗂️ Structure du dépôt

```text
redline-incident-response-lab/
├── README.md
├── images/
│   ├── 00-vue-ensemble.png
│   ├── 01-telechargement-redline.png
│   ├── ...
│   └── 28-timeline.png
└── notes/
    └── options-collecteur.md   (optionnel : récapitulatif des options choisies)
```

---

## 14. 📎 Ressources et crédits

- **FireEye Redline** : outil de collecte et d'analyse de données en réponse à incident, avec son **guide utilisateur**
- **Série Blue Team Training** (HackerSploit, avec Linode) : démonstration suivie pour ce laboratoire, reproduite et documentée ici
- Laboratoires liés : [Volatility](../volatility/README.md) · [Autopsy](../autopsy/README.md)

---

### 👤 Auteur

**Dannon Rock** : Réseaux, systèmes et cybersécurité
🔗 [LinkedIn](https://www.linkedin.com/in/rock-dannon)

> ℹ️ *Ce dépôt est un projet d'apprentissage réalisé dans un environnement de laboratoire isolé.*
