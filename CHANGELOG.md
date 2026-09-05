# Changelog

Toutes les modifications notables apportées à ce projet seront documentées
dans ce fichier.

Le format est basé sur [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/),
et ce projet adhère au [Semantic Versioning](https://semver.org/lang/fr/).

## [Unreleased]

## [1.0.0] - 2026-09-05

### Corrigé

- Démarrage : la fenêtre principale ne s'affichait plus (rectangle blanc) car
  `FluentWindow` (WPF-UI) n'avait aucun template — les dictionnaires de
  contrôles WPF-UI n'étaient jamais fusionnés. La fenêtre principale est
  désormais une `Window` standard avec chrome custom (`WindowChrome` +
  `CustomTitleBar`), sans dépendance de fenêtrage tierce ; le paquet WPF-UI,
  devenu inutilisé, a été retiré.
- Rendu : plantage `InvalidOperationException` (« `{DependencyProperty.UnsetValue}`
  n'est pas une valeur valide pour la propriété `Foreground` ») au premier
  `Measure`. Les tokens de thème étaient fusionnés via un dictionnaire
  intermédiaire (`Themes/Tokens.xaml`) dont l'imbrication empêchait la
  résolution de certains `StaticResource` dans les styles implicites ; les
  dictionnaires de tokens sont maintenant fusionnés à plat dans `App.xaml`.
- Installateur : `AppId` Inno Setup corrigé (GUID malformé qui compromettait
  la détection de mise à jour et la désinstallation).
- `scripts/release.ps1` : encodage UTF-8 (BOM) pour un parsing fiable sous
  Windows PowerShell comme sous PowerShell 7.
- Rendu : `SceneRenderer.RenderLayer` appelait `SKCanvas.SaveLayer` pour chaque
  calque sans condition (tampon hors écran plein écran recomposé 24×/frame).
  `SaveLayer` n'est plus utilisé que pour un calque translucide portant des
  effets ; sinon l'alpha est appliqué directement au tracé. Rendu du canvas :
  ~183 → ~7 ms/frame sur un plan de 24 calques (voir `docs/PERFORMANCE.md`).
- Export : la rasterisation frame par frame s'exécutait sur le thread UI et
  figeait la fenêtre pendant tout l'export ; elle est déplacée sur un thread de
  fond (`Task.Run`).
- Auto-save / récupération après crash : le service `AutoSaveService` existait
  mais n'était pas branché dans l'application. `MainWindow` démarre désormais
  une sauvegarde de secours toutes les 30 s (`%APPDATA%\Animate\recovery.animate`,
  uniquement projet ouvert), propose de restaurer ce fichier au redémarrage
  si la session précédente s'est terminée par un crash, et l'efface à la
  fermeture propre. La boîte de récupération est déclenchée sur
  `ContentRendered` (et non `Loaded`) pour que la fenêtre principale soit
  peinte derrière, plutôt qu'un dialogue orphelin.
- Export vidéo : aucun retour visuel pendant la rasterisation (qui dure
  ~50 s pour 240 frames en 1080p). Ajout d'un toast « Export en cours… », de
  la progression en pourcentage dans la barre de statut, et d'un toast de fin ;
  la barre de statut revient à « Prêt » à la fin. `IProgress<double>` de
  `VideoExportService` désormais consommé par l'UI.

### Ajouté

- Tests unitaires dédiés du moteur `Animate.Timeline` (projet
  `Animate.Timeline.Tests`, jusqu'ici vide) : `TimelineMath` (conversion
  frame ↔ pixels, arrondi, garde-fous), `TimelineClock` (clamp, événement
  `FrameChanged`, lecture/pause/stop) et `AnimationCycleLibrary` (cycles
  intégrés, instanciation, cas d'erreur) — 19 tests.
- Distribution (`scripts/release.ps1`) : signature optionnelle de
  `Animate.exe`, `ffmpeg.exe` et de l'installateur (`-Sign`, certificat via
  `ANIMATE_SIGN_PFX`/`ANIMATE_SIGN_CERT_THUMBPRINT`) ; vérification stricte du
  package en trois contrôles (fichiers interdits, liste blanche d'extensions,
  présence des éléments requis) ; génération automatique des notes de version
  depuis `CHANGELOG.md` vers `scripts/release-notes/<version>.md` et
  `/dist/RELEASE-NOTES-<version>.md`.
- Tests du pipeline d'export : `FfmpegProgressParser`, `ExportPresets` et
  `VideoExportService` avec un ffmpeg simulé (rendu des frames, invocation,
  progression, annulation, validation) — 23 tests.
- Tests de sérialisation projet (`ProjectSerializationTests`) : round-trip
  résolution/fps/scènes/keyframes/transform/audio, écriture atomique, et
  migration ascendante (version manquante, mineure ancienne, majeure trop
  récente, invalide, JSON corrompu).
- Revue de performance : `docs/PERFORMANCE.md` et `PerformanceTests`
  (budget 60 fps du canvas, empreinte mémoire de l'index d'assets).
- `README.md` complété : 4 captures d'écran (`docs/img/`), guide de prise en
  main, lien de téléchargement Releases.
- Site de présentation GitHub Pages : `docs/index.html` (vitrine statique,
  captures, fonctionnalités, installation, licence) + `docs/.nojekyll`.
- Versioning : `SemanticVersion` (parsing/comparaison SemVer stricts) et
  `GitHubReleaseUpdateChecker` (vérification de mise à jour via l'API Releases,
  échec réseau silencieux) dans `Animate.Core` ; l'onglet *À propos* affiche
  désormais l'état de mise à jour. `scripts/release.ps1 -BumpVersion` bumpe
  `version.json`, promeut `[Unreleased]` → `[X.Y.Z] - <date>` dans le CHANGELOG,
  et crée le commit + tag annoté `vX.Y.Z`.

- Tests : moteur de keyframes (`KeyframeInterpolationTests` — linéaire,
  ease-in/out, stepped, multi-segments, bornes, `SetKeyframe`) et
  `AutoSaveServiceTests` (écriture périodique, récupération, effacement).

### Corrigé (audit 1.0.0)

- Robustesse : `App` installe des gestionnaires d'exceptions globaux
  (`DispatcherUnhandledException`, `AppDomain.UnhandledException`) qui
  journalisent dans `%APPDATA%\Animate\crash.log` et affichent un message clair
  au lieu d'un crash silencieux ; langue invalide dans les réglages → repli
  sur `fr` au lieu de `CultureNotFoundException`.
- Robustesse E/S : `UserSettingsStore`, `RecentProjectsStore`,
  `JsonLocalizationService.Load`, `AssetLibraryIndex.Refresh`, l'import d'assets,
  l'import audio, la génération de vignettes et le chargement/enregistrement de
  projet interceptent désormais `IOException`/`UnauthorizedAccessException` (et
  `ProjectFormatException` pour un `.animate` invalide) et affichent une erreur
  au lieu de planter.
- Rendu : un SVG absent ou malformé produisait une exception fatale dans le
  handler `PaintSurface` (crash total). `SvgPictureCache` met en cache une image
  vide sur échec de parsing ; `CanvasPanel.OnPaintSurface` est protégé.
- `AssetThumbnailCache` : clé de cache stable (SHA-256) au lieu de
  `string.GetHashCode()` (randomisé par processus → cache jamais réutilisé).
- Réglages : « Dossier des assets » et « Chemin FFmpeg » étaient sauvegardés
  mais **ignorés**. `AppPaths` les applique désormais (bibliothèque et export)
  au démarrage et à l'enregistrement des réglages.
- Écran d'accueil : cliquer un projet récent ouvrait un sélecteur de fichier
  au lieu du projet ; le modèle « Storyboard » ne créait pas 3 scènes ; la
  vignette du projet récent verrouillait le fichier PNG
  (`BitmapCacheOption.OnLoad`).
- Inspecteur : les modifications de calque (transform, couleurs, effets) ne
  rafraîchissaient pas le canvas (`PropertiesChanged` n'était abonné nulle part).
- Réglages : les deux champs de raccourci se superposaient (lignes de Grid
  manquantes dans le XAML).
- Multi-scènes : la gestion des plans existait dans le modèle mais sans aucune
  UI. Ajout d'une barre de plans (`SceneBar`) : liste, sélection, ajout,
  réorganisation ; toutes les opérations (drag&drop, timeline, export,
  vignette, récupération) passent par le plan actif.
- i18n : extraction de toutes les chaînes des dialogues, notifications,
  messages d'erreur, écran À propos, écran Réglages, barre de plans et écran
  d'accueil vers `fr.json` / `en.json` (83 clés, parité garantie par un test) ;
  bascule de langue à chaud sur ces écrans.
- `AutoSaveService` : une erreur d'écriture (disque plein, dossier supprimé,
  permissions) dans le tick du timer remontait en exception non gérée sur un
  thread de fond = **crash du process**. Le tick intercepte désormais
  `IOException`/`UnauthorizedAccessException`/`ObjectDisposedException` et
  ignore les ticks après `Dispose()`.
- `AppVersionInfo.DisplayVersion` : normalisation en SemVer 3 composants
  (`1.0.0` et non `1.0.0.0` ni `1.0.0-g<hash>`) pour l'affichage dans
  l'onglet À propos.

### Retiré

- `MainWindow.VersionLabel` (propriété morte — la version est affichée par
  `AboutWindow`).
- Fichiers `PlaceholderInfo.cs` (squelettes de la PHASE 0, jamais référencés).

### Ajouté (fonctionnalités)

- Packaging Windows 11 : icône multi-résolution, installateur Inno Setup 7,
  raccourcis Start Menu/Bureau, association `.animate` et script de release
  produisant l'installateur dans `/dist/installer`
- Écrans transverses : fenêtre À propos avec version et mentions FFmpeg,
  fenêtre Réglages persistante, aide contextuelle et base de stockage des
  projets récents ; raccourcis Ctrl+S/Ctrl+O configurables, sauvegarde et
  ouverture des projets `.animate`, miniatures PNG des scènes et affichage
  du dernier projet sur l'écran d'accueil
- Internationalisation JSON française et anglaise : service de traduction avec
  fallback, changement de culture à chaud depuis l'écran d'accueil et
  formatage localisé des nombres et durées
- Correction du démarrage WPF : suppression des dictionnaires de thème WPF-UI
  qui fournissaient un `Foreground` non résolu (`UnsetValue`), maintien de la
  fenêtre principale avant la fermeture du splash et cycle de vie configuré
  sur `OnMainWindowClose`
- Export vidéo MP4 : rasterisation offline des scènes en PNG, encodage H.264/AAC
  via FFmpeg embarqué, mixage des clips audio avec volume, position et fondus,
  progression de traitement et nettoyage des fichiers temporaires ; presets
  480p/1080p, file d'attente séquentielle et export PNG d'une frame ajoutés
- Inspecteur de propriétés complété : édition des couleurs SVG par zone via les
  overrides de l'asset, et sélection des clips audio depuis la timeline avec
  réglage du nom, du volume et des fondus d'entrée/sortie en frames
- Fenêtre principale : barre de titre custom (`Animate.App.Controls.CustomTitleBar`)
  remplaçant le chrome système — glyphes vectoriels redessinés pour
  réduire/agrandir-restaurer/fermer (trait épais, sans police d'icônes),
  zone de drag avec double-clic pour agrandir/restaurer, snap Windows 11
  préservé via `WindowChrome`
- Système de design tokens centralisé (`Animate.App/Themes/`) : `Colors.xaml`
  (palette thème "studio" sombre, accents cel-shading rouge/orange/bleu/vert/jaune),
  `Spacing.xaml` (échelle base 4px), `Radii.xaml`, `Shadows.xaml`, fusionnés
  via `Tokens.xaml` et référencés dans `App.xaml`
- `CustomTitleBar` et `MainWindow` migrés pour consommer exclusivement les
  tokens centralisés (plus aucune couleur codée en dur)
- Typographie custom (`Animate.App/Themes/Typography.xaml`) : police display
  arrondie **Baloo 2** (titres, en-têtes de panneaux, écran d'accueil) et
  police technique **JetBrains Mono** (UI dense : inspecteur de propriétés,
  timeline, valeurs numériques/timecodes en chasse fixe) ; polices variables
  embarquées en `Resource` WPF (`assets/fonts/`, licence SIL OFL 1.1) et
  référencées via `FontFamily.Display` / `FontFamily.UI` / `FontFamily.Body` ;
  échelle `FontSize.*` et styles `Typo.*` prêts à l'emploi ; `Window` et
  `TextBlock` (`AppStyles.xaml`) migrés pour consommer les tokens au lieu de
  `Segoe UI` codé en dur
- Iconographie cohérente (`Animate.App/Themes/Icons.xaml`) : 44 icônes
  vectorielles dessinées à main levée, style "papercraft" (coins coupés,
  trait épais uniforme `Icon.StrokeThickness`, jonctions anguleuses, grille
  source 24×24), couvrant navigation (`Library`, `Canvas`, `Inspector`,
  `Timeline`, `Settings`, `Info`), fichier/projet (`NewFile`, `Open`, `Save`,
  `Recent`, `Import`, `Export`), édition canvas (`Select`, `Move`, `Rotate`,
  `Scale`, `Crop`, `Layers`), timeline/lecture (`Play`, `Pause`, `Stop`,
  `SkipPrevious`/`SkipNext`, `Keyframe`, `Marker`, `Audio`, `Loop`), couleur/
  effets (`ColorPicker`, `Stroke`, `Shadow`) et actions génériques (`Add`,
  `Remove`, `Close`, `Check`, `Search`, `Filter`, `Star`, `Trash`, `Undo`,
  `Redo`, `Warning`, `Success`) ; contrôle réutilisable `Controls/Icon.xaml`
  (Viewbox + Path, taille/couleur/épaisseur de trait paramétrables) pour que
  toute vue consomme une clé `Icon.*` plutôt que de dupliquer du balisage
  `<Path Data="...">` ; `CustomTitleBar` migré (bouton fermer) pour valider
  l'intégration
- Layout principal (`MainWindow.xaml`) : quatre zones structurelles —
  Bibliothèque (`Controls/Panels/LibraryPanel`, colonne gauche), Canvas
  scène (`Controls/Panels/CanvasPanel`, colonne centrale, surface
  `Brush.Surface.Sunken`), Inspecteur de propriétés
  (`Controls/Panels/InspectorPanel`, colonne droite) et Timeline
  (`Controls/Panels/TimelinePanel`, bande basse pleine largeur) ;
  redimensionnement des zones via `GridSplitter` stylés
  (`Themes/AppStyles.xaml` — `GridSplitter.Vertical` / `GridSplitter.Horizontal`,
  trait fin qui s'accentue en bleu au survol) avec bornes `MinWidth`/`MaxWidth`/
  `MinHeight`/`MaxHeight` par zone ; en-tête commun `Controls/PanelHeader.xaml`
  (icône + titre, séparateur bas) et contenu provisoire commun
  `Controls/PanelPlaceholder.xaml` (icône atténuée + message + référence à la
  phase du ROADMAP qui implémentera le panneau) réutilisés par les quatre
  panneaux ; galerie de démonstration temporaire de `MainWindow` retirée au
  profit de ces panneaux réels
- Docking léger des panneaux : commandes dans la barre de titre pour masquer
  ou restaurer la Bibliothèque, l'Inspecteur et la Timeline ; les dimensions
  précédentes sont conservées pendant la session et les séparateurs associés
  sont masqués avec leur panneau
- Shell applicatif enrichi : écran de démarrage WPF avec transition d'apparition,
  écran d'accueil avec nouveau projet, ouverture et modèles, barre de statut
  (zoom, fps, durée, état de sauvegarde) et hôte de notifications toast pour
  les succès et erreurs
- Transitions animées lors de l'ouverture du workspace et de l'affichage ou
  masquage des panneaux, avec animations d'apparition et de disparition sans
  dépendance supplémentaire
- Modèle de domaine dans `Animate.Core` : `AnimateProject`, `Scene`, `Layer`,
  `AssetReference`, transformations et pistes de keyframes numériques avec
  interpolations linéaire, ease-in, ease-out et stepped
- Historique global undo/redo basé sur des commandes réutilisables, avec
  commande générique de modification de propriété
- Persistance `.animate` dans `Animate.Persistence` : JSON indenté et versionné,
  migration ascendante de version, écriture atomique et gestion des scènes
- Auto-save récupérable après incident via `AutoSaveStore` et
  `AutoSaveService`, avec couverture de tests du round-trip, de la migration,
  des keyframes et de l'historique d'édition
- Bibliothèque d'assets dans `Animate.AssetLibrary` : indexation des catégories
  et sous-catégories, recherche par nom/tag, favoris de session, import SVG/
  PNG/audio, cache de sources et placeholders SVG papercraft
- Panneau Bibliothèque connecté au Canvas : grille filtrable et glisser-déposer
  créant un `Layer` positionné dans la scène active ; 14 tests couvrent le
  domaine, la persistance et l'indexeur d'assets
- Moteur de rendu SkiaSharp : `SKElement`, compositing des calques visibles,
  transformations, opacité, grille et sélection ; cache SVG basé sur parsing
  XML vers `SKPicture`, décodage bitmap et rendu sur surface GPU
- Canvas interactif : zoom par molette, pan au bouton central, sélection et
  déplacement avec snapping, rotation/échelle via modificateurs clavier et
  lecture à 24 fps avec interpolation des propriétés animées ; 15 tests dont
  un test de pixels du rendu SVG
- Compositing de calques finalisé : clipping rectangulaire local, pivot de
  transformation et isolation alpha par calque, avec test pixel couvrant le
  clipping et l'ordre z
- Outils d'alignement du canvas : grille activable, règles graduées, guides
  verticaux/horizontaux configurables et snapping commutable sur grille ou
  guides ; test pixel ajouté pour le rendu des guides
- Masques de découpe dans `Animate.Core` et `Animate.Rendering` : formes
  rectangulaires ou elliptiques, inversion possible et application dans la
  matrice de transformation de chaque calque
- Calques texte SkiaSharp avec styles `SpeechBubble` et `TitleCard`, fond,
  couleurs, taille et retour à la ligne ; tests pixels ajoutés pour les
  masques et le rendu typographique
- Timeline custom : règle graduée, pistes par calque, marqueurs de keyframes,
  curseur, timecode et commandes lecture/arrêt ; scrub par clic et ajout de
  keyframes de position par double-clic
- Horloge `Animate.Timeline.TimelineClock` synchronisée avec le rendu Canvas,
  conversion frame/pixels et modèles sérialisables de marqueurs, clips audio
  et cycles d'animation ; 23 tests couvrent désormais la Phase 5 livrée
- Phase 3 complétée : rasterisation des sources SVG/bitmap en miniatures PNG
  mises en cache, réorganisation drag & drop des pistes de calques avec mise à
  jour des `ZIndex`, et clips déposables puis repositionnables sur la Timeline
- Phase 5 complétée : marqueurs de scènes ajoutables sur la règle, découpage
  par repères, import de fichiers audio avec waveform affichée et clips
  synchronisés, mapping lip-sync phonème → variante de bouche, et bibliothèque
  de cycles `Walk`, `Talk` et `Idle` glissables vers la Timeline
- Inspecteur contextuel de calques : édition du nom, position, rotation,
  échelle, opacité, visibilité et verrouillage ; overrides de couleurs SVG,
  ombre portée et contour épais reliés au modèle et au renderer

## [0.1.0] - 2026-09-04

### Ajouté

- Initialisation de la solution `.sln` et du projet `Animate.App`
  (WPF, .NET 8, WPF-UI)
- Projets squelettes des bibliothèques du domaine : `Animate.Core`,
  `Animate.Rendering` (SkiaSharp.Views.WPF), `Animate.Timeline`,
  `Animate.AssetLibrary`, `Animate.Export`, `Animate.Localization`,
  `Animate.Persistence`
- Projets de tests xUnit : `Animate.Core.Tests`, `Animate.Timeline.Tests`,
  `Animate.Export.Tests`
- Fenêtre principale `FluentWindow` (WPF-UI) avec persistance de la
  taille, de la position et de l'état (maximisé) entre deux sessions
  (`Animate.Persistence.WindowState`)
- Configuration `.editorconfig` stricte (nullable en erreur, analyzers,
  conventions de nommage C# imposées)
- Structure de dossiers cible du dépôt (`src/`, `assets/`, `ffmpeg/`,
  `i18n/`, `installer/`, `docs/img/`, `tests/`)
- `Directory.Build.props` avec métadonnées d'assembly communes
  (copyright, auteur, description, version `0.1.0`)
- `.gitignore` couvrant les documents internes non publiés, les
  artefacts de build et les fichiers sensibles
- `LICENSE` (Freeware, code source non fourni)
- `README.md` squelette (sections figées, captures d'écran en attente)
- `CHANGELOG.md` initialisé au format Keep a Changelog + SemVer
- `COMPILATION.md` (guide interne de compilation, non publié)
- `GITHUB_DEPOT.md` (checklist de préparation du dépôt public, non publié)
- Pipeline de versioning SemVer automatique via Nerdbank.GitVersioning
  (`version.json`), injectant `AssemblyVersion`/`FileVersion`/version
  informationnelle sur tous les projets de la solution ; exposé via
  `Animate.Core.AppInfo.AppVersionInfo` et affiché dans la fenêtre
  principale
- Script `scripts/release.ps1` (gitignored, outil de dev local) : build de
  release produisant dans `/dist` uniquement l'exécutable publié et les
  ressources de déploiement (i18n, assets, ffmpeg), avec vérification
  stricte de l'absence de tout fichier source

[Unreleased]: https://github.com/Patrickjaillet/animate/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/Patrickjaillet/animate/releases/tag/v1.0.0
[0.1.0]: https://github.com/Patrickjaillet/animate/releases/tag/v0.1.0
