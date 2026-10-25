<p align="right"><a href="README.md">한국어</a> · <a href="README.en.md">English</a> · <a href="README.ja.md">日本語</a> · <a href="README.zh.md">中文</a> · <b>Français</b> · <a href="README.es.md">Español</a></p>

# 🔍 Upscaly : agrandisseur d'images visant l'absence de perte (무손실 지향 확대기) `v1.2.2`

Upscaly est un outil d'agrandissement d'images qui vise l'absence de perte. Il tient dans un seul fichier (`upscaler.html`) et effectue tous les calculs dans le navigateur.
Rien à installer, rien à téléverser. Vos images ne quittent jamais votre appareil.

Version actuelle **v1.2.2** · les changements sont décrits dans [12. Historique des versions](#12-historique-des-versions). Les sources de développement, les tests et les données de recherche se trouvent dans le dépôt [Upscaly-dev](https://github.com/Velvitty/Upscaly-dev).

**Sommaire**

1. [Présentation simple](#1-présentation-simple)
2. [Principes](#2-principes)
3. [Fonctionnement](#3-fonctionnement)
4. [Algorithmes et formules](#4-algorithmes-et-formules)
5. [Le monde physique](#5-le-monde-physique)
6. [Architecture](#6-architecture)
7. [Problèmes et solutions](#7-problèmes-et-solutions)
8. [Optimisation](#8-optimisation)
9. [Validation](#9-validation)
10. [Points forts](#10-points-forts)
11. [Limites](#11-limites)
12. [Historique des versions](#12-historique-des-versions)
13. [Références](#13-références)

---

## 1. Présentation simple

### Qu'est-ce que cet outil ?

Quand on agrandit une petite image, elle devient en général floue, crénelée, ou un liseré blanc apparaît autour des contours. Upscaly agrandit chaque type d'image — photo, illustration, bande dessinée, document, logo, pixel art — avec la méthode qui lui convient, afin de réduire ces défauts tout en préservant au maximum l'information que contenait l'image d'origine.

« Viser l'absence de perte » signifie que la restauration est ajustée de sorte que l'image agrandie, une fois réduite à sa taille d'origine, redevienne identique à l'original. L'agrandissement n'invente pas d'image qui n'existait pas dans l'original ; il se concentre sur le déploiement le plus plausible de l'information que possède l'original. Les corrections ajoutées par-dessus, comme le renforcement des contours ou la suppression du crénelage, sont des étapes qui affinent cette restauration pour l'œil ; si vous les activez, le résultat s'écarte légèrement de l'information d'origine ([2.1](#21-agrandir-ne-crée-pas-dinformation)).

### Ses avantages

- **Rien à installer.** Il suffit d'ouvrir un fichier dans le navigateur. Aucune connexion Internet n'est nécessaire.
- **Vos images ne sortent pas.** Ni serveur ni bibliothèque externe.
- **Il s'adapte tout seul.** Le bouton 「Auto」(자동) identifie le type d'image parmi 9 catégories et choisit les réglages en mesurant directement l'image.
- **Il tient bon même en très grand.** L'image est calculée par tuiles, puis compressée et assemblée ligne de tuiles par ligne de tuiles ; même si le résultat compte des centaines de millions de pixels, le résultat complet non compressé n'est jamais chargé en mémoire. En revanche, le fichier PNG compressé s'accumule en mémoire jusqu'à la fin de l'enregistrement.
- **Des dégradés sans bandes.** À l'enregistrement en 8 bits, un tramage fin est ajouté pour que les ciels et dégradés fortement agrandis ne présentent pas de bandes en escalier. L'enregistrement en PNG 16 bits est aussi possible.
- **Les couleurs des photos de téléphone préservées.** Les photos Display P3 (iPhone, etc.) sont lues en P3 et enregistrées en P3, afin que les rouges et verts vifs ne deviennent pas ternes.
- **Restauration de texture (optionnelle).** Sur une peau ou un tissu devenu lisse comme de la cire après un fort agrandissement, un grain fin peut être ajouté de nouveau. Elle est désactivée par défaut.
- **Les images transparentes restent propres.** Les couleurs ne bavent pas sur les bords semi-transparents.
- **Deux objectifs : précision et perception.** Par défaut, 「Précision」(정확도) agrandit au plus près de l'original. 「Perception」(체감) rend les contours plus raides et fait ressortir les contrastes enfouis, pour une image plus nette à l'œil. Aucun liseré blanc (halo) n'apparaît.

### Mode d'emploi

1. Ouvrez `upscaler.html` dans un navigateur récent.
2. Cliquez sur **Ouvrir une image** (이미지 열기), glissez-déposez une image sur l'écran ou collez-la. Vous pouvez ouvrir jusqu'à 20 images à la fois, en JPEG, PNG, WebP ou AVIF.
3. Choisissez le **facteur** (배율). Il peut être défini sur la largeur et la hauteur (1 à 12×) ou sur le nombre de pixels (surface) ; si vous choisissez 「Auto」, le plus grand facteur (3 à 8×) que cet appareil peut traiter jusqu'au bout est sélectionné.
4. Cliquez sur **「Auto : (conseillé, recommandé)」(자동: (권장, 추천))**. Le type et l'état de l'image sont mesurés, tous les réglages ci-dessous sont remplis, et une courte note indique sur quoi reposent ces choix. Si paraître net compte plus que rester fidèle à l'original, choisissez 「Perception」 dans **Objectif** (목표).
5. Cliquez sur **Lancer l'agrandissement** (확대 실행). Si **Comparer à l'original** (원본 비교) est activé, la moitié gauche de l'écran montre l'original agrandi par le navigateur, la moitié droite le résultat. Faites glisser pour vous déplacer, utilisez la molette pour zoomer, ainsi que les boutons **Ajuster** (맞춤, touche F), **1:1** et **Comparer à l'original** (touche C).
6. **Enregistrer en PNG** (PNG 저장) enregistre l'image actuelle, **Tout enregistrer** (모두 저장) enregistre d'un coup toutes les images traitées.

### Choisir les réglages soi-même

Les panneaux à droite de l'écran sont disposés dans l'ordre du traitement.

| Panneau | Rôle |
|---|---|
| Facteur (배율) | De combien agrandir. 1× ne fait qu'un nettoyage, sans agrandissement |
| Préréglage · Auto (프리셋 · 자동) | 33 préréglages ou 「Auto」 |
| Objectif (목표) | 「Précision」 (au plus près de l'original, par défaut) ou 「Perception」 (augmente la pente des contours et le contraste local pour paraître plus net, intensité 25 à 150 %). Les préréglages et 「Auto」 ne le modifient pas |
| Noyau de reconstruction (재구성 커널) | Comment remplir les nouveaux pixels (DCCI, pixel art, Lanczos, Catmull-Rom, Mitchell, plus proche voisin), agrandissement par étapes, blocage du ringing, cohérence avec l'original |
| Nettoyage du bruit et des artefacts (노이즈·잡티 정리) | Retire le bruit et les traces JPEG avant l'agrandissement |
| Restauration de la netteté (선명도 복원) | Annule avant l'agrandissement le flou introduit à la prise de vue ou à la numérisation |
| Suppression du crénelage (계단 제거) | Supprime les marches en moyennant uniquement le long des contours, retracé des contours, atténuation des blocs JPEG |
| Renforcement des contours (윤곽 강조) | Masque flou (unsharp mask) avec suppression des halos |
| Restauration de texture (질감 되살리기) | Ajoute un grain fin absent de l'original. Désactivée par défaut, 20 à 30 % conseillés pour les photos et peintures |
| Traitement des couleurs (색 처리) | Balance des blancs (fonction séparée, que les préréglages et 「Auto」 ne touchent pas), calcul en lumière linéaire, tramage 8 bits, enregistrement 16 bits |

Les cases à cocher et curseurs font exactement ce que vous choisissez. Par exemple, 「Agrandir par étapes」(단계로 나눠 확대), une fois activé, découpe l'agrandissement même si c'est moins bon selon la mesure. En contrepartie, le voyant à côté indique le résultat mesuré sur l'image actuelle. Juste après l'ouverture d'une image, la valeur initiale de cette case est déterminée en mesurant l'image ; si vous modifiez la case ou cliquez sur 「Auto」 avant la fin de cette mesure, la mesure terminée en retard ne touche pas à votre choix.

### Bon à savoir

- Une image déjà réduite et réenregistrée plusieurs fois a perdu ses détails d'origine, qui ne peuvent pas être récupérés. Upscaly ne fait que déployer de la manière la plus plausible l'information restante ; il n'invente ni visages ni lettres absents.
- Pour un logo ou un plan dont il existe un original vectoriel (SVG, AI), il vaut toujours mieux utiliser cet original.
- À 8×, le résultat atteint plusieurs centaines de mégapixels et le traitement prend longtemps. L'indicateur de progression affiche le temps restant.

---

## 2. Principes

### 2.1 Agrandir ne crée pas d'information

Une image numérique est un échantillonnage, à intervalles réguliers, d'une scène continue. Les motifs plus fins que le pas d'échantillonnage n'ont jamais été enregistrés. L'essence de l'agrandissement est donc « comment combler l'espace entre les échantillons », et selon la manière de le faire apparaissent des défauts comme le flou, le crénelage ou le ringing.

Pour choisir la manière de combler, Upscaly suit trois principes. Les deux premiers sont des contraintes de l'étape de restauration (interpolation → contrainte de plage par cellule → cohérence) et ne valent que si les cases correspondantes (「Bloquer le ringing et les halos」(링잉·후광 차단), 「Correction de cohérence avec l'original」(원본 정합 보정)) sont activées.

1. **Ne pas créer de nouveaux extrema.** Un pixel restauré se situe entre la valeur la plus sombre et la plus claire de la cellule 2×2 de l'original à laquelle il appartient (contrainte de plage par cellule). Le ringing et les halos sont des extrema absents de l'original ; ils n'apparaissent donc pas à cette étape.
2. **Réduit de nouveau, on retrouve l'original.** L'image restaurée est ajustée de sorte que sa réduction par moyenne de surface de la lumière redonne l'original (correction de cohérence avec l'original). Autrement dit, elle ne contredit pas l'information de l'original.
3. **Les contours sont comblés en suivant leur direction.** Mélanger en travers d'un contour le rend flou, mélanger le long des axes crée des marches. La direction dans laquelle s'étend le contour est lue, et le comblement se fait uniquement dans cette direction (DCCI).

Les deux contraintes (1 et 2) sont difficiles à satisfaire simultanément : en satisfaire une fait légèrement dévier l'autre. On utilise donc des projections alternées, qui les satisfont tour à tour ([4.6](#46-projections-alternées)).

Le masque flou, le lissage directionnel, le retracé des contours et la restauration de texture qui suivent la restauration sont des étapes d'affinage pour l'œil et ne respectent plus ces deux contraintes. Le résultat final avec ces étapes activées peut donc légèrement dépasser la plage des cellules de l'original, ou différer un peu de l'original une fois réduit. L'« erreur de cohérence de l'agrandissement » affichée à l'écran est mesurée sur les valeurs avant ces corrections. Les préréglages comme 「Intervention minimale · Lanczos pur」(최소 개입 · 순수 Lanczos) désactivent les deux contraintes.

### 2.2 La lumière se mélange en lumière linéaire

Les valeurs stockées pour l'écran (sRGB) sont courbées pour correspondre à l'œil humain. Mélanger la lumière de deux pixels doit se faire non pas sur ces valeurs courbées, mais sur la quantité réelle de lumière (lumière linéaire). Interpoler directement les valeurs courbées assombrit et empâte la frontière entre zones claires et sombres. Par défaut, Upscaly calcule en lumière linéaire et ne revient en sRGB qu'à l'enregistrement. Si vous décochez 「Calculer en lumière linéaire」(선형광에서 계산), le calcul se fait directement dans l'espace gamma (déconseillé).

Les images transparentes sont calculées sur des valeurs où la couleur est multipliée par l'alpha (prémultiplication). Et lors de la préparation de l'original, des étapes intermédiaires de l'agrandissement par étapes comme du calcul final des tuiles, **l'alpha subit exactement les mêmes opérations que la couleur.** Si la couleur et l'alpha étaient filtrés différemment, la couleur des bords semi-transparents serait faussée au moment de diviser la couleur par l'alpha.

### 2.3 Le flou et le bruit se traitent avant l'agrandissement, à la résolution d'origine

Les objectifs d'appareil photo et les scanners captent la scène légèrement floue, et les capteurs et la compression ajoutent du bruit. Ces défauts sont nés à la résolution d'origine ; c'est donc à la résolution d'origine qu'il faut les corriger. Après l'agrandissement, les défauts ont grandi eux aussi, ce qui rend le traitement plus difficile et plus lent.

- **Le flou** est estimé en sens inverse par déconvolution de Richardson–Lucy. C'est de là que vient, pour l'essentiel, le fait que l'image agrandie paraisse plus nette que l'original.
- **Le bruit** est supprimé en se référant au spectre de bruit mesuré directement sur l'image. Les composantes plus grandes que le bruit (contours, texture) sont conservées ; seules celles qu'on ne peut distinguer du bruit sont réduites.
- **Les traces JPEG** sont supprimées en lisant la table de quantification du fichier. Les coefficients DCT de la luminance sont alors ramenés dans les intervalles de quantification du fichier d'origine, ce qui évite d'effacer les détails enregistrés dans le fichier. Cette garantie ne concerne que la luminance de cette étape de projection. Les composantes de couleur, ainsi que l'atténuation des blocs, le nettoyage du bruit, la déconvolution, l'agrandissement et l'accentuation qui suivent, peuvent déplacer les coefficients hors des intervalles.

### 2.4 Les traits sont redessinés

Dans les graphismes où deux couleurs se séparent nettement (logos, dessins au trait, texte), les marches de l'original risquent d'être conservées comme si elles étaient la « bonne réponse ». Upscaly repère ces frontières et les redessine à la résolution agrandie sous forme de contours d'un pixel de large. L'isoligne à 50 % des valeurs interpolées lisses est prise comme position du contour, et le profil est redressé selon cette distance (méthode de la distance signée). Le lissage directionnel redresse la forme du contour et le retracé redresse le profil ; utilisés ensemble, ils donnent des traits lisses, sans marches et sans bavure.

### 2.5 Une texture inventée n'est pas une texture, mais du grain

L'agrandissement ne peut pas créer de motifs plus fins que le pas des pixels de l'original. C'est pourquoi, sur une photo fortement agrandie, le grain de la peau, les fils d'un tissu ou les nervures des feuilles disparaissent et l'image paraît lisse comme de la cire. La **quantité** de texture perdue peut être estimée : dans les images naturelles, l'énergie contenue est à peu près la même à chaque doublement de la fréquence spatiale (spectre de puissance d'environ $`1/f^2`$). À partir de l'intensité de la texture mesurée dans la bande fine de l'original, on peut estimer combien de texture contenaient les octaves manquantes.

Mais **l'emplacement et la forme** de la texture ne figurent pas dans l'original. Ce qu'Upscaly ajoute est un bruit dosé selon cette quantité, c'est-à-dire un grain fin comparable au grain d'une pellicule. Comme il ne coïncide pas en position avec la vérité, plus on remplit l'énergie jusqu'au niveau réel, plus l'écart à l'original (PSNR) grandit, et plus l'œil y voit du bruit plutôt qu'une texture. La restauration de texture est donc une fonction optionnelle que 「Auto」 ne touche pas, et il est conseillé de l'utiliser à faible intensité sur les surfaces lisses comme les portraits, la peau ou les tissus. Dessiner de manière plausible jusqu'à la forme de la texture nécessite un modèle génératif (réseau de neurones) ([11. Limites](#11-limites)).

### 2.6 Les marches du 8 bits se masquent par un bruit fin

Le traitement se fait en virgule flottante, mais le PNG 8 bits qu'on enregistre habituellement n'écrit la luminosité qu'en 256 niveaux. En agrandissant 8 fois un ciel lisse, une luminosité qui changeait d'un niveau tous les quelques pixels dans l'original devient des bandes de plusieurs dizaines de pixels de large. Si l'on ajoute juste avant l'arrondi un bruit inférieur à un demi-niveau (tramage), les limites des bandes se dispersent en un grain fin invisible à l'œil. Ce bruit est un motif déterminé uniquement par les coordonnées de l'image ; il est donc identique d'une tuile à l'autre, et appliqué de la même façon aux trois canaux, le gris reste gris. Les valeurs déjà exactes, comme celles d'un aplat, ne changent pas.

### 2.7 Les couleurs des photos de téléphone dépassent le sRGB

Les photos de téléphone récentes sont souvent prises dans le gamut Display P3, plus large que le sRGB. Lues sur un canevas ordinaire (sRGB), les rouges et verts vifs hors du sRGB sont écrasés par le navigateur dans le sRGB, et le résultat agrandi devient plus terne que l'original. Upscaly lit le profil colorimétrique du fichier d'origine ; si le gamut est large, il lit l'image sur un canevas P3 et enregistre aussi le résultat avec un profil P3. Display P3 utilise la même courbe de transfert que le sRGB : le calcul en lumière linéaire reste inchangé, seules les primaires diffèrent.

### 2.8 Chaque image a sa propre réponse

Les réglages adaptés aux photos abîment les dessins au trait, et ceux adaptés aux dessins au trait abîment les photos. La cohérence avec l'original est bénéfique pour les photos, mais pour un dessin au trait où deux couleurs se séparent nettement, elle restitue les marches de l'original comme bonne réponse. 「Auto」 commence donc par identifier le type d'image (classifieur à 9 classes), part du préréglage adapté à ce type, puis ajuste les détails à partir de valeurs mesurées sur l'image. Pour les forts agrandissements, il utilise des valeurs trouvées en explorant un espace de 1 520 milliards de réglages ([9.4](#94-recherche-de-restauration-poussée)).

### 2.9 Précision et perception sont deux objectifs différents

Le résultat le plus proche de l'original (celui dont le PSNR est le plus élevé) n'est pas celui qui paraît le plus net. L'agrandissement émousse même les contours que la résolution d'origine peut contenir (pente des contours 0,81× la vérité terrain) et affaiblit le contraste à l'échelle de quelques pixels d'origine (0,94×). L'œil perçoit les deux comme de la netteté. Il y a donc deux objectifs. 「Précision」 est la restauration habituelle ; 「Perception」 ramène, par-dessus, ces deux grandeurs au niveau de la vérité terrain. Elle ne crée aucun nouvel extremum, donc aucun liseré blanc, et ne dessine pas de texture absente de l'original. En contrepartie, les mesures par rapport à l'original baissent légèrement ([4.21](#421-netteté-perçue), [9.10](#910-netteté-perçue)).

---

## 3. Fonctionnement

### 3.1 Vue d'ensemble

```
Fichier ─▶ Décodage (orientation EXIF) ─▶ Profil colorimétrique (canevas P3 si gamut large) ─▶ En-tête JPEG (table de quantification, échantillonnage) ─▶ Cadre du sujet
                                              │
              「Auto」 : image d'analyse ─▶ Classification 9 classes ─▶ Préréglage + ajustements mesurés ─▶ Valeurs de la recherche de restauration poussée / mesure du lissage directionnel / mesure de l'agrandissement par étapes
                                              │
Préparation de l'original (résolution d'origine) : plans en lumière linéaire (prémultipliés) ─▶ Traitement des couleurs ─▶ Restauration DCT fine ─▶ Atténuation des blocs ─▶ Nettoyage du bruit ─▶ Déconvolution RL ─▶ Étapes intermédiaires de l'agrandissement par étapes
                                              │
Calcul des tuiles (pool de workers) : Interpolation ─▶ Plage par cellule ─▶ Projections alternées cohérence↔plage par cellule ─▶ Statistiques ─▶ Masque flou ─▶ Lissage directionnel ─▶ Retracé des contours ─▶ (Texture) ─▶ Annulation de la prémultiplication ─▶ Encodage (tramage 8 bits)
                                              │
                         Cache de tuiles (LRU) ─▶ Écran / enregistrement PNG par lignes de tuiles (8·16 bits, sRGB ou P3)
```

### 3.2 Chargement

Le fichier est d'abord décodé avec `createImageBitmap` (orientation EXIF appliquée) ; en cas d'échec, on se replie sur une Blob URL dans un `<img>`, puis sur une Data URL. Les fichiers à gamut large (section 2.7) sont détectés avant le décodage et passent d'abord par le chemin `<img>` (Blob URL, puis Data URL en cas de blocage), le repli sur `createImageBitmap` n'intervenant que si les deux échouent. En effet, la conversion de couleurs par défaut de `createImageBitmap` diffère selon les navigateurs et peut tronquer en sRGB. Comme les navigateurs passent parfois réellement plusieurs dizaines de secondes sur les gros fichiers, le délai d'attente est proportionnel à la taille du fichier (45 s + 9 s par Mo, 5 min au maximum). Aucun chemin ne reste bloqué indéfiniment en attente.

Pour un JPEG, seul l'en-tête est lu afin d'obtenir la table de quantification, le sous-échantillonnage (4:2:0, etc.), une estimation de la qualité et l'orientation EXIF. Les données entropiques ne sont pas décodées. On vérifie à partir de la largeur et de la hauteur du résultat décodé si le décodeur a appliqué l'orientation EXIF, et la grille JPEG 8×8 est réalignée sur l'orientation d'affichage.

Le profil colorimétrique est aussi recherché dans le fichier (en suivant les en-têtes de segments, de chunks et de boîtes et en ne lisant que les parties nécessaires, il est trouvé même s'il est précédé de grosses métadonnées ou de boîtes de données d'image). Pour le JPEG, ce sont les segments APP2 `ICC_PROFILE` ; pour le PNG, les chunks cICP, iCCP et sRGB (dans cet ordre de priorité, quel que soit leur ordre dans le fichier) ; pour le WebP, le chunk ICCP ; pour l'AVIF et le HEIF, la boîte colr rattachée à l'image principale. Si le gamut est plus large que le sRGB, le canevas de l'original est créé en `display-p3`, l'image y est dessinée, et « Display P3 » est indiqué sur la ligne d'informations. Si le navigateur ne prend pas en charge le canevas P3, l'image est lue en sRGB comme auparavant.

Pour chaque image, l'original est réduit à environ 360 px et les amas de saillance (distance à la couleur moyenne) sont recherchés pour construire le cadre du sujet. Il peut être affiché avec le bouton 「Sujet」(피사체).

### 3.3 Préparation de l'original (`prepareSource`)

Toutes les corrections qui se terminent à la résolution d'origine se font ici. Les résultats sont mis en cache, avec la clé des réglages, pour les deux plus récents ; ainsi, même si le résultat agrandi est évincé du cache, il est réutilisé tant que les réglages sont identiques.

1. **Plans en lumière linéaire.** Le sRGB 8 bits est converti en lumière linéaire par une table de 256 entrées, et si l'image a un alpha, la couleur est multipliée par l'alpha.
2. **Traitement des couleurs.** Uniquement si l'intensité de la balance des blancs n'est pas nulle (0 par défaut).
3. **Restauration DCT fine.** L'original est nettoyé à l'aide du spectre de bruit et de la table de quantification JPEG. Le travail est découpé en bandes confiées aux workers, et la grille DCT est fixée en coordonnées globales, si bien qu'il n'y a pas de jointure aux limites des bandes.
4. **Atténuation des blocs JPEG.** Les sauts aux positions multiples de 8 ne sont aplanis que s'ils sont plus faibles que les variations environnantes.
5. **Nettoyage du bruit.** Seuls les voisins de luminosité proche sont moyennés (filtre bilatéral). L'alpha est moyenné avec les mêmes poids.
6. **Déconvolution RL.** L'alpha la subit aussi, avec le même nombre d'itérations.
7. **Étapes intermédiaires de l'agrandissement par étapes.** Si un grand facteur est découpé en deux étapes ou plus, l'image est agrandie ici jusqu'à la dernière étape. Avec DCCI, les étapes intermédiaires utilisent aussi l'interpolation directionnelle ; pour le pixel art, les contours sont restaurés à chaque étape. L'alpha subit les mêmes poids directionnels, la même plage par cellule et la même accentuation entre les étapes que la couleur. La référence de la plage par cellule et de la cohérence de la dernière étape n'est pas l'image intermédiaire mais **l'original**.

### 3.4 Calcul des tuiles (`renderTile`)

Le résultat est calculé par tuiles (512 à 1024 px selon le facteur ; pour un facteur entier, un multiple de ce facteur). Chaque tuile reçoit le fragment d'original dont elle a besoin (support d'interpolation + marge de cohérence + marge du champ de directions et du modèle de contours) et est calculée indépendamment ; le résultat est donc le même quel que soit l'ordre de calcul des tuiles.

1. **Interpolation.** Avec DCCI, tous les canaux sont agrandis en une seule fois (`dcciStageMulti`). Les poids directionnels sont calculés une seule fois sur un plan guide (luminance gamma + terme de contour alpha) et partagés entre les canaux. Catmull-Rom ajuste ensuite au facteur exact.
2. **Contrainte de plage par cellule.** Les valeurs sont écrêtées au minimum et au maximum par canal de la cellule 2×2 de l'original.
3. **Projections alternées entre correction de cohérence et plage par cellule.** Deux projections alternées sont effectuées (une seule correction de cohérence si la plage par cellule est désactivée). Chaque passe est résolue sur une zone 7 pixels plus large que la zone de résultat, de sorte qu'aucune passe ne lise hors de la grille des coefficients.
4. **Statistiques.** Pour afficher à l'écran l'écart à l'original du résultat réduit (erreur de cohérence de l'agrandissement), la mesure est faite sur les valeurs avant correction.
5. **Masque flou.** Les halos sont supprimés par un limiteur à coude proportionnel à l'amplitude locale.
6. **Lissage directionnel.** Moyenne uniquement dans la direction tangente trouvée par le tenseur de structure de l'original.
7. **Retracé des contours.** Les contours d'un pixel de large sont redessinés par distance signée.
8. **Netteté perçue.** Uniquement quand l'objectif est 「Perception」, augmente la pente des contours et le contraste local dans la luminance, sans jamais dépasser la plage de luminosité du voisinage ([4.21](#421-netteté-perçue)).
9. **Restauration de texture.** Uniquement si elle est activée : la texture fine manquant au résultat est ajoutée selon l'intensité de texture mesurée sur l'original. Avec l'agrandissement par étapes, la mesure se fait sur l'original et non sur l'image intermédiaire. Elle n'est pas utilisée avec le plus proche voisin, qui reproduit l'original tel quel, ni avec le noyau pixel art, qui préserve la palette ; le curseur est alors désactivé.
10. **Annulation de la prémultiplication et encodage.** Conversion en 8 bits pour l'écran et en 8 ou 16 bits pour l'enregistrement. En 8 bits, si le tramage est activé (par défaut), un bruit fin déterminé par les coordonnées est ajouté juste avant l'arrondi. L'écran et l'enregistrement 8 bits utilisent les mêmes tuiles : ce que vous voyez est ce qui est enregistré.

Le pool de workers lance nombre de cœurs − 1 workers (8 au maximum) ; s'ils ne répondent pas dans les 5 secondes, le calcul se fait sur le thread principal avec le même moteur. Le moteur (`ENGINE_FACTORY`) est un ensemble de fonctions de calcul pures ; le même code est donc chargé aussi dans un worker via une Blob URL.

### 3.5 Agrandissement par étapes

Cette méthode agrandit un grand facteur en plusieurs fois. On choisit parmi : 2× d'abord, grand facteur d'abord, équilibré (petit ou grand d'abord), égal (√×√) et trois étapes. Une accentuation peut être appliquée entre les étapes. Le voyant réduit le centre de l'image actuelle du facteur cible par moyenne de surface de la lumière, puis compare à l'original (PSNR) l'agrandissement en une fois et l'agrandissement par étapes selon la méthode actuelle, et indique si l'état actuel de la case est le meilleur. Le résultat est mémorisé par image et par réglages.

### 3.6 Auto

1. **Analyse.** Si le grand côté dépasse 1024, l'image est réduite de moitié par moyennes 2×2, puis par moyenne de boîte d'un facteur entier jusqu'à 512 ou moins ; 27 caractéristiques sont mesurées sur cette image d'analyse : tenseur de structure (proportion de texture, contours alignés sur les axes), contours forts et lignes de contour, période des lignes de texte, longues lignes droites, composition des couleurs et de la luminosité, résidu de planéité des blocs 8×8 sur un fragment à la résolution d'origine, grille d'agrandissement au plus proche voisin, nombre de couleurs de l'original, trames, etc.
2. **Classification.** Une régression logistique multinomiale choisit l'une des classes : photo, illustration, peinture, bande dessinée, dessin au trait, document, UI, logo, pixel art. Le pixel art n'est pas décidé par une probabilité mais par la structure (grille au plus proche voisin d'un facteur entier, ou petite palette à 1×). Les éléments qui ont fait pencher la décision (contribution de chaque caractéristique à l'écart de logit entre la 1re et la 2e classe) sont indiqués à l'écran.
3. **Préréglage et ajustements mesurés.** En partant du préréglage adapté au type, le nombre d'itérations et le σ de la déconvolution, la restauration JPEG, l'atténuation des blocs, etc., sont ajustés selon le bruit (médiane du laplacien sur les zones plates), la largeur des contours, le contraste de la grille de 8 px et la qualité JPEG.
4. **Valeurs de la recherche de restauration poussée.** Si le type et le facteur satisfont les conditions de validation, les réglages trouvés par la recherche sont utilisés tels quels (`AUTO_BIG`). Pour les photos, dessins au trait, UI et peintures, à partir de 2× ; pour les documents, à partir de 4×. Pour les logos et bandes dessinées, à partir de 2× et uniquement s'il s'agit d'un fichier JPEG, car ils n'ont pas passé la validation sur des entrées propres ([9.4](#94-recherche-de-restauration-poussée)). Les illustrations ne les utilisent pas.
5. **Mesures par image.** Quand les valeurs de recherche ne sont pas utilisées, pour les images dessinées (hors photos et pixel art), l'intensité du lissage directionnel est choisie en la mesurant sur l'image (`autoAA`), puis toutes les méthodes d'agrandissement par étapes sont mesurées et la meilleure est retenue (`autoStage`).

Les requêtes automatiques sont numérotées, de sorte qu'une mesure antérieure terminée en retard ne s'applique pas à une image nouvellement ouverte. Si vous changez le facteur après 「Auto」, un message vous invite à cliquer de nouveau.

### 3.7 Facteur automatique

Le plus grand facteur entre 3 et 8× dont le rendu est **garanti** est choisi. Appuyer sur 「Auto」 dans la ligne du facteur déplace aussitôt le facteur vers le plus grand garanti avec les réglages actuels. Garanti signifie que la mémoire maximale sur l'ensemble du traitement reste sous 80 % du budget mémoire, que la marge restante dépasse le cache minimal et que le résultat respecte la limite d'enregistrement. Sont aussi comptés le PNG compressé qui s'accumule en mémoire pendant l'enregistrement (taux de compression maximal mesuré sur 47 images du jeu de données, plus une marge de 10 %) et les originaux préparés des images précédentes restés dans le cache. Comme les réglages choisis par 「Auto」 (préréglage, valeurs de la recherche de restauration poussée, agrandissement par étapes) diffèrent selon le facteur, le budget diffère aussi ; 「Auto」 applique donc réellement, en partant des grands facteurs, les réglages de chaque facteur et choisit le premier facteur garanti. Lorsque l'agrandissement par étapes est déterminé par mesure sur la photo, seules les méthodes qui restent dans la garantie sont retenues. Une fois le choix fait, la limite du cache de tuiles est effectivement abaissée à cette valeur. Si même 3× n'est pas garanti, on se replie sur le plus grand facteur garanti inférieur.

### 3.8 Enregistrement

Si le navigateur prend en charge `CompressionStream('deflate')`, chaque ligne de tuiles (bande de 128 lignes en 16 bits) est compressée et les fragments PNG sont assemblés à la suite. En 8 bits, les tuiles en cache sont réutilisées ; en 16 bits, chaque bande est recalculée. Aucun grand canevas contenant tout le résultat n'est créé, mais les fragments compressés sont accumulés en mémoire jusqu'à la fin de l'enregistrement, puis exportés en un seul fichier. Le fichier contient un chunk d'espace colorimétrique et un chunk iTXt `Software` (UTF-8) indiquant le noyau et le facteur. Le chunk d'espace colorimétrique est normalement sRGB ; pour les images lues en P3, c'est un cICP (primaires 12 = Display P3, transfert 13 = courbe sRGB) accompagné d'un profil ICC Display P3 de même signification (iCCP). Les programmes qui lisent cICP l'utilisent, les autres utilisent l'ICC. Les navigateurs sans compression en flux ne peuvent enregistrer qu'en 8 bits : le résultat est rassemblé sur un seul canevas et enregistré avec l'encodeur du navigateur, et si la limite du canevas est dépassée, la limite réelle de l'appareil est mémorisée. Sur ces navigateurs, choisir l'enregistrement 16 bits fait échouer l'enregistrement.

### 3.9 Traitement des couleurs

Le traitement des couleurs est une fonction séparée de l'agrandissement, avec une valeur par défaut de 0. 「Couleurs auto」(색 자동) estime la couleur de l'éclairage par trois méthodes (moyenne, contours, zones claires) et fixe l'intensité de la balance des blancs à hauteur de ce que les trois admettent. Une véritable dominante d'éclairage teinte jusqu'aux zones claires, et les trois estimations grandissent ensemble, alors qu'une couleur propre au sujet, comme un chat roux ou un coucher de soleil, n'attire que la moyenne. Les couleurs des illustrations, peintures, UI, logos et pixel art sont considérées comme voulues et ne sont pas corrigées. La luminosité globale est conservée et les gains par canal sont limités à 0,55–1,8.

---

## 4. Algorithmes et formules

Dans ce qui suit, $`I`$ désigne l'original (lumière linéaire, prémultiplié), $`U`$ le résultat, et $`s_x, s_y`$ les facteurs horizontal et vertical.

### 4.1 Espace colorimétrique

```math
\mathrm{lin}(u) =
\begin{cases}
u / 12.92 & u \le 0.04045 \\[2pt]
\left(\dfrac{u + 0.055}{1.055}\right)^{2.4} & \text{sinon}
\end{cases}
\qquad
\mathrm{srgb}(v) =
\begin{cases}
12.92\,v & v \le 0.0031308 \\[2pt]
1.055\,v^{1/2.4} - 0.055 & \text{sinon}
\end{cases}
```

La conversion 8 bits → lumière linéaire utilise une table de 256 entrées, et la conversion lumière linéaire → 8 bits une table de 8 192 entrées (erreur de quantification inférieure à un niveau). L'encodeur 16 bits construit sa table sur l'axe $`\sqrt{v}`$, ce qui le maintient à moins d'un niveau même dans la zone abrupte proche du noir. Les 256 niveaux 8 bits convertis en lumière linéaire puis reconvertis restent tous identiques ([9.1](#91-invariants)).

En présence d'alpha, le calcul se fait sur les valeurs prémultipliées $`(\alpha R, \alpha G, \alpha B, \alpha)`$, et juste avant l'encodage on divise par $`\alpha`$ si $`\alpha > 10^{-7}`$.

### 4.2 Noyaux de reconstruction

Ils sont appliqués de façon séparable (horizontal → vertical), en alignant les centres des pixels ($`u = (X + 0.5)/s - 0.5`$). En réduction, le noyau est étiré du facteur pour éviter le repliement.

```math
L_a(x) = \mathrm{sinc}(x)\,\mathrm{sinc}(x/a)\ \ (|x| < a),\qquad
k_{B,C}(x) = \tfrac16
\begin{cases}
(12 - 9B - 6C)|x|^3 + (-18 + 12B + 6C)|x|^2 + (6 - 2B) & |x| < 1 \\
(-B - 6C)|x|^3 + (6B + 30C)|x|^2 + (-12B - 48C)|x| + (8B + 24C) & 1 \le |x| < 2
\end{cases}
```

Lanczos-3 et 4 correspondent à $`a = 3, 4`$, Catmull-Rom à $`(B, C) = (0, 0.5)`$ et Mitchell–Netravali à $`(1/3, 1/3)`$. Le plus proche voisin reproduit l'original tel quel pour les facteurs entiers ; réduit de nouveau, il redonne l'original au bit près.

### 4.3 DCCI adaptative aux contours

Sur la base de l'interpolation par convolution cubique directionnelle de Zhou et al. (2012), l'image est agrandie de 2× par étape (alignement des coins : sortie $`(2i, 2j)`$ = entrée $`(i, j)`$).

**Étape 1 : centre des cellules 2×2.** On mesure la variation selon les deux directions diagonales dans un voisinage 4×4. $`G`$ est le plan guide (luminance gamma 0–255) et $`n_{\mathrm{off}}`$ le plancher de bruit.

```math
d_1 = \sum_{(p,q) \in \nearrow} |G_p - G_q| + n_{\mathrm{off}},\qquad
d_2 = \sum_{(p,q) \in \searrow} |G_p - G_q| + n_{\mathrm{off}}
```

Chaque somme porte sur les 9 paires voisines dans la direction diagonale correspondante. Les poids sont les suivants ($`T = 1.15`$).

```math
(w_a, w_b) =
\begin{cases}
(0, 1) & 1 + d_1 > T(1 + d_2) \\
(1, 0) & 1 + d_2 > T(1 + d_1) \\
\left(\dfrac{a}{a+b}, \dfrac{b}{a+b}\right),\ a = \dfrac{1}{1 + d_1^{5}},\ b = \dfrac{1}{1 + d_2^{5}} & \text{sinon}
\end{cases}
```

Si la variation selon $`\nearrow`$ est forte, le contour s'étend dans la direction $`\searrow`$, et l'interpolation se fait donc dans cette direction. La valeur dans chaque direction est donnée par les coefficients de convolution cubique $`(-1, 9, 9, -1)/16`$.

```math
p_a = \tfrac{1}{16}\big(9(f_{12} + f_{21}) - (f_{03} + f_{30})\big),\quad
p_b = \tfrac{1}{16}\big(9(f_{11} + f_{22}) - (f_{00} + f_{33})\big),\quad
v = \mathrm{clip}\big(w_a p_a + w_b p_b,\ \min_4 f,\ \max_4 f\big)
```

Le `clip` final est un ajout absent de l'article original. Il empêche la nouvelle valeur de sortir de la plage des 4 voisins les plus proches, si bien que ni ringing ni halo n'apparaissent dès cette étape.

**Étape 2 : pixels entre les lignes et les colonnes.** La même procédure est appliquée dans les directions horizontale et verticale sur la grille tournée de 45°. Les bords sont repliés par réflexion miroir pour préserver la parité.

**Nombre d'étapes.** Une table mesurée et figée pour chaque facteur (`EDI_PLAN`) est utilisée. Jusqu'à 2×, une étape sur une grille 2× ; au-delà, l'image est agrandie jusqu'à une grille 4× (deux étapes), puis Catmull-Rom ajuste au facteur exact ([8. Optimisation](#8-optimisation)).

### 4.4 Contrainte de plage par cellule

L'échantillon de sortie $`U(X, Y)`$ est écrêté à la plage, par canal, de la cellule de l'original $`(i..i{+}1,\ j..j{+}1)`$ à laquelle il appartient.

```math
U(X, Y) \leftarrow \mathrm{clip}\Big(U(X, Y),\ \min_{(p,q) \in \mathrm{cell}} I_{p,q},\ \max_{(p,q) \in \mathrm{cell}} I_{p,q}\Big)
```

L'agrandissement au plus proche voisin reste toujours dans cette plage ; une solution existe donc. Les minima et maxima par cellule ne sont construits qu'une fois par plan de canal (cache `cellRange`).

### 4.5 Correction de cohérence avec l'original

Si $`D`$ désigne l'opération qui réduit le résultat par moyenne de surface, l'objectif est $`D(U) = I`$. À l'estimation initiale $`U_0`$, on ajoute des coefficients $`c`$ à la résolution d'origine, déployés par l'interpolation bilinéaire $`P`$.

```math
U = U_0 + P(c),\qquad D(U_0 + P c) = I \;\Longrightarrow\; K c = I - D(U_0),\quad K = D P
```

Comme $`D`$ et $`P`$ sont tous deux séparables par axe, $`K = K_x \otimes K_y`$, et le $`K`$ de chaque axe est une matrice tridiagonale. $`c`$ est donc résolu exactement à la résolution d'origine par l'algorithme de Thomas. Les coefficients de chaque axe sont construits à partir de l'empreinte des pixels de sortie dans l'original (recouvrements fractionnaires compris). L'atténuation de l'inverse étant d'environ 0,17 par pixel, l'influence au-delà de 7 pixels est inférieure à $`0.17^7 \approx 4 \times 10^{-6}`$. Résoudre chaque tuile avec une marge de 7 pixels revient donc à tout résoudre d'un coup.

### 4.6 Projections alternées

La condition de cohérence $`\{U : D U = I\}`$ est un ensemble affine, et la plage par cellule $`\{U : m \le U \le M\}`$ un ensemble boîte. Tous deux étant convexes, des projections alternées se rapprochent de leur intersection.

```math
U^{(k+1)} = \Pi_{\mathrm{box}}\Big(\Pi_{\mathrm{affine}}\big(U^{(k)}\big)\Big),\qquad k = 0, 1
```

Upscaly projette deux fois (une seule correction de cohérence si la plage par cellule est désactivée). La projection de cohérence n'est pas une projection orthogonale stricte mais une correction selon la direction de $`P`$, et comme la dernière projection est celle de la plage par cellule, la promesse « aucun nouvel extremum » est toujours tenue.

### 4.7 Déconvolution de Richardson–Lucy

Le flou est modélisé par une gaussienne $`h`$ d'écart type $`\sigma`$, et l'estimation $`e`$ est mise à jour de façon multiplicative à partir de l'observation $`o`$.

```math
e^{(t+1)} = e^{(t)} \cdot \left( h \ast \mathrm{clip}\!\left(\frac{o}{h \ast e^{(t)}}\right) \right)
```

Le rapport est écrêté pour éviter la divergence. Comme la mise à jour multiplicative préserve les multiples constants, appliquer le même nombre d'itérations à la couleur prémultipliée et à l'alpha conserve le rapport couleur/alpha dans une même zone de couleur.

### 4.8 Nettoyage du bruit (filtre bilatéral)

Il est appliqué séparément en horizontal et en vertical, et les poids d'écart de luminosité sont tabulés sur 1 025 entrées pour éviter les appels à la fonction exponentielle.

```math
\hat I_p = \frac{\sum_q w_s(\|p - q\|)\, w_r(|G_p - G_q|)\, I_q}{\sum_q w_s(\|p - q\|)\, w_r(|G_p - G_q|)}
```

$`G`$ est une luminosité approchée en gamma. Le plan alpha est moyenné avec les mêmes poids.

### 4.9 Restauration DCT fine

**Modèle de bruit.** Pour chaque canal (Y, Cb, Cr), le bruit est représenté par une variance $`S_c[k]`$ par fréquence DCT 8×8 et un gain $`g_c(I)`$ par plage de luminosité.

1. Des blocs à faible énergie haute fréquence ($`u + v \ge 6`$) sont sélectionnés de manière itérative, et la moyenne par fréquence donne la forme du spectre. L'énergie de la texture se concentre dans les basses fréquences ; seules les hautes fréquences servent donc à la sélection et au gain.
2. La luminance est plafonnée par un plancher de bruit PCA : la moyenne des 3 plus petites valeurs propres de la covariance à 63 dimensions des blocs faiblement texturés. La texture ne s'étale que dans quelques directions alors que le bruit s'étale dans les 63, ce qui évite de prendre la texture pour du bruit et de l'effacer. La finitude du nombre d'échantillons est corrigée par la borne inférieure de Marchenko–Pastur $`\gamma = \sqrt{n / n_{\mathrm{sel}}}`$.
3. En cas d'incertitude, l'estimation est biaisée vers le bas (côté qui préserve les détails). Une image propre où ne reste que le bruit de quantification 8 bits ($`1/12`$) est considérée comme pratiquement nulle.

**Débruitage.** C'est une structure en deux étapes sur une DCT 8×8 glissante, soit BM3D sans l'appariement de blocs. À partir de la luminosité moyenne du bloc $`\bar I`$, la variance du bruit par coefficient est posée à $`\sigma_k^2 = S[k]\, g(\bar I)\, \lambda^2`$ ($`\lambda`$ étant l'intensité). $`F_k`$ est le coefficient du bloc d'entrée, $`B_k`$ le coefficient du même bloc dans l'image résultat de l'étape 1.

```math
\text{Étape 1 : } \hat F_k = F_k \cdot \mathbf{1}\big[\,F_k^2 \ge \tau^2 \sigma_k^2\,\big],\qquad
\text{Étape 2 : } \tilde F_k = F_k \cdot W_k,\quad W_k = \frac{B_k^2}{B_k^2 + \sigma_k^2}
```

Les blocs qui se recouvrent sont combinés avec une fenêtre de Kaiser ($`\beta = 2`$) et, comme dans BM3D, avec un poids par bloc de $`1/(1 + n_{\mathrm{nz}})`$ à l'étape 1 et de $`1/\sum_k W_k^2 \sigma_k^2`$ à l'étape 2 ($`n_{\mathrm{nz}}`$ étant le nombre de coefficients restants). La grille des blocs est fixée en coordonnées globales, d'où l'absence de jointures entre tuiles.

**Projection sur les intervalles de quantification JPEG (QCS).** Pour un JPEG, la variance du bruit de quantification est ajoutée au modèle de bruit à l'aide de la table de quantification $`q_k`$ du fichier, et les coefficients DCT de la luminance sont ramenés dans l'intervalle autour de la valeur quantifiée observée. Les données entropiques n'étant pas décodées, les coefficients observés sont obtenus en arrondissant, selon la table de quantification, les valeurs $`G_k`$ de la DCT de l'image décodée alignée sur la grille 8×8 du fichier.

```math
c_k = q_k \cdot \mathrm{round}\!\left(\frac{G_k - \ell_k}{q_k}\right) + \ell_k,\qquad
F_k \leftarrow \mathrm{clip}\big(F_k,\ c_k - \Delta_k,\ c_k + \Delta_k\big),\qquad
\Delta_k = \tfrac12 \rho_k q_k + \delta_k
```

$`\ell_k`$ est le décalage de niveau de 128, présent seulement sur le DC. Pour les coefficients AC enregistrés avec une valeur non nulle, hors fréquences les plus basses ($`u + v \ge 2`$, la vraie texture laissée par l'encodeur), l'intervalle est resserré avec $`\rho_k = \rho`$ (0,5 par défaut) pour préserver l'intensité de la texture ; pour les coefficients nuls, le DC et les fréquences les plus basses (à l'origine des blocs et du ringing), on utilise tout l'intervalle avec $`\rho_k = 1`$ pour effacer les artefacts. $`\delta_k = 2.5\sigma_k`$ (AC seulement) laisse la marge d'effacer le bruit lorsqu'une image bruitée a été compressée. (Débruitage → projection) est répété deux fois, la seconde avec une variance de bruit de quantification réduite de moitié.

**Couleur.** Les plans de couleur sont nettoyés de la même manière, puis les frontières de couleur étalées par un sous-échantillonnage chromatique comme le 4:2:0 sont alignées, par un filtre guidé, sur les contours de la luminance restaurée (localement, couleur $`= a \cdot`$ luminance $`+ b`$).

**Récupération des détails.** Hors JPEG, lorsque l'énergie locale $`v`$ de ce qui a été retiré dépasse la quantité $`\tau \sigma^2`$ expliquée par le modèle de bruit, la fraction excédentaire est restituée.

```math
\hat I \leftarrow \hat I + \mu \left(1 - \frac{\tau \sigma^2}{v}\right)(I - \hat I)\qquad (v > \tau \sigma^2)
```

### 4.10 Masque flou (suppression des halos)

Après avoir calculé le détail $`d = I - G_\sigma \ast I`$ et l'amplitude locale $`A = G_{2\sigma} \ast |d|`$, on procède dans l'ordre suivant.

```math
d \leftarrow d \cdot \frac{d^2}{d^2 + t^2}\ \ (\text{protection des aplats}),\qquad
d \leftarrow a\, d,\qquad
d \leftarrow \frac{d}{1 + |d| / (o A + 10^{-4})}\ \ (\text{tolérance de halo})
```

$`a`$ est l'intensité, $`t`$ la protection des aplats et $`o`$ la tolérance de halo. La dernière formule est un limiteur à coude doux : sur les grands contours, le dépassement ne peut excéder $`o`$ fois l'amplitude locale. Un rayon de 0 est fixé automatiquement selon le facteur.

### 4.11 Lissage directionnel (suppression du crénelage)

Le tenseur de structure $`J = G \ast (\nabla Y \nabla Y^{\top})`$ est calculé sur l'original ; le vecteur propre de la petite valeur propre donne la direction tangente $`\mathbf t`$, et la grande valeur propre $`\lambda_1`$ fixe l'intensité $`w = \min(1, \lambda_1 / 0.0016)`$. Sur l'image agrandie, la moyenne se fait uniquement dans la direction tangente.

```math
\bar U = 0.34\,U(\mathbf x) + 0.22\,\big(U(\mathbf x + \ell \mathbf t) + U(\mathbf x - \ell \mathbf t)\big) + 0.11\,\big(U(\mathbf x + 2\ell \mathbf t) + U(\mathbf x - 2\ell \mathbf t)\big),\qquad
U \leftarrow U + w\,\kappa\,(\bar U - U)
```

La période des marches étant égale au facteur, le pas $`\ell`$ est fixé à 0,8 fois le facteur (1 à 8). Une seule passe ne suffit pas ; elle est répétée 1 à 4 fois en proportion de l'intensité. À chaque répétition, l'ondulation diminue de façon multiplicative alors que la largeur de transition reste la même. Le champ de directions est calculé en coordonnées globales ; il donne donc les mêmes valeurs aux limites des tuiles.

### 4.12 Retracé des contours (distance signée)

Lorsque les pixels se trouvent sur le segment reliant la couleur sombre et la couleur claire $`(\mathrm{lo}, \mathrm{hi})`$ d'une fenêtre 5×5 de l'original et que la transition se fait sur 1 à 2 pixels, on considère qu'il s'agit d'un contour. L'image agrandie est convertie en position $`t \in [0, 1]`$ sur ce segment, et une rampe d'un pixel de large est redessinée selon la distance signée à l'isoligne 0,5.

```math
t' = \mathrm{clip}\!\left(0.5 + \frac{t - 0.5}{|\nabla t|},\ 0,\ 1\right),\qquad
U \leftarrow U + \gamma\,w_{\mathrm{fit}}\big(\mathrm{lo} + t'(\mathrm{hi} - \mathrm{lo}) - U\big)
```

$`|\nabla t|`$ est la pente par pixel de sortie. Dans le calcul réel, la cible est $`U + (t' - t)(\mathrm{hi} - \mathrm{lo})`$ écrêtée à $`[\mathrm{lo}, \mathrm{hi}]`$, et le poids de mélange $`\gamma\,w_{\mathrm{fit}}`$ est obtenu en déployant de manière bilinéaire un poids construit à la résolution d'origine à partir de l'intensité du retracé et de la qualité d'ajustement (à quel point le 3×3 central s'écarte du segment). Le retracé à l'isoligne de 50 % ne conserve la surface (densité) que pour les traits de plus d'un pixel de large ; les traits fins dont la largeur, mesurée par la densité intégrée en travers du trait, est d'environ un pixel ne sont donc pas touchés, ce qui évite qu'ils ondulent.

### 4.13 Restauration des contours du pixel art

La forme est déterminée par le champ DCCI (les marches à 45° et 2:1 deviennent des lignes droites), et la couleur de chaque sous-échantillon est fixée à celle, parmi les 4 couleurs d'origine de la cellule 2×2 de l'original à laquelle il appartient, qui est la plus proche de la valeur du champ. Pour chaque pixel de sortie, 4×4 sous-échantillons sont moyennés afin de n'anticréneler que les contours ; les nouvelles couleurs n'apparaissent donc qu'aux limites des contours. Si les 4 couleurs de la cellule sont identiques, un chemin rapide la saute.

### 4.14 Consensus de balance des blancs

「Couleurs auto」 exclut de l'image d'analyse (lumière linéaire) les pixels saturés (0,95 ou plus) et presque noirs (0,004 ou moins), et obtient la couleur de l'illuminant par trois estimateurs.

```math
e^{\mathrm{sog}}_c = \Big(\tfrac1n \sum f_c^{6}\Big)^{1/6},\qquad
e^{\mathrm{ge}}_c = \Big(\tfrac1n \sum |\nabla (G \ast f_c)|^{6}\Big)^{1/6},\qquad
e^{\mathrm{wp}}_c = Q_{0.99}(f_c)
```

Ce sont Shades of Gray (moyenne), Gray Edge (contours) et White Patch (zones claires, 99e centile). $`G`$ est un lissage $`[1,4,6,4,1]/16`$. L'intensité $`a`$ est déterminée à partir de l'angle $`\theta`$ que forme chaque estimation avec le neutre $`(1,1,1)`$.

```math
a =
\begin{cases}
0 & \theta_{\min} < 3^\circ \ \text{ou}\ \theta_{\min} > 28^\circ \ \text{ou}\ \theta_{\max} - \theta_{\min} > 10^\circ \\[2pt]
\min\!\big(1,\ \theta_{\min} / \theta_{\mathrm{sog}}\big) \cdot h,\quad h = \begin{cases} \tfrac12 & \theta_{\mathrm{wp}} < 0.8\,\theta_{\mathrm{sog}} \\ 1 & \text{sinon} \end{cases} & \text{sinon}
\end{cases}
```

Les trois conditions correspondent, dans l'ordre, à l'absence de dominante, à une couleur propre à la scène difficilement attribuable à l'éclairage, et à un éclairage mixte. Si les zones claires sont moins teintées que la moyenne, on considère que la couleur du sujet a tiré la moyenne, et seule la moitié est corrigée. L'intensité est arrondie au pas de 0,05. Lors du traitement, les gains par canal sont calculés sur au plus 150 000 pixels, en excluant les pixels semi-transparents (alpha inférieur à 0,5), saturés et presque noirs ; ils sont normalisés pour conserver la luminance globale ($`0.2126R + 0.7152G + 0.0722B`$), mélangés par $`1 + (g - 1)a`$, puis écrêtés à 0,55–1,8.

### 4.15 Classifieur à 9 classes

Une régression logistique multinomiale est appliquée à $`\mathbf z`$, le vecteur de 27 caractéristiques $`\mathbf x`$ standardisé avec la moyenne et l'écart type d'apprentissage.

```math
P(k \mid \mathbf x) = \frac{\exp(\mathbf w_k^{\top} \mathbf z + b_k)}{\sum_j \exp(\mathbf w_j^{\top} \mathbf z + b_j)}
```

Comme le nombre d'exemples diffère selon les classes, des poids d'équilibrage des classes sont utilisés, et l'entropie croisée régularisée L2 a été minimisée par L-BFGS-B. Les caractéristiques influencées par la qualité JPEG (blocs d'une seule couleur, blocs sans bruit, plancher de bruit, blocs lisses) sont multipliées par une confiance de qualité $`r = \mathrm{clip}((q - 50)/40,\ 0.05,\ 1)`$. Les éléments de décision sont les contributions par caractéristique $`z_j (w_{k_1 j} - w_{k_2 j})`$ à l'écart de logit entre la 1re classe $`k_1`$ et la 2e $`k_2`$, par ordre décroissant.

### 4.16 Budget de rendu

Pour un facteur $`L`$, un nombre de pixels de l'original $`n_0`$ et un nombre de pixels du résultat $`N`$, la mémoire maximale de chaque étape est estimée ainsi (en octets).

```math
\begin{aligned}
M_{\mathrm{prep}} &= \max\big(32\,n_0,\ \ 4 c\, n_0 + 32\, n_0 s_k^2\big) && \text{préparation de l'original, étapes intermédiaires} \\
M_{\mathrm{fixed}} &= M_{\mathrm{other}} + M_{\mathrm{src}}^{1} + M_{\mathrm{keep}} + n_w (T + 64 + 2t + 2p)^2 (48 + 24[t > 0] + 32[p > 0]) + 1.5\,W \max(4T, 1024) + 1.1\,\rho\, c\, b\, N && \text{autres originaux et originaux préparés, plans conservés, tuiles des workers, bandes d'enregistrement, PNG compressé} \\
\text{garanti} &\iff M_{\mathrm{other}} + M_{\mathrm{src}}^{2} + M_{\mathrm{prep}} \le 0.8B \ \wedge\ 0.8B - M_{\mathrm{fixed}} \ge 4 \min\big(N, \max(4 \times 10^6, 2WT)\big)
\end{aligned}
```

$`c`$ est le nombre de canaux, $`s_k`$ le facteur intermédiaire de l'agrandissement par étapes, $`n_w`$ le nombre de workers, $`T`$ la taille des tuiles, $`t`$ la marge de résultat de la restauration de texture (0 si désactivée), $`W`$ la largeur du résultat et $`B`$ le budget mémoire. $`M_{\mathrm{src}}^{2}`$ représente tous les autres originaux préparés restés dans le cache (pendant une nouvelle préparation, les deux restent), et $`M_{\mathrm{src}}^{1}`$ le plus récent d'entre eux (une fois le nouveau ajouté, un seul reste). $`b`$ est le nombre d'octets par canal (1 en 8 bits, 2 en 16 bits) et $`\rho`$ le taux de compression maximal du PNG enregistré. Il a été mesuré avec l'encodeur de l'application en agrandissant les 47 images de `data/big` à 1,1, 1,25, 1,5, 2, 3 et 4×. À partir de 2×, on mesure à la fois le cas avec préréglage seul et le cas avec en plus les valeurs de la recherche de restauration poussée de 「Auto」 (`AUTO_BIG`), et on retient le plus grand ; en dessous de 2×, les valeurs de recherche n'étant pas utilisées, on mesure le préréglage seul (`research/budget/png_ratio.js`, `research/results/png_ratio.json`). On utilise la plus grande des valeurs des deux facteurs mesurés qui encadrent le facteur du résultat (la valeur de 4× au-delà de 4×). Un facteur de surface de 1,25× correspond à 1,12× en largeur et en hauteur et reste donc dans la plage mesurée ; en dessous de 1,1×, la taille non compressée est comptée. $`p`$ est la marge de résultat de 「Perception」 (les deux flous et le minimum/maximum du voisinage ; 0 en 「Précision」). Le taux de compression a aussi été mesuré en 「Perception」 (intensité jusqu'à 150 %, texture désactivée et activée), et la plus grande valeur des deux objectifs est utilisée (`research/results/png_ratio_pop.json`).

| Facteur | 8 bits, texture désactivée | 8 bits, texture activée | 16 bits, texture désactivée | 16 bits, texture activée |
|---|---|---|---|---|
| 1.1× | 0.721 | 0.767 | 0.928 | 0.933 |
| 1.25× | 0.686 | 0.766 | 0.913 | 0.928 |
| 1.5× | 0.650 | 0.760 | 0.898 | 0.926 |
| 2× | 0.688 | 0.790 | 0.876 | 0.933 |
| 3× | 0.615 | 0.751 | 0.842 | 0.931 |
| 4× et plus | 0.562 | 0.731 | 0.817 | 0.934 |

Les valeurs 8 bits sont mesurées avec le tramage activé, et « texture activée » correspond au plus grand des cas 25 % et 100 %. Le dernier terme est le cache de deux lignes de tuiles (au moins 4 mégapixels) nécessaire à l'enregistrement. Le budget mémoire $`B`$ vaut 15 % de `navigator.deviceMemory` (400 à 1500 Mo), à défaut 50 % de la limite du tas JS (1500 Mo au maximum), et à défaut 600 Mo. Sur mobile, c'est 12 % (400 Mo au maximum, 180 Mo à défaut).

### 4.17 Tramage 8 bits

À la valeur sRGB continue juste avant l'arrondi $`s = 255\,\mathrm{srgb}(v)`$, on ajoute un bruit à gradient entrelacé (IGN, Jimenez 2014) déterminé uniquement par les coordonnées du résultat $`(X, Y)`$.

```math
\nu(X, Y) = \mathrm{frac}\!\big(52.9829189 \cdot \mathrm{frac}(0.06711056\,X + 0.00583715\,Y)\big) - \tfrac12,\qquad
b = \big\lfloor s + \tfrac12 + 0.98\,\nu \big\rfloor
```

L'amplitude étant de ±0,49 niveau, une valeur telle que $`|s - k| < 0.01`$ (code exact $`k`$) devient bien $`k`$. La valeur sRGB continue est obtenue en interpolant la table sur l'axe $`\sqrt v`$ de l'encodeur 16 bits, avec une erreur d'interpolation d'au plus 0,002 niveau. Le même $`\nu`$ est utilisé pour les trois canaux, et l'alpha n'est pas tramé.

### 4.18 Restauration de texture

Sur la luminosité approchée en gamma de l'original (lumière linéaire, prémultiplié) $`Y = \sqrt{\ell}`$, $`\ell = 0.2126R + 0.7152G + 0.0722B`$, on mesure l'intensité locale de la deuxième bande la plus fine et le masquage par les contours.

```math
a = \mathrm{erode}_{5\times5}\!\left(\sqrt{G_{1.5} \ast (G_1 \ast Y - G_2 \ast Y)^2}\right),\qquad
m = \big(1 - c^2\big)^2,\quad c = \frac{\sqrt{(J_{xx} - J_{yy})^2 + 4J_{xy}^2}}{J_{xx} + J_{yy}}
```

$`G_\sigma`$ est une gaussienne, $`J = G_{1.5} \ast (\nabla Y \nabla Y^{\top})`$ le tenseur de structure et $`c`$ la cohérence directionnelle. Le minimum 5×5 (érosion) efface l'intensité des structures fortes et étroites comme les contours, les traits fins et les coins, et ne laisse que la texture largement répartie. $`m`$ tend vers 0 sur les contours qui s'étendent dans une seule direction.

À la résolution du résultat (facteur $`L`$), on crée un bruit de variance unitaire qui ne garde que la bande au-dessus de la fréquence de Nyquist de l'original. $`h(X,Y,s)`$ est un hachage entier ramené à [0, 1).

```math
n = \sqrt3\Big(\textstyle\sum_{s=1}^{4} h(X, Y, s) - 2\Big),\qquad
\eta = \frac{n - G_{0.375L} \ast n}{\sqrt{1 - 2k_0^2 + (\sum_i k_i^2)^2}}
```

$`k_i`$ est le noyau gaussien unidimensionnel, et le dénominateur ramène exactement à 1 la variance de cette différence. $`\sigma = 0.375L`$ est la valeur pour laquelle la réponse vaut la moitié à la fréquence de Nyquist de l'original. À partir de la variance cible et de la variance déjà présente dans la même bande du résultat, seule la part manquante est ajoutée.

```math
T = \big(\alpha\, C\, \sqrt{\log_2 L}\; a\, m\big)^2,\qquad
E = G_{0.5L} \ast \big(Y_U - G_{0.375L} \ast Y_U\big)^2,\qquad
\Delta = \mathrm{clip}\!\big(\sqrt{\max(0, T - E)}\;\eta,\ \pm 0.4\,Y_U\big)
```

$`\alpha`$ est le curseur d'intensité, $`C = 6.256`$ une constante ajustée par la mesure ([9.8](#98-restauration-de-texture)), et $`Y_U`$ la luminosité approchée en gamma du résultat. Enfin, les trois canaux (prémultipliés) sont multipliés par le même rapport $`f = \big((Y_U + \Delta)/Y_U\big)^2`$, ce qui laisse la teinte et l'alpha inchangés. Lors d'un éclaircissement ($`f > 1`$), le rapport est réduit à $`f \le \max\!\big(1,\ \min(1, A) / \max(R, G, B)\big)`$ pour que le plus grand canal ne dépasse pas la limite (l'alpha, ou 1 si opaque). Sans cela, sur les couleurs claires et saturées, un seul canal serait écrêté et la teinte changerait (par exemple 250·127·73 devenant 255·130·76, corrigé en v1.1.1). Le bruit, les statistiques de l'original et la fenêtre d'énergie du résultat sont tous déterminés en coordonnées globales, et une marge correspondante est calculée en plus ; le résultat est donc le même quelles que soient les tuiles.

### 4.19 Détection du gamut et profil Display P3

La matrice des primaires $`(\mathbf r, \mathbf g, \mathbf b)`$ du profil ICC (XYZ en D50) est comparée aux valeurs du sRGB ; si une seule composante diffère de plus de 0,02, le gamut est considéré comme large. Pour les profils LUT sans matrice, on examine la chaîne de description (P3, Adobe, 2020, ProPhoto). Pour le cICP du PNG et le nclx de l'AVIF, le gamut est considéré comme large si le numéro des primaires n'est ni 1 (BT.709 = sRGB) ni 2 (non spécifié). Pour l'AVIF et le HEIF, on lit dans la boîte `meta` le `pitm` (élément principal) et, dans `iprp`, `ipma` (élément ↔ numéros de propriétés) et `ipco` (propriétés), afin de ne considérer que la `colr` rattachée à l'image principale. Même si la `colr` d'une vignette ou d'une image auxiliaire vient en premier, c'est l'image principale qui compte. Si cette structure ne peut pas être lue, la première `colr` rencontrée est utilisée.

Le profil Display P3 joint à l'enregistrement est construit par l'application elle-même (ICC v4.3, 532 octets). Les primaires sont obtenues en calculant la matrice RGB→XYZ à partir des coordonnées chromatiques P3 (0.680, 0.320), (0.265, 0.690), (0.150, 0.060) et du blanc D65, puis en l'adaptant à D50 par la transformation de Bradford.

```math
\mathbf r = (0.515119,\ 0.241189,\ -0.001050),\quad
\mathbf g = (0.291978,\ 0.692244,\ 0.041879),\quad
\mathbf b = (0.157103,\ 0.066567,\ 0.784071)
```

La courbe est la même courbe paramétrique que le sRGB (type 3 : $`\gamma = 2.4`$, $`a = 1/1.055`$, $`b = 0.055/1.055`$, $`c = 1/12.92`$, $`d = 0.04045`$). Ces valeurs coïncident à $`10^{-5}`$ près avec les primaires du profil Display P3 d'Apple.

### 4.20 Mesures

```math
\mathrm{PSNR} = 10 \log_{10} \frac{255^2}{\frac{1}{3n}\sum (U - I_{\mathrm{gt}})^2},\qquad
E_{\mathrm{dir}} = \frac{\sum_p m_p\, \arccos\!\big(|\hat{\mathbf g}^{U}_p \cdot \hat{\mathbf g}^{\mathrm{gt}}_p|\big)}{\sum_p m_p}\quad (m_p \ge 80)
```

$`E_{\mathrm{dir}}`$ est l'erreur de direction des contours. Sur les contours forts de l'original (magnitude de Sobel de 80 ou plus, soit un saut d'environ 20 niveaux), l'angle d'écart entre les directions de gradient du résultat et de l'original est moyenné, pondéré par l'intensité du contour. Quand des marches apparaissent, la direction du gradient penche vers les axes et l'erreur augmente. Le SSIM suit la définition de Wang et al. (2004).

### 4.21 Netteté perçue

En mode 「Perception」, deux bandes sont ajoutées à la luminance du résultat $`Y`$ ($`\sqrt{\ell}`$ en lumière linéaire, $`\ell`$ pour des valeurs gamma ; $`\ell`$ est la luminance des canaux écrêtés entre 0 et le plafond). $`L`$ est le facteur, $`G_\sigma`$ une gaussienne (approximation par trois boîtes) et $`a`$ l'intensité de perception.

```math
d_1 = Y - G_{0.5L}Y,\qquad d_2 = G_{0.5L}Y - G_{2L}Y,\qquad
\tilde d = d\,\frac{d^2}{d^2+\tau^2}\quad(\tau = 0.004)
```

```math
\Delta = a\Big(k_1\,\tilde d_1 + k_2\,\frac{\tilde d_2}{1+|\tilde d_2|/c_2}\Big),\qquad
Y' = Y + \Delta\,\frac{h}{h+|\Delta|},\qquad
h = \begin{cases}\max_{\mathcal N}Y - Y & \Delta>0\\ Y-\min_{\mathcal N}Y & \Delta\le 0\end{cases}
```

$`d_1`$ est la pente des contours près de la fréquence de Nyquist de l'original, $`d_2`$ le contraste local à l'échelle de 1 à 4 pixels d'origine. $`\tilde d`$ est un seuil de même forme que le masque flou : les petites amplitudes comme le bruit et les traces JPEG ne sont presque pas augmentées. Le contraste local est d'autant moins augmenté que son amplitude est grande ($`c_2 = 0.05`$) : un contraste déjà marqué reste tel quel, un contraste enfoui ressort. $`\mathcal N`$ est un voisinage carré de rayon $`\mathrm{round}(0.75L)`$ (à l'intérieur de l'image seulement) et $`h`$ la marge restante jusqu'au minimum ou au maximum du voisinage. $`\Delta\,h/(h+|\Delta|)`$ ne fait que s'approcher doucement de cette marge sans la franchir : aucun nouvel extremum (halo) n'apparaît, et comme les valeurs ne se collent pas à la borne, les surfaces ne s'agglutinent pas (pas de postérisation). Une première version écrêtait exactement à la plage du voisinage ; les surfaces s'agglutinaient et l'image ressemblait à une peinture à l'huile, d'où ce changement. La nouvelle luminance est appliquée comme un rapport $`s = Y'/Y`$ ($`s^2`$ en lumière linéaire) multiplié à l'identique dans les trois canaux de couleur prémultipliés, ce qui conserve teinte, gris et alpha ; quand on éclaircit, le rapport est réduit pour que le plus grand canal reste sous le plafond (même traitement que la restauration de texture). Avec un canal alpha, la luminance est mesurée sur la couleur non prémultipliée, les flous sont des moyennes pondérées par l'opacité, et les pixels presque transparents (alpha inférieur à 2 %) sont exclus du minimum/maximum du voisinage : une frontière où seule l'opacité change, à couleur égale, ne reçoit donc aucun liseré.

$`k_1 = 0.5`$ et $`k_2 = 1.5`$ ont été choisis sur les images d'exploration. Les 16 combinaisons $`k_1 \in \{0.5, 1, 2, 3\}`$ × $`k_2 \in \{0, 0.5, 1, 1.5\}`$ ont été mesurées sur 22 images × 3 conditions (4× propre, 4× dégradé, 8× dégradé ; 66 cas), en comparant la médiane de l'erreur logarithmique $`|\ln A| + |\ln C|`$ de la pente des contours $`A`$ et du contraste local $`C`$ par rapport à la vérité terrain. $`A`$ est le rapport des pentes mesurées sur les contours de la vérité terrain (10 % de pente la plus forte) après avoir flouté les deux avec $`\sigma = 0.5L`$ ; $`C`$ est le rapport des RMS de la bande $`G_{0.5L}-G_{2L}`$. Les 16 combinaisons ont toutes une erreur inférieure à 「Précision」 (0,418), de 0,343 à 0,410 ; parmi elles, celle qui ajoute le moins de halo et de bruit sur les aplats a été retenue (erreur 0,357, halo +0,20 niveau, bruit sur les aplats ×1,14, SSIM +0,004). Les résultats sont dans `research/results/pop_grid.json`.

---

## 5. Le monde physique

| Phénomène physique | Modèle dans Upscaly |
|---|---|
| Un pixel de capteur additionne toute la lumière tombée sur sa surface | Modèle de réduction = moyenne de surface en lumière linéaire $`D`$. L'objectif de la correction de cohérence $`DU = I`$, la mesure de l'agrandissement par étapes et la réduction des données de recherche reposent tous sur ce modèle |
| Étalement optique des objectifs et scanners | Fonction d'étalement du point gaussienne (σ 0,45 à 0,95). Estimée en sens inverse par déconvolution RL |
| L'intensité du bruit de capteur varie avec la luminosité | Spectre de bruit avec un gain $`g(I)`$ par plage de luminosité |
| Le JPEG arrondit les coefficients DCT à des entiers | Un intervalle de quantification de largeur $`q_k`$ par coefficient. Le vrai original se trouve quelque part dans cet intervalle |
| Le sous-échantillonnage chromatique (4:2:0) enregistre la couleur à demi-résolution | Filtre guidé qui aligne les frontières de couleur sur les contours de luminance |
| La couleur de l'éclairage teinte jusqu'aux zones claires | Règle de consensus de trois estimateurs |
| Un bord semi-transparent mélange la lumière du premier plan et de l'arrière-plan selon leur proportion de surface | Alpha prémultiplié. Mêmes opérations pour la couleur et l'alpha |
| L'œil perçoit la luminosité de façon presque logarithmique | Enregistrement en sRGB, décisions de direction et de contour sur un plan guide en luminance gamma |
| Le spectre de puissance des images naturelles est d'environ $`1/f^2`$ (énergie à peu près égale par octave) | Estimation des octaves manquantes de la restauration de texture : intensité de bande de l'original × $`\sqrt{\log_2 L}`$ |
| Le grain de pellicule est fait de fines particules placées au hasard | La restauration de texture est un grain qui n'ajuste que la quantité de texture. L'emplacement et la forme de la texture ne figurent pas dans l'original |
| L'œil est sensible aux marches (bandes) sur de grandes surfaces, et peu sensible à un bruit très fin | Tramage 8 bits : déplace l'erreur de quantification basse fréquence vers un bruit haute fréquence |
| La plage de couleurs reproductibles (gamut) diffère selon les écrans et les appareils photo | Le profil colorimétrique de l'original est lu ; traitement et enregistrement avec un canevas P3 et un profil P3 |

---

## 6. Architecture

### Fichiers et environnement d'exécution

L'outil entier tient dans un seul fichier, `upscaler.html` (environ 6 140 lignes, 360 Ko), contenant le HTML, le CSS et le JavaScript. Il n'y a ni bibliothèque externe ni serveur. Seule la police de l'interface (IBM Plex Sans KR) est téléchargée depuis Google Fonts ; à défaut, la police système est utilisée. `CompressionStream` sert à l'enregistrement PNG en flux et les Web Workers au calcul parallèle ; en leur absence, on se replie sur des chemins plus lents. Si l'appareil est en mode sombre, un thème sombre est utilisé.

### Composition

| Couche | Fonctions principales | Rôle |
|---|---|---|
| Espace colorimétrique | `S2L`, `L2S`, `enc16Linear` | Tables sRGB ↔ lumière linéaire, encodeur 16 bits |
| Moteur | `ENGINE_FACTORY`, `PORT` | Calcul pur commun au thread principal et aux workers |
| Interpolation | `dcciStageMulti`, `ediUpsample`, `resample2D`, `interpRegion` | DCCI, rééchantillonnage par noyau |
| Contraintes | `cellRange`, `clampRegion`, `consistencyCorrectPlanar` | Plage par cellule, cohérence (Thomas) |
| Restauration | `estimateNoise`, `sdctDenoise`, `qcsLuma`, `guidedFilter`, `restoreTile`, `richardsonLucy`, `denoise`, `deblock` | Bruit, JPEG, flou |
| Contours | `edgeField`, `antialiasRegion`, `crispFields`, `steepenRegion`, `pixelTile` | Lissage directionnel, retracé, pixel art |
| Tuiles | `renderTile`, `tileJob`, `cropRectFor`, `outMarginFor` | Calcul d'une tuile, planification des marges |
| Préparation de l'original | `prepareSource`, `dctRestorePool` | Corrections à la résolution d'origine, étapes intermédiaires de l'agrandissement par étapes |
| Parallélisme | `initPool`, `poolRun`, `runLimited` | Pool de workers, file d'attente, limite du nombre de fragments simultanés |
| Cache | `tcGet`, `tcPut`, `tcMax`, `srcGet` | LRU des tuiles de résultat, 2 originaux préparés |
| Auto | `analyzeImage`, `autoClassify`, `autoPick`, `AUTO_BIG`, `autoAA`, `autoStage`, `pickAutoScale`, `renderBudget` | Analyse, classification, réglages, facteur |
| Traitement des couleurs | `estimateWB`, `colorAuto` | Balance des blancs |
| Tramage 8 bits | `ditherTile`, `srgbF` | Tramage IGN déterminé par les coordonnées |
| Texture | `texFields`, `texNoise`, `addTexture`, `texMargin` | Intensité de texture de l'original, bruit de bande, ajout de la seule part manquante |
| Gamut | `P3_OK`, `sniffWideGamut`, `iccIsWide`, `displayP3Icc`, `p3IccpChunk` | Détection du gamut large, canevas P3, profil P3 |
| Enregistrement | `createPngEncoder`, `exportPng`, `exportViaCanvas`, `saveAll` | PNG par lignes de tuiles, enregistrement groupé |
| Écran | `render`, `drawTiles`, `detectSubjects` | Vue de comparaison, cadre du sujet |

### Flux de données et cache

Chaque image conserve séparément son canevas d'origine, sa grille et sa qualité JPEG, son modèle de bruit (calculé une fois, au premier besoin), son cadre du sujet et ses réglages. Chaque résultat contient un instantané des réglages au moment de sa création ; même si les réglages changent ensuite, l'enregistrement et le recalcul donnent les mêmes valeurs que ce résultat.

Les tuiles de résultat sont conservées dans un cache LRU. La limite du cache est de 60 % du budget mémoire ; si la création d'un canevas échoue réellement, 80 % de la taille du cache à ce moment est mémorisé comme limite de l'appareil. Pour le facteur choisi par le facteur automatique, la limite est encore abaissée selon les conditions de garantie. Pour un résultat qui ne tient pas entièrement dans le cache, seule la moitié du cache est calculée d'abord, en partant du centre, et le reste est calculé quand il apparaît à l'écran ou à l'enregistrement. Une tuile visible pas encore calculée est remplacée provisoirement par l'original agrandi, et son calcul est confié en arrière-plan.

### Déterminisme : écran = résultat enregistré

L'écran, l'enregistrement et le recalcul utilisent tous le même `renderTile`. Les traitements qui regardent les voisins (interpolation, cohérence, masque flou, lissage directionnel, retracé) calculent une marge supplémentaire du rayon nécessaire puis la rognent, et la grille des blocs DCT et le champ de directions sont fixés en coordonnées globales. Le résultat est donc le même quels que soient la taille et l'ordre des tuiles et le nombre de workers. Même en découpant finement en tuiles de 64 px, les jointures restent à au plus 1 niveau, et deux calculs avec les mêmes réglages diffèrent de 0 ([9.1](#91-invariants)).

---

## 7. Problèmes et solutions

### Qualité d'image

| Problème | Cause | Solution |
|---|---|---|
| Liseré blanc autour des contours (ringing, halo) | Les lobes négatifs du noyau créent des extrema absents de l'original | Écrêtage aux 4 voisins de DCCI, contrainte de plage par cellule, limiteur à coude du masque flou |
| Le résultat réduit diffère de l'original | Même si l'interpolation passe par les échantillons de l'original, la moyenne de surface ne correspond pas | Résolution exacte de la correction de cohérence à la résolution d'origine (Thomas) |
| La cohérence et la plage par cellule se cassent mutuellement | En satisfaire une fait dévier l'autre | Deux projections alternées |
| Bavure de couleur sur les bords semi-transparents (erreur de composition jusqu'à 29 niveaux) | L'alpha et la couleur subissaient des opérations différentes | L'alpha subit les mêmes opérations que la couleur (0,1 à 4 niveaux) |
| L'agrandissement par étapes dégrade au contraire (−0,4 à −0,8 dB) | La référence de la plage par cellule et de la cohérence était l'image intermédiaire | Référence ramenée à l'original (+0,03 à 0,06 dB) |
| Les marches de l'original réapparaissent sur les dessins au trait | La cohérence traite les marches de l'original comme la bonne réponse | Retracé et lissage directionnel pour les dessins au trait et logos ; la recherche décide par type si la cohérence est utilisée |
| Les traits fins ondulent | Le retracé à l'isoligne de 50 % ne conserve pas la densité des traits plus fins qu'un pixel | Mesure de la largeur des traits par densité intégrée, traits fins laissés tels quels |
| Erreurs de direction de DCCI dans le bruit | Le bruit perturbe les variations directionnelles | Ajout d'un plancher de bruit $`n_{\mathrm{off}}`$ aux variations (σ d'Immerkær) |
| Texture prise pour du bruit et effacée | Les blocs texturés gonflent le spectre de bruit | Plafond par plancher de bruit PCA, sélection sur les seules hautes fréquences |
| La restauration JPEG efface jusqu'aux détails du fichier | Le débruitage seul ne connaît pas les coefficients d'origine | Projection sur les intervalles de quantification (QCS) |

### Couleur

| Problème | Cause | Solution |
|---|---|---|
| Les préréglages modifiaient aussi les couleurs | Autrefois, la balance des blancs faisait partie des préréglages | Traitement des couleurs séparé en fonction distincte, 0 par défaut, uniquement via le bouton 「Couleurs auto」 |
| Chat roux et coucher de soleil pris pour une dominante | Un seul estimateur (la moyenne) était utilisé | Consensus de trois estimateurs (sur 54 paires, forte dérive de couleur 13,9° → 4,0°) |
| Les frontières sombres s'empâtent | Interpolation dans l'espace gamma | Calcul en lumière linéaire |
| Les photos de téléphone deviennent ternes après agrandissement | Le canevas sRGB écrasait les couleurs Display P3 dans le sRGB (rouge P3 : 234,51,35 en valeurs P3) | Lecture du profil de l'original, traitement sur canevas P3 et enregistrement avec un profil P3 (cICP·iCCP) (v1.1.0) |
| Bandes en escalier dans les ciels fortement agrandis | Les bandes de l'arrondi 8 bits s'élargissent du facteur | Tramage déterminé par les coordonnées (v1.1.0) |
| Points blancs dans les coins et zones sombres quand la texture est ajoutée | L'intensité de bande des structures fortes et étroites est élevée, et dans les zones sombres une même variation est relativement plus grande | Érosion 5×5 de l'intensité de texture, variation par pixel limitée à ±40 % de la luminosité (√) |
| Avec 「Calculer en lumière linéaire」 désactivé, la texture déborde d'un facteur 1,8 | Les valeurs gamma étaient prises pour de la lumière linéaire, et √ puis le carré étaient appliqués une fois de trop | En mode gamma, l'axe de luminosité reste en valeurs gamma et le rapport n'est pas élevé au carré. R sur 12 photos : 2,24 avant correction, 1,32 après (mode lumière linéaire 1,22, v1.1.1) |
| La texture modifie la teinte des couleurs claires et saturées | Même avec le même rapport sur les trois canaux, seul le plus grand canal est écrêté à la limite | Rapport d'éclaircissement limité par la marge du plus grand canal (alpha·1). Rouge clair à 4× : variation maximale du rapport de couleur linéaire 14,0 % → 2,9 % (niveau de la quantification 8 bits, v1.1.1) |
| L'intensité de texture déborde sur les entrées propres et manque sur les entrées dégradées | La bande la plus fine est la première rognée par le flou et le JPEG | Mesure sur la bande une octave plus bas, soustraction des hautes fréquences déjà présentes dans le résultat et ajout de la seule part manquante |

### Stabilité et entrées/sorties

| Problème | Cause | Solution |
|---|---|---|
| L'application se fige dès le démarrage | `updateSizeNote`, appelée au démarrage, lisait un global `let` déclaré plus tard : erreur TDZ | Ces globaux sont déclarés avec `var` ou placés en tête |
| Une mesure terminée en retard s'applique à une nouvelle image | Concurrence entre mesures asynchrones | Numéro de requête automatique `AUTO_SEQ` et vérification `alive()` |
| Un grand canevas reste vide sans exception | Limite de surface cachée du navigateur | Si le canevas de l'original est entièrement transparent, l'original est lu directement par fragments de 1024² pour vérifier s'il contient des pixels visibles ; à l'assemblage pour l'enregistrement, un pixel visible du résultat est recherché et la même position du canevas est vérifiée, puis la limite est mémorisée. Détecte aussi une image où un seul point est visible (v1.1.1 ; auparavant seul le pixel en bas à droite était examiné, et s'il était transparent à l'origine, rien n'était détecté) |
| Le gamut large d'un AVIF était détecté comme sRGB | La première boîte colr du fichier était utilisée, or elle pouvait appartenir à une vignette | Seule la colr rattachée à l'image principale via `pitm`·`ipma`·`ipco` est considérée (v1.1.1) |
| Le gamut large d'un PNG était détecté comme sRGB | La lecture s'arrêtait si un chunk sRGB ou iCCP précédait le cICP | Tous les chunks de couleur avant IDAT sont examinés, puis cICP > iCCP > sRGB. Si les primaires cICP sont non spécifiées (2) ou un numéro réservé, iCCP est utilisé (v1.1.1) |
| 「Agrandir par étapes」 modifié juste après l'ouverture d'une image revenait en arrière | La mesure qui fixe la valeur initiale de la case se terminait en retard et l'écrasait | Un numéro (`STAGE_SEQ`) est incrémenté chaque fois que l'utilisateur ou 「Auto」 décide, et la mesure n'est pas appliquée s'il diffère de celui du début de la mesure ou si les conditions ont changé (v1.1.1) |
| Bavure de couleur sur les bords et jointures entre tuiles pour les images transparentes agrandies par étapes | Aux étapes intermédiaires, seul l'alpha passait par un rééchantillonnage ordinaire | L'alpha subit les mêmes opérations que la couleur (couleur des bords DCCI 4× : maximum 19 à 38 → 1 à 4 niveaux, jointure du logo 206 → 1, v1.1.1) |
| Le décodage des gros fichiers ne se termine pas | Cas où ni `onload` ni `onerror` n'arrive | Les trois chemins sont essayés tour à tour avec un délai proportionnel à la taille |
| Environnement où les workers sont bloqués | Politique de sécurité | Vérification de réponse de 5 secondes, puis thread principal |
| Dépassement de mémoire à cause d'un grand résultat | Canevas du résultat complet | Cache de tuiles et PNG en flux |
| Changer le facteur après 「Auto」 désaccorde les réglages | 「Auto」 décide en fonction de ce facteur | Avertissement via `AUTO_LAST` et invitation à cliquer de nouveau |
| La validation sur entrées propres était traitée comme une entrée JPEG | Le script de validation transmettait la qualité JPEG 85 même pour les entrées propres | Nouvelle validation sans information JPEG, valeurs de recherche des logos et bandes dessinées réservées aux entrées JPEG (v1.1.0) |

---

## 8. Optimisation

Les optimisations de vitesse n'ont été introduites que lorsque la sortie reste identique à l'octet près. Les sorties 16 bits et les statistiques de 134 combinaisons de réglages sont comparées à la version de référence (`tests/engine_regression.js`). Exception : la table de grille interne DCCI, première ligne du tableau ci-dessous. C'est un compromis retenu après mesure, qui modifie légèrement la sortie (±0,007 dB) en échange de vitesse ; la version de référence actuelle est postérieure à l'introduction de cette table.

| Optimisation | Contenu | Effet |
|---|---|---|
| Table de grille interne DCCI | À 6, 8 et 12×, grille 4× au lieu de 8×. Moins de ±0,007 dB sur 25 préréglages × 1 800 paires | 25 à 30 % plus rapide |
| Partage des poids directionnels DCCI | Calculés une fois sur le plan guide et utilisés pour tous les canaux | |
| Cache de plage par cellule | Minima et maxima par cellule calculés une seule fois par plan de canal | |
| Regroupement des canaux pour la cohérence | Empreintes, coefficients et poids d'interpolation partagés sans copier les plans de canal dans un tableau entrelacé | |
| Total | 1024×768 → 4×, 「Standard」(표준) | 13,9 s → 12,0 s, 134 combinaisons identiques à l'octet |

Par ailleurs, la gaussienne est approchée par trois flous en boîte (méthode de Kutskir), les poids de luminosité du filtre bilatéral sont tabulés, et les bandes 16 bits sont recalculées par 128 lignes pour économiser la mémoire. La grille 4× a aussi été essayée jusqu'à 2×, mais bien que la moyenne ait gagné +0,013 dB, les cas dégradés (24 %) étaient plus nombreux que les cas améliorés (21 %) et le temps doublait ; elle n'a donc pas été retenue.

Les goulots d'étranglement restants sont l'interpolation DCCI (par canal), la correction de cohérence et le rééchantillonnage. Il est difficile de les réduire nettement sans modifier la sortie.

---

## 9. Validation

### 9.1 Invariants

`tests/verify_suite.js` charge le script de l'application tel quel dans un faux DOM et exécute jusqu'au bout le vrai chemin de traitement.

| Élément | Attendu | v1.1.1 |
|---|---|---|
| Jointures entre tuiles (29 préréglages, tuiles de 64 px) | 1 au maximum | Toutes à 1 au maximum |
| Jointures entre tuiles de l'agrandissement par étapes (lanczos3 4×·edi 2×·pixelart 4×) | 0 | 0 · 0 · 0 |
| Déterminisme (5 préréglages calculés deux fois) | 0 | 0 |
| Identité à 1× sans traitement | 0 | 0 |
| Exactitude au bit du plus proche voisin 3× | 0 valeur divergente | 0 |
| Conservation du noir et blanc (écart maximal R·G·B sur 29 préréglages) | 0 | 0 |
| Couleur des bords alpha (alpha d'au moins 5 %) | 4 au maximum | 4 |
| Couleur des bords alpha et jointures entre tuiles de l'agrandissement par étapes 4× (photo·logo × lanczos3·edi) | 4 au maximum · 1 au maximum | 4 · 0, 0, 1, 1 (v1.1.0 : couleur 22 à 38, jointure du logo 206) |
| Aller-retour de la table de lumière linéaire (256 niveaux) | 0 divergence | 0 |

Les invariants sont mesurés avec le tramage 8 bits activé (par défaut). Hormis l'élément alpha de l'agrandissement par étapes, ils sont identiques à ceux de la v1.0.0. Voici les autres tests.

| Test | Contenu | v1.2.2 |
|---|---|---|
| `e2e_presets.js` | Traitement de 8 préréglages + enregistrement 8 et 16 bits | ✓ 8, 45,711 · 239,731 bytes (en 8 bits, +5 % par rapport aux 43,446 de la v1.0.0 à cause du tramage) |
| `engine_regression.js` | Comparaison des sorties 16 bits et statistiques de 134 combinaisons avec la version de référence | 134/134 identiques à l'octet |
| `worker_test.js` | Tuile du worker = calcul principal | Identiques à l'octet |
| `alpha_probe.js` | Erreur de couleur des bords semi-transparents par préréglage | |
| `e2e_autoscale.js` | Facteur automatique : avec un budget de 600 Mo, 512²·800×600 → 8×, 4000×3000 → 2×. Avec le plus proche voisin et DCCI en trois étapes (texture activée), la garantie tient même après 「Auto」, et le facteur supérieur n'est pas garanti avec les réglages 「Auto」 de ce facteur (6 cas). 4000×3000 avec classification fixée dans l'interface, 「Auto」 depuis 1.25× → 2× | 4000×3000 donnait 4× en v1.1.0. À 4×, le résultat fait 192 MP et le seul PNG compressé accumule jusqu'à environ 300 Mo : ce n'est pas garanti. Dans le cas de l'interface, les versions antérieures à v1.1.1 restaient à 1.25× avec 「Aucune garantie」(보장 없음) |
| `e2e_auto_stale.js` | Avertissement en cas de changement de facteur après 「Auto」 | |
| `e2e_stage_force.js` | Activation forcée de l'agrandissement par étapes et voyant, conservation des changements de l'utilisateur pendant la première mesure | ✓ 2 (v1.1.0 écrasait les deux) |
| `blank_canvas.js` | Détection des canevas vides : 6 cas de lecture de l'original, 4 cas d'enregistrement par canevas | ✓ 10 (v1.1.0 manquait 4 cas : coin inférieur droit transparent, un seul point visible) |
| `gamut_sniff.js` | Détection du gamut large : 5 cas de colr de l'image principale AVIF, 4 cas d'ordre des chunks de couleur PNG, 5 cas de métadonnées ou boîtes de plus de 2 Mo avant l'information de couleur (XMP étendu JPEG, chunks texte PNG, mdat AVIF avant meta), 4 cas de numéros de primaires cICP (non spécifié·réservé·sRGB) | ✓ 18 (v1.1.0 en ratait 10) |
| `dither_probe.js` | Indicateur de bandes du tramage 8 bits | [9.7](#97-tramage-8-bits) |
| `texture_seam.js` | Texture activée : jointures, déterminisme, noir et blanc, teinte des couleurs claires | Jointure maximale 1 (comme avec la texture désactivée), 0 entre deux calculs, 0 en noir et blanc, variation du rapport de couleur du rouge clair 2.9 % (v1.1.0 : 14.0 %) |
| `pop_seam.js` | 「Perception」 : jointures de tuiles · déterminisme · niveaux de gris · absence de halo (dans la plage du voisinage du résultat Précision) · mode gamma · teinte des couleurs vives · frontières où seule l'opacité change | Jointure au plus 1 (comme Précision), 0 entre deux calculs, niveaux de gris 0, au plus 0,9 niveau au-delà de la plage du voisinage (arrondi 8 bits), variation du rapport de couleur du rouge vif 2,8 %, écart de couleur aux frontières d'opacité au plus 1 (24 avant la correction, 18 sur un dégradé blanc) |
| `p3_browser.js` | Préservation des couleurs P3 dans un vrai Chromium (primaires et couleurs mélangées, y compris la simulation d'environnements où les images blob: sont bloquées et où `createImageBitmap` convertit en sRGB) | [9.9](#99-display-p3) |

### 9.2 Données

`data/ds` contient 567 originaux répartis en 9 classes. Il contient aussi des variantes créées à partir des mêmes originaux par qualité JPEG, réduction, flou, bruit, simulation de numérisation, etc.

| Type | Photo | Illustration | Peinture | Bande dessinée | Dessin au trait | Document | UI | Logo | Pixel | Total |
|---|---|---|---|---|---|---|---|---|---|---|
| Nombre | 135 | 152 | 36 | 45 | 40 | 55 | 38 | 43 | 23 | 567 |

`data/big` sert à la recherche de restauration poussée (`research/search/prep_big.py`). Pour chaque type, 3 images de recherche et 3 images de validation sont mises à part ; dans chaque original, le fragment de 256 px le plus riche en détails sert de vérité terrain, à partir de laquelle quatre entrées sont créées.

| Condition | Entrée | Vérité terrain |
|---|---|---|
| Dégradation réaliste 2·4·8× | Réduction par surface en lumière linéaire → flou σ0.6 → JPEG 85 (4:2:0) | Fragment de 256 px |
| Entrée propre 4× | Réduction par surface en lumière linéaire seule | Fragment de 256 px |
| Grande image 4× (validation seulement) | Dégradation réaliste des 384 à 512 px centraux | Ce fragment |

### 9.3 Préréglages graphiques et Auto pour les images dessinées

Les 11 préréglages celluloïd, couleur numérique, affiche, bande dessinée, dessin au trait, document, écriture manuscrite, plan, UI, texte d'écran et logo activent la cohérence ; le lissage directionnel est abaissé à 50 % pour le celluloïd et à 75 % pour la couleur numérique et l'affiche. Ce sont des réglages qui, sur 29 images d'ajustement et 30 images de validation dégradées par flou + JPEG, font gagner 0,2 à 0,6 dB en PSNR·SSIM, tout en maintenant les marches des contours à 45° et autres (erreur de direction) à moins de 10 % de l'état initial. Lorsque 「Auto」 pour les images dessinées a mesuré le lissage directionnel image par image pour le choisir, on a obtenu en moyenne +0,44 dB et une erreur de direction des contours de −8,4 % sur 59 images graphiques.

### 9.4 Recherche de restauration poussée

**Espace.** Noyau 4 × cohérence 2 × plage par cellule 2 × quantité d'accentuation 19 × (rayon, seuil, dépassement, lissage directionnel, retracé, multiplicateur de déconvolution, multiplicateur de bruit, multiplicateur de restauration JPEG, agrandissement par étapes) 10 chacun × multiplicateur du σ de déconvolution 5 = **1,520,000,000,000 combinaisons**.

**Contraintes.** Baisse du SSIM d'au plus 0,003 et erreur de direction des contours d'au plus +10 % par rapport à l'Auto actuel.

**Procédure.** 1 500 tirages aléatoires → réévaluation des 150 meilleurs → descente par coordonnées depuis 4 points de départ (toutes les étapes, jusqu'à convergence) → pour les valeurs collées au bout d'un curseur, élargissement de la plage et nouvelle passe. Environ 55 000 évaluations au total. Le journal (`data/big/search.jsonl`) s'accumule, ce qui permet de reprendre là où on s'est arrêté.

**Validation.** Écart de PSNR (dB) par rapport à l'Auto précédent, sur les 3 images de validation mises à part. Seuls les types et facteurs dont la moyenne augmente et où au moins 2 images sur 3 s'améliorent sont retenus. L'ensemble du tableau a été remesuré en v1.1.0 (`research/results/val_v3.json`). L'ancienne validation différait de l'usage réel sur deux points. Premièrement, la qualité JPEG 85 était transmise même pour les entrées propres, si bien que la restauration JPEG s'appliquait à l'Auto précédent servant de référence. Deuxièmement, seul le nombre de qualité était transmis pour les entrées JPEG, sans la table de quantification ni la grille, si bien que la restauration par intervalles de quantification utilisée sur les vrais fichiers JPEG manquait. Après correction de ces deux points, le périmètre retenu n'a changé que pour la condition JPEG des logos et bandes dessinées.

| Type | 2× | 4× | 8× | 4× entrée propre | 4× grande image | Retenu |
|---|---|---|---|---|---|---|
| Photo | +1.73 | +0.36 | +0.19 | +0.94 | +0.89 | 2× et plus |
| Dessin au trait | +1.22 | +1.08 | +0.32 | +0.42 | +1.67 | 2× et plus |
| UI | +0.84 | +0.36 | +0.15 | +0.37 | +0.22 | 2× et plus |
| Bande dessinée | +2.27 | +0.62 | +0.07 | −0.15 | +0.44 | 2× et plus, entrées JPEG seulement |
| Peinture | +2.37 | +1.46 | +0.25 | +1.50 | +1.51 | 2× et plus |
| Document | −0.48 | +0.55 | +0.22 | +0.58 | +0.55 | 4× et plus seulement |
| Illustration | +0.28 | +0.26 | +0.04 | −0.12 | +0.16 | Non utilisé |
| Logo | +2.09 | +3.04 | +1.57 | +0.18 (1/3 images) | +1.36 | 2× et plus, entrées JPEG seulement |

Pour les logos, la nouvelle mesure sur entrées propres n'améliore qu'une image sur 3, et l'erreur de direction des contours atteint +14,8 %, au-delà de la contrainte (+10 %). Sur les entrées passées par JPEG (dégradation réaliste 2·4·8×, grande image 4×), tout passe ; les valeurs ne sont donc utilisées que pour les fichiers JPEG. Pour les bandes dessinées, la moyenne sur les entrées propres remesurées descend à −0,15 dB, d'où la même condition. Les chiffres des photos et peintures sur entrées propres ont baissé mais passent toujours.

### 9.5 Classifieur

L'apprentissage a porté sur 567 images avec les 27 caractéristiques de `research/classifier` (`features2.py`). Les variantes issues d'un même original sont regroupées et la validation croisée se fait par groupe, afin que différentes versions d'une même image ne se retrouvent pas à la fois dans l'apprentissage et dans le test. La précision en validation croisée par groupe est de **92.9%**. Les poids figurent en JSON intégré dans `AUTO_MODEL` de `upscaler.html`, et les décisions en JS concordent avec Python sur 567/567.

### 9.6 Traitement des couleurs

Avec la règle de consensus de trois estimateurs, la forte dérive de couleur (angle par rapport au neutre) est passée de 13,9° à 4,0° sur 54 paires de test.

### 9.7 Tramage 8 bits

L'erreur de la sortie 8 bits est mesurée par rapport à la sortie 16 bits du même traitement (`tests/dither_probe.js`). L'indicateur de bandes est la RMS de l'erreur après un flou gaussien σ2 (en pixels du résultat). Les bandes sont une erreur basse fréquence étalée sur de grandes surfaces et survivent au flou, alors que le grain fin du tramage disparaît quand on floute. 20 cas : un dégradé de ciel synthétique et 9 fragments de photos, peintures et illustrations de validation de la restauration poussée, agrandis 4 et 8 fois.

| Indicateur (moyenne de 20 cas, en niveaux) | Arrondi | Tramage | Variation |
|---|---|---|---|
| Indicateur de bandes σ2 | 0.059 | 0.033 | −44% (environ −50 % à 8×) |
| Indicateur de bandes σ4 | 0.040 | 0.017 | −58% |
| RMS globale | 0.245 | 0.340 | +39% |

La RMS globale augmente à hauteur du bruit fin ajouté par le tramage. Même avec le tramage activé, la sortie au plus proche voisin 3× ne présente aucune valeur divergente, et l'écart maximal R·G·B d'une entrée en noir et blanc est de 0.

### 9.8 Restauration de texture

Sur les données de restauration poussée (vérité de 256 px → réduction 4 ou 8 fois → nouvel agrandissement), la mesure porte sur l'énergie de la bande au-dessus de la fréquence de Nyquist de l'original (différence gaussienne σ = 0,375·facteur) (`research/texture/tex_probe.js`, résultats `research/results/texture*.json`).

- **Rapport d'énergie haute fréquence R** = résultat / vérité. L'agrandissement ne peut pas créer cette bande ; R est donc bien inférieur à 1.
- **Erreur de carte de texture** = RMS du rapport logarithmique de l'énergie haute fréquence par blocs 16×16. Elle indique si la texture se trouve là où elle doit être.
- **PSNR·SSIM.** Une texture inventée ne coïncide pas en position avec la vérité ; il est donc normal qu'ils baissent.

**Ajustement de la constante.** Sur 9 cas, 3 photos de recherche × (entrée propre 4×, dégradation réaliste 4× et 8×), la constante donnant R = 1 a été recherchée par la méthode de la sécante, et la médiane des 8 cas où R a pu être atteint, $`C = 6.256`$, a été fixée à 100 % (5,915 en v1.1.0 ; elle a un peu augmenté depuis que l'éclaircissement est limité par la marge des canaux). Les réglages sont les préréglages et valeurs de recherche choisis, comme 「Auto」, en classant l'entrée réduite. Les valeurs sont très dispersées d'une image à l'autre (2,4 à 12), car la quantité de texture varie selon les photos et une seule bande de l'original ne permet pas d'estimer exactement l'énergie manquante. Vérifié sur les images de vérité, le produit « deuxième bande de l'original × nombre d'octaves » était le plus exact sur les photos parmi les quatre formules de prédiction testées (énergie ×÷1,45).

**Validation (3 photos de validation × 4 conditions = 12 cas).**

| Intensité | R | Erreur de carte de texture | Cas améliorés sur les deux | PSNR | SSIM |
|---|---|---|---|---|---|
| Désactivée | 0.11 | 1.04 | | | |
| 25% | 0.27 | 0.70 | 12/12 | −0.39 dB | −0.05 |
| 100% | 1.22 | 0.57 | 12/12 | −2.24 dB | −0.18 |

**Autres types.** Valeurs mesurées avec la même constante (`research/results/texture.log`, `texture_25.log`). Les réglages sont les mêmes que pour 「Auto」 : l'entrée réduite est classée pour choisir le préréglage, et les valeurs de la recherche de restauration poussée sont lues directement dans `AUTO_BIG` de l'application et utilisées uniquement dans les conditions de type, de facteur et de JPEG correspondantes (la mesure de la v1.1.0 choisissait le préréglage à partir de l'image de vérité et de la classe des données, et utilisait les valeurs de recherche même pour les illustrations en entrée JPEG, ce qui différait de l'application réelle).

| Type (cas) | Intensité | R | Erreur de carte de texture | Cas améliorés sur les deux | PSNR |
|---|---|---|---|---|---|
| Peinture (12) | 25% | 0.18 → 0.78 | 0.88 → 0.41 | 12/12 | −0.63 dB |
| Peinture (12) | 100% | 0.18 → 8.51 | 0.88 → 0.94 | 7/12 | −4.50 dB |
| Illustration (12) | 25% | 0.07 → 0.11 | 1.12 → 1.01 | 10/12 | −0.12 dB |
| Illustration (12) | 100% | 0.07 → 0.41 | 1.12 → 0.99 | 6/12 | −1.02 dB |
| Graphisme en aplats (24) | 25% | 0.27 → 0.28 | 0.67 → 0.65 | 14/24 | −0.04 dB |
| Graphisme en aplats (24) | 100% | 0.27 → 0.63 | 0.67 → 0.56 | 12/24 | −0.94 dB |

Pour les peintures, tout s'améliore à 25 %, mais à 100 % R déborde fortement (8,5 fois) et l'erreur de carte de texture se dégrade au contraire. Les peintures au pinceau ont une forte énergie dans la bande intermédiaire, et la constante ajustée sur les photos devient excessive. Pour les illustrations, 10/12 s'améliorent à 25 %, mais R ne passe que de 0,07 à 0,11 : l'effet est faible. Pour les graphismes en aplats (logos, UI, dessins au trait, documents), à peine plus de la moitié s'améliore. Ces deux catégories ne sont pas recommandées.

**Résultat à l'œil.** Les chiffres sont plus proches de la vérité à 100 %, mais sur les images de comparaison (`research/results/texture_samples/`, de gauche à droite : vérité, texture désactivée, texture activée), 100 % ressemble le plus souvent à du bruit. Dans les scènes à texture dense (moteur de moto, pylône), des points fins sont visibles même à 25 %. Sur les surfaces lisses comme les visages et la peau, 20 à 30 % ajoutent une texture discrète. C'est pourquoi le texte d'aide recommande 20 à 30 % pour les photos et peintures.

### 9.9 Display P3

La mesure se fait dans un vrai Chromium (Playwright) (`tests/p3_browser.js`). Des patchs de primaires P3 sont dessinés sur un canevas display-p3 de Chromium et convertis en JPEG et PNG (l'encodeur du navigateur joint un profil P3) ; ils sont chargés dans l'application, agrandis 2× → enregistrés en PNG → le fichier enregistré est relu sur un canevas P3 et on examine la couleur au centre des patchs.

| Patch (valeurs P3) | Résultat enregistré v1.0.0 | Résultat enregistré v1.1.0 |
|---|---|---|
| Rouge 255, 0, 0 | 234, 51, 35 | 255, 0, 0 (entrée JPEG 254, 0, 0) |
| Vert 0, 255, 0 | 117, 251, 76 | 0, 255, 0 (entrée JPEG 0, 255, 1) |
| Gris 128, 128, 128 | 127, 127, 127 | 127, 127, 127 |

Le gris 127 est la valeur de l'image de test elle-même. Le profil ICC Display P3 généré par l'application a aussi été vérifié avec LittleCMS (Pillow ImageCms) : la conversion en sRGB de P3 (200, 100, 50) et (60, 180, 220) donne les mêmes valeurs que le calcul par la matrice des primaires. Les JPEG et PNG avec profil sRGB et les fichiers du jeu de données sans profil ont été détectés comme sRGB, et les images de test P3 ainsi que les fichiers enregistrés par la v1.1.0 comme gamut large.

### 9.10 Netteté perçue

Les 23 images de validation (pixel art exclu) × 3 conditions (4× propre, 4× dégradé, 8× dégradé) = 70 cas sont agrandies avec les mêmes réglages que 「Auto」, en comparant 「Précision」 et 「Perception」 (`research/pop/pop_probe.js`, `research/results/pop_val.json`). L'erreur est le $`|\ln A| + |\ln C|`$ de [4.21](#421-netteté-perçue), qui compte à égalité le fait d'être plus émoussé que la vérité terrain et celui de la dépasser. Le halo est la quantité moyenne dont le résultat dépasse la plage du voisinage de la vérité terrain (rayon = facteur) autour de ses contours (à moins de 2× le facteur) ; le bruit sur les aplats est le RMS haute fréquence du résultat là où la vérité terrain est plate (30 % de pente la plus faible).

| Perception 100 % | Erreur (Précision → Perception) | Améliorés | Pente des contours A | Contraste local C | Halo | Bruit sur les aplats | PSNR | SSIM |
|---|---|---|---|---|---|---|---|---|
| Photos, 9 cas | 0,303 → 0,226 | 6/9 | 0,849 → 0,949 | 0,967 → 1,088 | +0,16 niveau | ×1,09 | −0,22 dB | −0,0003 |
| Peintures · illustrations · manga, 26 cas | 0,676 → 0,529 | 20/26 | 0,731 → 0,825 | 0,865 → 0,983 | +0,12 niveau | ×1,16 | −0,16 dB | +0,006 |
| Dessin au trait · documents · UI · logos, 35 cas | 0,621 → 0,559 | 20/35 | 0,865 → 0,949 | 0,982 → 1,078 | +0,52 niveau | ×0,95 | −0,06 dB | +0,012 |
| Ensemble, 70 cas | 0,600 → 0,505 | 46/70 | 0,813 → 0,903 | 0,937 → 1,044 | +0,33 niveau | ×1,05 | −0,12 dB | +0,008 |

| Intensité (ensemble, 70 cas) | Erreur | Améliorés | Halo | Bruit sur les aplats | PSNR | SSIM |
|---|---|---|---|---|---|---|
| 50 % | 0,526 | 49/70 | +0,22 niveau | ×1,00 | −0,06 dB | +0,006 |
| 100 % | 0,505 | 46/70 | +0,33 niveau | ×1,05 | −0,12 dB | +0,008 |
| 150 % | 0,497 | 46/70 | +0,40 niveau | ×1,09 | −0,17 dB | +0,008 |

La pente des contours et le contraste local montent vers la vérité terrain, tandis que le halo et le bruit sur les aplats n'augmentent presque pas. Photos et images s'améliorent dans au moins deux tiers des cas, mais le dessin au trait, les documents, l'UI et les logos dans 20/35, à peine plus de la moitié : leurs contours sont déjà raides et le contraste local dépasse facilement la vérité terrain. Les images de comparaison sont dans `research/results/pop_samples/` (de gauche à droite : vérité terrain · Précision · Perception 50 % · 100 % · 150 % ; 4× dégradé, et une vérité terrain de 500 px réduite de moitié puis agrandie 2×).

Au départ, 「Perception」 ajoutait aussi un grain fin (restauration de texture, 25 % à l'intensité 100 %). La mesure de texture s'améliorait (rapport d'énergie haute fréquence R 0,13 → 0,30), mais le bruit sur les aplats était multiplié par 4,0 et le SSIM baissait de 0,044 (`pop_val_grain.json`), avec des points visibles sur les surfaces lisses. 「Perception」 n'augmente donc que les contours et le contraste local ; le grain fin se règle séparément avec le curseur 「Restauration de texture」.

---

## 10. Points forts

**Une restauration honnête envers l'information.** À l'étape de restauration, aucun nouvel extremum n'est créé (plage par cellule), et l'image est ajustée pour redonner l'original une fois réduite (cohérence). Les corrections par-dessus ne sont ajoutées que dans la mesure où l'utilisateur les active.

**Un Auto qui connaît le type d'image.** Les décisions reposent sur une classification en 9 classes (92,9 %), une recherche dans 1 520 milliards de réglages et des mesures faites sur chaque image. Les raisons de chaque choix sont indiquées à l'écran.

**Une restauration achevée à la résolution d'origine.** La restauration DCT qui supprime le bruit en mesurant son spectre, la restauration JPEG qui respecte la table de quantification du fichier et la déconvolution RL sont effectuées avant l'agrandissement.

**Des graphismes aux traits vivants.** Le lissage directionnel et le retracé par distance signée produisent des traits nets sans marches. Pour le pixel art, chaque sous-échantillon ne prend que des couleurs de la palette d'origine, et les nouvelles couleurs n'apparaissent que dans l'anticrénelage des contours.

**Des réglages utilisateur honnêtes.** Les cases et curseurs font exactement ce que vous choisissez (pour l'exception juste après l'ouverture d'une image, voir [11. Limites](#11-limites)). L'automatisme est réservé au bouton 「Auto」, et le voyant se contente d'indiquer le résultat de la mesure.

**Pas limité par la taille.** Grâce au cache de tuiles et à la compression PNG par lignes de tuiles, des résultats de plusieurs centaines de millions de pixels sont enregistrés sans charger en mémoire le résultat complet non compressé. Le facteur automatique ne choisit que des facteurs qui peuvent être traités jusqu'au bout selon le budget mémoire.

**Écran = enregistrement.** Avec les mêmes noyaux, marges et grilles globales, ce que vous voyez à l'écran est enregistré tel quel.

**Du 8 bits sans bandes.** Le tramage déterminé par les coordonnées réduit l'indicateur de bandes de près de moitié, tout en laissant intacts les couleurs exactes, les gris et les jointures entre tuiles.

**Les couleurs du téléphone préservées.** Le profil de l'original est lu, les photos Display P3 sont traitées en P3, et l'enregistrement porte un profil P3 construit par l'application ainsi qu'un cICP.

**Confidentialité et portabilité.** Ni serveur ni installation, et vos images ne quittent pas votre appareil. Un seul fichier, facile à conserver et à transporter.

---

## 11. Limites

**Rien n'est inventé.** Les détails non enregistrés dans l'original (un cheveu, de petits caractères) ne peuvent pas être récupérés. Comme rien n'est dessiné pour paraître plausible, contrairement aux agrandissements par réseaux de neurones, un original très flou reste flou même fortement agrandi.

**La restauration de texture est du grain.** Elle ajuste la quantité de texture, mais son emplacement et sa forme sont un bruit inventé ; à forte intensité, elle ressemble à du bruit. À 100 %, elle déborde fortement sur les peintures. L'effet est faible sur les illustrations, et les graphismes en aplats ne s'améliorent qu'à moitié environ, quelle que soit l'intensité. Elle n'est conseillée qu'à faible intensité sur les photos et peintures ([9.8](#98-restauration-de-texture)).

**「Perception」 n'ajoute aucune information.** Elle redessine de façon plus visible les contours et les contrastes que l'original contient, sans créer de détail qu'il n'avait pas. Sur des images aux contours déjà raides comme le dessin au trait, les documents, l'UI et les logos, à peine plus de la moitié des cas s'améliorent (20/35). Un résultat flou, par exemple un tout petit original fortement agrandi, reste flou avec 「Perception」 ([9.10](#910-netteté-perçue)).

**Logos et bandes dessinées : entrées JPEG seulement.** Faute d'avoir passé la règle d'adoption sur les entrées propres remesurées, les valeurs de la recherche de restauration poussée ne sont pas utilisées pour les entrées propres comme le PNG. Une image à l'origine en JPEG puis réenregistrée en PNG est considérée comme une entrée propre, même si elle porte des traces JPEG.

**Illustrations non retenues.** Les résultats divergent selon les conditions ; les valeurs de la recherche de restauration poussée ne sont donc pas utilisées. Il reste possible de les réexaminer avec d'autres dégradations (bruit fort, autre flou, originaux nets au départ) et des images plus grandes.

**Portée de la recherche.** La recherche repose sur de petits fragments (entrée de 64 px) et 3 images par type. La quantité d'accentuation, le dépassement, etc. avaient tendance à se coller au bout de leur plage, et élargir la plage n'apportait pas plus de +0,1 dB.

**Un facteur automatique qui ne regarde que la mémoire.** Il n'y a pas de limite de temps de traitement. À 8×, le résultat atteint plusieurs centaines de mégapixels. Le budget mémoire est une estimation de la mémoire de l'appareil, et le 「garanti」(보장) s'entend par rapport à ce budget, car le navigateur n'indique pas précisément la mémoire restante.

**Gradation des entrées 8 bits.** L'entrée est lue en 8 bits en passant par un canevas du navigateur. Le tramage et l'enregistrement 16 bits ne font que préserver les valeurs intermédiaires nouvellement créées par l'agrandissement ; ils ne suppriment pas les bandes déjà présentes dans l'original (surtout à 1× ou sur des surfaces très plates). Sans tramage, l'enregistrement 8 bits est un simple arrondi et les bandes s'élargissent du facteur.

**Gestion des couleurs.** Le gamut large n'est préservé que jusqu'au Display P3. Les originaux contenant des couleurs hors P3, comme en Adobe RGB, BT.2020 ou ProPhoto, sont réduits dans le P3, et le PNG enregistré porte un profil P3, car le canevas ne propose que le sRGB et le Display P3 comme espaces de travail. Cela reste moins réducteur qu'une lecture en sRGB (en surface CIE xy, le triangle BT.2020 est couvert à 53 % par le sRGB et à 72 % par le P3). Préserver le profil d'origine tel quel exigerait de lire sans conversion de couleurs, de traiter dans ce codage et de rejoindre le profil ; c'est laissé à une fonction distincte. Les navigateurs qui ne prennent pas en charge le canevas P3 (par exemple Firefox actuellement) lisent en sRGB comme auparavant. Les images collées sont toujours considérées comme sRGB. Si un AVIF ne place l'information de couleur que dans le flux AV1 (en-tête de séquence), sans boîte `colr`, elle n'est pas lue et l'image est considérée comme sRGB (les encodeurs courants comme libavif écrivent `colr`). Les navigateurs sans `DecompressionStream` (Safari 15.2 à 16.3) ne peuvent pas décompresser le profil iCCP compressé du PNG ; un PNG qui signale son gamut large uniquement par iCCP, sans cICP, est alors considéré comme sRGB. L'analyse de 「Auto」 (classification, mesures détaillées, couleurs auto) demande au navigateur les valeurs sRGB même pour les images P3, ce qui la maintient dans la référence sRGB sur laquelle le classifieur a été entraîné. Le chemin d'enregistrement par canevas des navigateurs sans compression en flux ne joint pas directement de profil P3.

**Hypothèse de taux de compression.** Le facteur automatique compte le taux de compression du PNG enregistré comme le maximum mesuré par facteur sur 47 images, plus une marge de 10 % (en 16 bits avec la texture activée, 0,934 × 1,1, soit 102 % de la taille non compressée). Une image bien plus bruitée que ces 47 images se compresse moins que prévu, et la mémoire peut manquer pendant l'enregistrement même avec un facteur 「garanti」.

**Données manquantes.** Le cache RGBA pour l'apprentissage du classifieur (environ 1,3 Go) et les données d'expérience sur la grille DCCI (environ 1,3 Go) ne sont pas dans le dépôt en raison de leur taille. Tous deux peuvent être régénérés à partir des originaux de `data/ds`.

---

## 12. Historique des versions

### v1.2.2 — Le facteur 「Auto」 se déplace aussitôt

- Appuyer sur 「Auto」 dans la ligne du facteur déplace maintenant aussitôt le facteur vers le plus grand garanti avec les réglages actuels (auparavant, il ne se déplaçait qu'avec le bouton 「Auto」). Le bouton 「Auto」 cherche toujours à nouveau avec les réglages propres à chaque facteur.

### v1.2.1 — Nom et affichage de la version

- Le titre de l'application devient **Upscaly: 무손실 지향 확대기** (Upscaly : agrandisseur d'images visant l'absence de perte), et la version (v1.2.1) s'affiche à côté du nom dans la barre du haut. Sur les écrans étroits (720 px ou moins), seuls « Upscaly » et la version s'affichent.
- Les calculs et les fichiers enregistrés sont identiques à ceux de la v1.2.0.

### v1.2.0 — Deux objectifs : précision et perception

- **「Objectif : Précision · Perception」.** Par défaut, 「Précision」 agrandit au plus près de l'original comme avant (la sortie est identique octet par octet à la v1.1.1). 「Perception」 porte la pente des contours et le contraste local au niveau de la vérité terrain pour une image plus nette à l'œil. Elle ne dépasse jamais la plage de luminosité du voisinage, donc aucun liseré blanc n'apparaît. Sur 70 cas de validation, l'erreur par rapport à la vérité terrain passe de 0,600 à 0,505, avec un bruit sur les aplats ×1,05 et un PSNR de −0,12 dB ([2.9](#29-précision-et-perception-sont-deux-objectifs-différents), [9.10](#910-netteté-perçue)).
- Intensité de perception de 25 à 150 % ; les préréglages et 「Auto」 ne modifient pas ce choix. Le facteur automatique tient compte de la marge, de la mémoire et du taux de compression du PNG enregistré en mode 「Perception」.

### v1.1.1 — Corrections issues de la revue de code

- **Images transparentes agrandies par étapes.** Aux étapes intermédiaires, l'alpha subit aussi les mêmes poids directionnels, la même plage par cellule et la même accentuation entre étapes que la couleur. Avec DCCI 4×, l'erreur de couleur des bords semi-transparents est passée d'un maximum de 19 à 38 niveaux à 1 à 4 niveaux, et les jointures entre tuiles du préréglage logo 4× (jusqu'à 206 niveaux) ont disparu.
- **Valeur initiale de 「Agrandir par étapes」.** Si vous modifiez la case ou cliquez sur 「Auto」 avant la fin de la mesure faite juste après l'ouverture d'une image, la mesure terminée en retard n'écrase plus votre choix.
- **Vérification des canevas vides.** Si le canevas de l'original est entièrement transparent, l'original est lu directement par fragments pour vérifier s'il contient des pixels visibles, et l'enregistrement par canevas recherche un pixel visible du résultat et vérifie cette position. La limite du canevas est détectée même si le coin inférieur droit est transparent à l'origine ou si un seul point est visible.
- **Couleurs claires de la restauration de texture.** Le rapport d'éclaircissement est limité par la marge du plus grand canal, si bien que la teinte des couleurs claires et saturées ne change plus. La constante a été réajustée en conséquence (5,915 → 6,256), et les mesures ont été refaites après avoir été ramenées aux mêmes conditions que 「Auto」. Le débordement d'un facteur 1,8 de la texture en mode gamma, avec 「Calculer en lumière linéaire」 désactivé, a aussi été corrigé.
- **Détection du gamut.** Même si plus de 2 Mo de métadonnées ou de boîtes (XMP étendu JPEG, chunks texte PNG, mdat AVIF avant meta) précèdent l'information de couleur, les en-têtes sont suivis pour la trouver. Pour l'AVIF, c'est l'information de couleur (colr) rattachée à l'image principale qui est examinée, et non celle d'une vignette ou d'une image auxiliaire ; pour le PNG, cICP est examiné en premier quel que soit l'ordre des chunks, mais si les primaires sont non spécifiées (2), la détection se fait par le profil iCCP. Les fichiers à gamut large sont lus via un `<img>` en Data URL même si la Blob URL est bloquée, ce qui préserve leurs couleurs.
- **Budget après 「Auto」.** 「Auto」 applique, en partant des grands facteurs, les réglages 「Auto」 de chaque facteur et choisit le premier facteur garanti. Le cas où, avec le plus proche voisin et la texture activée, 1200×900 était agrandi à 8× et manquait de mémoire pour la préparation de l'original a disparu. Le cas où l'on restait sur un facteur plus petit lorsque les réglages se répartissaient entre options bon marché et coûteuses selon le facteur a aussi disparu (image UI 4000×3000, facteur actuel 1.25×, agrandissement en trois étapes : 「Aucune garantie」 → 2×).
- **Calcul mémoire du facteur automatique.** Le PNG compressé qui s'accumule pendant l'enregistrement (taux de compression maximal par facteur mesuré en agrandissant 47 images du jeu de données de 1,1 à 4×, plus une marge de 10 %) et les autres originaux préparés restés dans le cache sont comptés. Le facteur choisi peut donc baisser pour les grandes photos (avec un budget de 600 Mo, 4000×3000 donne 2× et non 4×). Changer le nombre de bits d'enregistrement ou la texture recalcule aussi le message d'aide.

### v1.1.0 — 8 bits sans bandes, couleurs du téléphone, restauration de texture, nouvelle validation des logos

- **Tramage 8 bits (activé par défaut).** Lors de la conversion en 8 bits, un tramage fin déterminé par les coordonnées est ajouté, ce qui réduit les bandes dans les dégradés fortement agrandis (indicateur de bandes −44 %). Il peut être désactivé avec la case 「Tramage 8 bits」(8비트 디더링). La sortie 16 bits est identique à l'octet près à celle de la v1.0.0.
- **Préservation du Display P3.** Si le profil colorimétrique du fichier d'origine est plus large que le sRGB, l'image est lue sur un canevas P3, l'écran est aussi dessiné en P3, et le PNG enregistré porte un profil Display P3 (cICP·iCCP). Les rouges et verts P3 ne sont plus écrasés comme auparavant.
- **Restauration de texture (désactivée par défaut).** Un nouveau curseur ajoute la texture fine manquante en fonction de l'intensité de texture de l'original. 20 à 30 % sont conseillés pour les photos et peintures.
- **Nouvelle validation des logos et bandes dessinées.** L'erreur de validation sur entrées propres a été corrigée et les mesures refaites. Les logos utilisent désormais les valeurs de la recherche de restauration poussée pour les entrées JPEG, et les bandes dessinées ne les utilisent plus que pour les entrées JPEG.
- Le README a été entièrement réécrit sous forme d'article.

### v1.0.0 — Première version transférée vers Claude Code

- Le projet développé localement avec Claude a été transféré dans le dépôt. L'application (`upscaler.html`), la version de référence, les tests, les scripts et résultats de recherche et les données (567 originaux, fragments de recherche) ont été versés tels quels.
- Les résultats de conversion sont identiques à l'octet près à la version de référence sur 134 combinaisons.

---

## 13. Références

1. D. Zhou, X. Shen, W. Dong. "Image zooming using directional cubic convolution interpolation." *IET Image Processing* 6(6), 2012.
2. R. Keys. "Cubic convolution interpolation for digital image processing." *IEEE Trans. ASSP* 29(6), 1981.
3. D. P. Mitchell, A. N. Netravali. "Reconstruction filters in computer graphics." *SIGGRAPH* 1988.
4. C. E. Duchon. "Lanczos filtering in one and two dimensions." *J. Applied Meteorology* 18(8), 1979.
5. W. H. Richardson. "Bayesian-based iterative method of image restoration." *JOSA* 62(1), 1972.
6. L. B. Lucy. "An iterative technique for the rectification of observed distributions." *Astronomical Journal* 79, 1974.
7. C. Tomasi, R. Manduchi. "Bilateral filtering for gray and color images." *ICCV* 1998.
8. K. Dabov, A. Foi, V. Katkovnik, K. Egiazarian. "Image denoising by sparse 3-D transform-domain collaborative filtering." *IEEE TIP* 16(8), 2007.
9. A. Zakhor. "Iterative procedures for reduction of blocking effects in transform image coding." *IEEE TCSVT* 2(1), 1992.
10. Y. Yang, N. P. Galatsanos, A. K. Katsaggelos. "Regularized reconstruction to reduce blocking artifacts of block discrete cosine transform compressed images." *IEEE TCSVT* 3(6), 1993.
11. K. He, J. Sun, X. Tang. "Guided image filtering." *IEEE TPAMI* 35(6), 2013.
12. J. Immerkær. "Fast noise variance estimation." *CVIU* 64(2), 1996.
13. V. A. Marchenko, L. A. Pastur. "Distribution of eigenvalues for some sets of random matrices." *Mat. Sbornik* 72(4), 1967.
14. J. Bigün, G. H. Granlund. "Optimal orientation detection of linear symmetry." *ICCV* 1987.
15. R. Achanta, S. Hemami, F. Estrada, S. Süsstrunk. "Frequency-tuned salient region detection." *CVPR* 2009.
16. Z. Wang, A. C. Bovik, H. R. Sheikh, E. P. Simoncelli. "Image quality assessment: from error visibility to structural similarity." *IEEE TIP* 13(4), 2004.
17. G. D. Finlayson, E. Trezzi. "Shades of gray and colour constancy." *Color and Imaging Conference* 2004.
18. J. van de Weijer, T. Gevers, A. Gijsenij. "Edge-based color constancy." *IEEE TIP* 16(9), 2007.
19. H. H. Bauschke, J. M. Borwein. "On projection algorithms for solving convex feasibility problems." *SIAM Review* 38(3), 1996.
20. L. H. Thomas. "Elliptic problems in linear difference equations over a network." Watson Sci. Comput. Lab. Report, Columbia University, 1949.
21. I. Kutskir. "Fastest Gaussian blur (in linear time)." 2013. Article de blog.
22. IEC 61966-2-1:1999. Multimedia systems and equipment — Colour measurement and management — Part 2-1: Default RGB colour space — sRGB.
23. J. Jimenez. "Next generation post processing in Call of Duty: Advanced Warfare." Conférence SIGGRAPH 2014 Advances in Real-Time Rendering. (Bruit à gradient entrelacé)
24. D. J. Field. "Relations between the statistics of natural images and the response properties of cortical cells." *JOSA A* 4(12), 1987.
25. A. van der Schaaf, J. H. van Hateren. "Modelling the power spectra of natural images: statistics and information." *Vision Research* 36(17), 1996.
26. International Color Consortium. "ICC.1:2022 Image technology colour management — Architecture, profile format, and data structure." 2022.
27. W3C. "Portable Network Graphics (PNG) Specification (Third Edition)." 2025. (Chunk cICP)
28. SMPTE EG 432-1:2010. Digital Source Processing — Color Processing for D-Cinema. (Primaires P3)
29. R. A. Ulichney. *Digital Halftoning*. MIT Press, 1987.
