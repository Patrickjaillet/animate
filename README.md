<div align="center">

# Animate

**Logiciel d'animation 2D cutout avec export vidéo, pour Windows 11.**

[![Version](https://img.shields.io/badge/version-1.0.0-blue)](CHANGELOG.md)
[![Licence](https://img.shields.io/badge/licence-Freeware-green)](LICENSE)
[![Plateforme](https://img.shields.io/badge/plateforme-Windows%2011-0078D6)](#prérequis)

[Télécharger](#téléchargement) · [Fonctionnalités](#fonctionnalités) · [Captures d'écran](#captures-décran) · [À propos](#à-propos)

</div>

---

## Sommaire

- [Présentation](#présentation)
- [Fonctionnalités](#fonctionnalités)
- [Captures d'écran](#captures-décran)
- [Prérequis](#prérequis)
- [Téléchargement](#téléchargement)
- [Installation](#installation)
- [Utilisation rapide](#utilisation-rapide)
- [Internationalisation](#internationalisation)
- [Licence](#licence)
- [Composants tiers](#composants-tiers)
- [À propos](#à-propos)

---

## Présentation

Animate est un logiciel d'animation 2D en cutout (découpage articulé) permettant
de composer des scènes à partir d'une bibliothèque d'assets (personnages, décors,
objets), de les animer sur une timeline à base de keyframes, puis d'exporter le
résultat en vidéo via FFmpeg.

L'espace de travail réunit quatre zones redimensionnables : **Bibliothèque**
(gauche), **Canvas de scène** (centre, rendu SkiaSharp), **Inspecteur de
propriétés** (droite) et **Timeline** (bas), plus une **barre de plans** pour
les projets multi-scènes. Le rendu du canvas tient largement au-delà de 60 fps
sur un plan chargé (voir [PERFORMANCE.md](docs/PERFORMANCE.md)).

## Fonctionnalités

- **Bibliothèque d'assets** — personnages, décors, objets, texte/FX, audio ;
  recherche, catégories, tags, favoris, import de SVG/PNG.
- **Canvas SkiaSharp** — compositing par z-index, transformations, opacité,
  masques de découpe, grille/règles/guides/snapping, zoom/pan, sélection et
  manipulation directe.
- **Timeline à keyframes** — pistes par calque, interpolations linéaire /
  ease-in / ease-out / stepped, lecture temps réel, marqueurs, audio synchronisé
  avec waveform, lip-sync simplifié (phonème → bouche), cycles d'animation
  prédéfinis (marche, parole, idle).
- **Multi-scènes** — barre de plans : liste, sélection, ajout, réorganisation ;
  modèle « storyboard » 3 scènes.
- **Inspecteur** — transformations, opacité, recoloration d'assets vectoriels
  par zones, effets plats (ombre portée, contour), réglages audio par clip.
- **Export vidéo** — MP4 H.264/AAC via FFmpeg embarqué, mixage audio (volume,
  position, fondus), presets 480p / 1080p, file d'attente, export d'image fixe,
  progression en temps réel.
- **Projet `.animate`** — JSON versionné SemVer, migration ascendante,
  undo/redo global, auto-save toutes les 30 s et récupération après crash.
- **Interface française et anglaise**, changement de langue à chaud.
- **Projets récents** avec vignette de la dernière scène.

## Captures d'écran

| Accueil | Canvas + Bibliothèque |
|---|---|
| ![Écran d'accueil d'Animate](docs/img/01-accueil.png) | ![Canvas de scène et bibliothèque d'assets](docs/img/02-canvas-bibliotheque.png) |

| Timeline | Export |
|---|---|
| ![Timeline avec pistes et marqueurs](docs/img/03-timeline.png) | ![Export MP4 terminé](docs/img/04-export.png) |

## Prérequis

- Windows 11 (64 bits, x64)
- Aucun runtime à installer : l'application est distribuée autonome
  (self-contained) avec FFmpeg embarqué

## Téléchargement

Rendez-vous sur la page
[**Releases**](https://github.com/Patrickjaillet/animate/releases/latest) :

- **`Animate-1.0.0-Setup.exe`** — installateur Windows (recommandé) ;
- **`Animate-1.0.0-win-x64.zip`** — version portable (décompresser et lancer
  `Animate.exe`).

## Installation

L'installateur installe l'application, FFmpeg, les assets et les fichiers de
langue, puis peut créer un raccourci Bureau et associer les fichiers `.animate`.
La désinstallation se fait depuis *Paramètres Windows › Applications* et retire
l'ensemble des fichiers installés.

## Utilisation rapide

1. **Nouveau projet** depuis l'écran d'accueil (ou un modèle : plan vierge,
   storyboard).
2. Faites glisser un asset de la **Bibliothèque** vers le **Canvas** pour créer
   un calque.
3. Ajustez position, rotation, échelle, opacité et couleurs dans l'**Inspecteur**.
4. Posez des **keyframes** sur la **Timeline**, ajoutez marqueurs et pistes
   audio, puis prévisualisez avec Lecture.
5. **Exporter** (bouton de la barre de titre) → choisissez un fichier `.mp4` ;
   l'encodage H.264/AAC via FFmpeg s'exécute en arrière-plan, une notification
   confirme la fin.

`Ctrl+S` / `Ctrl+O` enregistrent et ouvrent un projet `.animate` (raccourcis
personnalisables dans les Réglages).

## Internationalisation

Animate est disponible en **français** et en **anglais**. La langue peut être
changée depuis l'écran d'accueil ou l'écran Réglages, sans redémarrage du
logiciel.

## Licence

Animate est distribué sous licence **Freeware** — le code source n'est pas
fourni. Voir le fichier [LICENSE](LICENSE) pour le détail des conditions
d'utilisation.

## Composants tiers

Animate embarque et/ou s'appuie sur les composants tiers suivants :

- **FFmpeg** — encodage vidéo (binaire embarqué ; mention légale complète dans
  l'onglet *À propos* du logiciel : licence LGPL/GPL selon les composants)
- **Baloo 2** (The Baloo 2 Project Authors) — police display arrondie,
  [SIL Open Font License 1.1](https://openfontlicense.org/)
- **JetBrains Mono** (The JetBrains Mono Project Authors) — police technique,
  [SIL Open Font License 1.1](https://openfontlicense.org/)
- Runtime .NET 8 et SkiaSharp, embarqués, sous leurs licences respectives

## À propos

**Animate**
Copyright © 2026 Patrick JAILLET — Tous droits réservés
E-mail : sandefjord.development@proton.me
Site web : https://patrickjaillet.github.io/animate

