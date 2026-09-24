# Planification de calendriers sportifs par programmation linéaire en nombres entiers

Construire le calendrier d'une compétition de football sous contraintes (rencontres aller/retour, alternance domicile/extérieur), le modéliser en PLNE, puis le résoudre avec **PuLP** et le solveur **CBC**.

Ce travail part des exercices du module *Gestion de projet et ordonnancement* (ENSA Béni Mellal, filière Transformation Digitale Industrielle). J'ai écrit moi-même les formulations, le code et les vérifications.

## Les trois problèmes

| Notebook | Problème | Taille | Résultat |
|---|---|---|---|
| [01_phase_de_groupes](notebooks/01_phase_de_groupes.ipynb) | 4 équipes, chacune se rencontre une fois, 3 journées | 36 variables | **6** calendriers (liste complète) |
| [02_aller_retour](notebooks/02_aller_retour.ipynb) | 4 équipes, aller-retour sur 6 journées, pas de double match à domicile ou à l'extérieur sur les journées (1,2), (3,4), (5,6) | 72 variables | **96** calendriers (liste complète, vérifiée par force brute) |
| [03_championnat_16_equipes](notebooks/03_championnat_16_equipes.ipynb) | Championnat à 16 équipes, 30 journées, aller puis retour, jamais plus de 2 matchs de suite au même endroit | 7 200 variables | calendriers valides trouvés en ~2 min chacun  |

## Modélisation (championnat à 16 équipes)

Variable de décision :

$$x_{ijk} = \begin{cases} 1 & \text{si l'équipe } i \text{ reçoit l'équipe } j \text{ lors de la journée } k \\ 0 & \text{sinon} \end{cases}$$

On cherche des calendriers valides, pas un calendrier « meilleur » que les autres : c'est un problème de **faisabilité**, avec l'objectif $\min 0$.

| | Contrainte | Signification |
|---|---|---|
| C1 | $\sum_{k} x_{ijk} = 1 \quad \forall i \neq j$ | le match « i reçoit j » a lieu une fois |
| C2 | $\sum_{j \neq i} (x_{ijk} + x_{jik}) = 1 \quad \forall i, k$ | chaque équipe joue un match par journée |
| C3 | $\sum_{k=1}^{15} (x_{ijk} + x_{jik}) = 1 \quad \forall i < j$ | chaque paire se rencontre pendant l'aller (le retour se déduit de C1 et C3) |
| C4 | $\sum_{j \neq i} \sum_{t=k}^{k+2} x_{ijt} \le 2 \quad \forall i,\ k \le 28$ | jamais 3 matchs de suite à domicile |
| C5 | $\sum_{j \neq i} \sum_{t=k}^{k+2} x_{jit} \le 2 \quad \forall i,\ k \le 28$ | jamais 3 matchs de suite à l'extérieur |

## Énumérer les solutions : les coupes d'exclusion

Pour lister les calendriers un par un, on résout le modèle, on enregistre la solution, puis on ajoute une contrainte qui l'interdit. Si $S$ est l'ensemble des matchs de la solution trouvée :

$$\sum_{(i,j,k) \in S} x_{ijk} \le |S| - 1$$

La solution déjà trouvée ne passe plus, mais toute solution qui diffère d'au moins un match reste possible. On recommence jusqu'à ce que le solveur réponde « infaisable » : on a alors toutes les solutions.

Cette méthode marche jusqu'au bout sur les petits cas (6, 96 et 528 calendriers). Pour 16 équipes, elle reste correcte mais n'est pas utilisable en pratique : rien qu'en renommant les équipes d'un calendrier valide, on obtient $16! \approx 2 \cdot 10^{13}$ calendriers.

## Un point de l'énoncé : « ni 2 matchs successifs à l'extérieur »

Pris au pied de la lettre (EE interdit), l'énoncé rend le problème **impossible**. Avec 15 matchs à l'extérieur sur 30 journées et jamais deux de suite, il n'existe que 16 suites domicile/extérieur possibles. Deux équipes qui ont la même suite ne peuvent jamais se rencontrer, donc les 16 équipes utilisent les 16 suites. Or 15 de ces suites commencent par un match à l'extérieur : il y aurait 15 équipes à l'extérieur à la journée 1, au lieu de 8.

Le solveur le confirme : le modèle avec cette lecture est infaisable dès 4 équipes. J'ai donc retenu la lecture symétrique, pas plus de 2 matchs de suite ni à domicile ni à l'extérieur. Le détail est dans le notebook 3.

## Vérifications

- Chaque nombre de solutions (6, 96, 528) est recompté par une énumération en force brute, indépendante du solveur.
- Le calendrier trouvé pour 16 équipes est vérifié contrainte par contrainte par une fonction `verifier()`.

## Lancer le projet

```bash
pip install -r requirements.txt
jupyter notebook
```

Les notebooks 1 et 2 s'exécutent en quelques secondes. Le notebook 3 prend environ 2 minutes pour trouver un calendrier à 16 équipes.

## Pistes d'amélioration

- Ajouter une vraie fonction objectif : minimiser le nombre de doubles matchs à domicile ou à l'extérieur (*breaks*), ou les distances de déplacement.
- Casser les symétries (fixer les matchs de la première journée) pour accélérer la résolution.
- Comparer CBC avec un solveur de programmation par contraintes (OR-Tools CP-SAT).

Ces mêmes techniques s'appliquent à l'ordonnancement industriel : affecter des tâches à des machines ou des équipes à des postes sur des créneaux, sous contraintes de capacité et d'enchaînement.

---

**Chaimaa Daanoun** · Élève ingénieure en Transformation Digitale Industrielle, ENSA Béni Mellal
