# Revue de performance

Objectifs de performance d'Animate 1.0.0 et mesures associées.

## Rendu du canvas (SkiaSharp)

### Cible

- **60 fps** en lecture, soit un budget de **16,6 ms par frame** pour une scène
  représentative (plan chargé : ~24 calques, transformations, fondus d'opacité,
  rendu 1920 × 1080).

### Constat initial

Le profilage (`PerformanceTests.CanvasRenderStaysWithinA60FpsBudgetForATypicalScene`,
rasterisation logicielle plein écran) a mesuré **~183 ms/frame (~5 fps)** sur la
scène de référence.

### Cause

`SceneRenderer.RenderLayer` appelait `SKCanvas.SaveLayer` **pour chaque calque,
sans condition**, avec un rectangle d'isolation couvrant toute la surface. Chaque
`SaveLayer` alloue un tampon hors écran de la taille du clip et le recompose —
opération coûteuse répétée 24 fois par frame.

### Correction

`SaveLayer` n'est plus utilisé que lorsqu'un calque **translucide** superpose
**plusieurs primitives** (effets d'ombre / contour + contenu) devant fusionner
comme un groupe. Dans tous les autres cas :

- calque opaque → simple `Save`/`Restore` de la matrice ;
- calque translucide à primitive unique (SVG, bitmap, placeholder) → l'alpha est
  appliqué directement au tracé (`DrawPicture`/`DrawBitmap` avec un `SKPaint`
  porteur de l'opacité).

Le cache `SKPicture` des SVG (`SvgPictureCache`) évite tout re-parsing entre
frames — vérifié par `SvgPictureCacheAvoidsReparsingOnRepeatedFrames`
(≈ 0,7 ms pour une frame à cache chaud).

### Résultat

| | ms/frame | fps équivalent |
|---|---|---|
| Avant | ~183 | ~5 |
| Après | **~7** | **~145** |

Soit **~27× plus rapide**, largement sous le budget 60 fps même en rendu
logiciel. Le test garde un seuil volontairement large (50 ms, 3× le budget) pour
rester stable en environnement virtualisé.

## Mémoire de la bibliothèque d'assets

### Cible

L'index de la bibliothèque (`AssetLibraryIndex`) doit rester léger même avec une
bibliothèque fournie.

### Constat

`AssetLibraryIndexMemoryFootprintStaysSmall` mesure l'empreinte de l'index pour
**600 assets** répartis sur les 5 catégories : **~360 Ko** (budget : 2 Mo).

L'index ne conserve que des métadonnées (`AssetItem` : id, nom, catégorie,
chemin, extension, tags) — aucune image, vignette ou `SKBitmap` n'est chargé tant
qu'il n'est pas demandé (`AssetThumbnailCache` génère et met en cache les
vignettes à la volée sur disque).

## Méthode de mesure

Les budgets ci-dessus sont vérifiés par une suite de tests automatisés
(rasterisation logicielle plein écran, environnement virtualisé) exécutée à
chaque build de release. Trois mesures sont relevées : ms/frame sur la scène de
référence, efficacité du cache `SKPicture` des SVG, et empreinte mémoire de
l'index d'assets.
