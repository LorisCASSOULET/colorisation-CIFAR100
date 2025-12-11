# Colorisation d'Images avec Deep Learning (CIFAR-100)

Ce projet explore l'application du Deep Learning pour la colorisation automatique d'images en noir et blanc. L'objectif est d'entraîner un réseau de neurones capable de prédire les informations de couleur (chrominance) manquantes à partir d'une image en niveaux de gris (luminance).

## Contexte et Objectifs

Le but est de passer d'une image en niveaux de gris (input) à une image colorisée (output) plausible.

Pour ce faire, nous utilisons le jeu de données **CIFAR-100**. Plutôt que de travailler dans l'espace RGB classique, nous convertissons les images dans l'espace colorimétrique **CIE L*a*b*** :
* **L (Lightness) :** Luminosité (l'image en noir et blanc). C'est notre **Input (X)**.
* **a et b :** Les axes de couleurs (Vert-Rouge et Bleu-Jaune). C'est notre **Target (Y)** à prédire.


## Architectures des Modèles

Deux approches ont été testées et comparées.

### 1. CNN Simple
Un réseau de neurones convolutionnel "maison" séquentiel.
* **Structure :** Alternance de couches de convolution (`Conv2D`) pour l'extraction de features et de sur-échantillonnage (`UpSampling2D`) pour reconstruire la dimension spatiale.
* **Entrée :** Image 32x32x1 (Canal L).
* **Sortie :** Image 32x32x2 (Canaux AB) avec activation `tanh`.

### 2. Transfer Learning (VGG16)
Utilisation d'un encodeur pré-entraîné sur ImageNet pour capturer des caractéristiques sémantiques complexes.
* **Encodeur :** VGG16 (poids figés, `include_top=False`).
    * *Adaptation :* L'entrée en niveaux de gris est dupliquée 3 fois pour simuler une image RGB compatible avec VGG16.
* **Décodeur :** Partie personnalisée composée de couches `UpSampling2D` et `Conv2D` pour projeter les caractéristiques extraites vers l'espace couleur.
* **Avantage :** VGG16 est excellent pour l'extraction de textures et sa taille d'entrée est plus flexible que d'autres modèles (ResNet, etc.).

## Métriques et Fonction de Perte

* **Loss :** MSE (Mean Squared Error) pour la précision pixel par pixel.
* **Métrique :** **SSIM (Structural Similarity Index)**.
    * *Pourquoi la SSIM ?* Contrairement à la MSE qui favorise le "moyennage" (produisant des images grisâtres/sépia), la SSIM mime la perception humaine. Elle analyse des fenêtres de pixels pour s'assurer que la structure et le contraste de la couleur prédite correspondent à l'objet de l'image.



## Résultats et Analyse

L'analyse comparative visuelle effectuée sur le jeu de test met en évidence un écart de performance significatif entre les deux approches. Le modèle **CNN Simple** montre rapidement ses limites : les images générées sont souvent ternes et désaturées, donnant un aspect délavé ou sépia. Ce réseau tend à apprendre des moyennes statistiques basiques (comme associer globalement le vert à la végétation ou le bleu au ciel) mais échoue à capturer les nuances vives ou à identifier des objets spécifiques complexes, résultant en une colorisation superficielle.

À l'opposé, l'approche par **Transfer Learning avec VGG16** offre des résultats nettement supérieurs. Les colorisations sont vibrantes et beaucoup plus fidèles à la réalité, avec une saturation marquée (par exemple, un fruit orange apparaît réellement orange plutôt que jaune pâle). Ce modèle parvient non seulement à mieux distinguer les différents éléments sémantiques d'une scène (séparation nette entre ciel, végétation et bâtiments), mais aussi à rendre les textures avec plus de réalisme, notamment sur les pelages d'animaux. 
