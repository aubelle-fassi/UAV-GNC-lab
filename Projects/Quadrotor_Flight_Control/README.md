# Contrôle de vol d'un quadricoptère

## Présentation

Ce projet porte sur l'étude de la modélisation, de l'analyse, de la
commande et de l'estimation d'état d'un quadricoptère évoluant dans un
plan vertical.

Le travail a été réalisé sous MATLAB et Simulink, en considérant d'abord
le modèle dynamique non linéaire, puis plusieurs modèles linéarisés autour
de différents points d'équilibre.

L'objectif est d'étudier progressivement la chaîne de contrôle et
d'estimation d'un système aéronautique autonome.

## Objectifs

- Modéliser le comportement dynamique non linéaire du quadricoptère.
- Linéariser le modèle autour de différents points d'équilibre.
- Étudier la commandabilité et l'observabilité du système.
- Concevoir un observateur de Luenberger par placement de pôles.
- Implémenter un observateur non linéaire sous Simulink.
- Discrétiser le modèle pour une étude en temps discret.
- Implémenter un filtre de Kalman étendu (EKF).
- Concevoir une commande par retour d'état.
- Étudier l'ajout d'une action intégrale.
- Valider les différentes approches par simulation.

---

## 1. Modèle dynamique

Le système étudié est un modèle simplifié d'un quadricoptère évoluant
dans un plan vertical.

Le vecteur d'état est défini par :
Le vecteur d'état est défini par :

$$
X =
\begin{bmatrix}
x & z & v_x & v_z & \theta & \dot{\theta}
\end{bmatrix}^{T}
$$

avec :

$$
\begin{aligned}
- \(x\) : position horizontale ;
- \(z\) : position verticale ;
- \(v_x\) : vitesse horizontale ;
- \(v_z\) : vitesse verticale ;
- \(\theta\) : angle d'attitude ;
- \(\dot{\theta}\) : vitesse angulaire.
  \end{aligned}
$$

Après transformation des variables de commande, la dynamique utilisée
dans le projet est donnée par :

$$
\begin{aligned}
\dot{x} &= v_x \\
\dot{z} &= v_z \\
\dot{v}_x &= c\*v_2\cos(\theta)-v_1\sin(\theta) \\
\dot{v}_z &= c\*v_2\sin(\theta)+v_1\cos(\theta)-g \\
\dot{\theta} &= \dot{\theta} \\
\ddot{\theta} &= v_2
\end{aligned}
$$
## 2. Linéarisation du modèle

Afin d'étudier le comportement local du système, le modèle non linéaire
est linéarisé autour de 03 points d'équilibre : 0°, 10°, 20°


Pour  le modèle linéarisé s'écrit :

\[
\delta\dot{X}
=
A_0\delta X+B_0\delta U
\]

avec :

\[
A_0=
\begin{bmatrix}
0&0&1&0&0&0\\
0&0&0&1&0&0\\
0&0&0&0&-9.81&0\\
0&0&0&0&0&0\\
0&0&0&0&0&1\\
0&0&0&0&0&0
\end{bmatrix}
\]

et

\[
B_0=
\begin{bmatrix}
0&0\\
0&0\\
0&0.0007\\
1&0\\
0&0\\
0&1
\end{bmatrix}
\]

Les matrices obtenues pour \(10^\circ\) et \(20^\circ\) montrent que la
dynamique locale dépend de l'angle d'équilibre.

Cette étude permet notamment d'observer l'évolution du couplage entre
les mouvements horizontaux, verticaux et l'attitude lorsque le point
de fonctionnement change.

---

## 3. Analyse de la commandabilité et de l'observabilité

La commandabilité du système est étudiée à l'aide de la matrice de
Kalman :

\[
\mathcal{C}
=
\begin{bmatrix}
B & AB & A^2B & \cdots & A^{n-1}B
\end{bmatrix}
\]

L'observabilité est également étudiée à partir des mesures disponibles.

Dans le cadre de ce projet, les sorties utilisées pour l'estimation
sont les positions :

\[
Y=
\begin{bmatrix}
x\\
z
\end{bmatrix}
\]

L'étude montre que l'utilisation conjointe des mesures \(x\) et \(z\)
permet de reconstruire l'ensemble des six états du modèle.

---

## 4. Observateur de Luenberger

Un observateur d'état de Luenberger est conçu à partir du modèle
linéarisé autour de \(\theta_e=0^\circ\).

L'équation de l'observateur est :

\[
\dot{\hat{X}}
=
A\hat{X}
+
BU
+
L(Y-\hat{Y})
\]

avec :

\[
\hat{Y}=C\hat{X}
\]

Le gain \(L\) est obtenu par placement de pôles.

Dans le projet, les pôles de l'observateur sont choisis à :

\[
-2,\;-4,\;-6,\;-8,\;-10,\;-12
\]

L'observateur permet ensuite d'estimer les états non directement mesurés
à partir des positions disponibles.

---

## 5. Observateur non linéaire

L'observateur conçu à partir du modèle linéarisé est ensuite intégré
dans une architecture utilisant directement la dynamique non linéaire
du quadricoptère.

L'objectif est de comparer les états réels du modèle avec les états
reconstruits par l'observateur.

Cette étape permet de vérifier le comportement de l'estimation lorsque
le système est décrit par sa dynamique non linéaire.

### Modèle Simulink

Le modèle correspondant est disponible dans :

`observateur_non_lineaire.slx`

---

## 6. Passage au temps discret

Le modèle linéarisé est discrétisé avec une période d'échantillonnage :

\[
T_e=0.01\;s
\]

La discrétisation est réalisée avec la méthode `ZOH` :

```matlab
syst_dis = c2d(sys0, Te, 'zoh');
7. Filtre de Kalman étendu

Un filtre de Kalman étendu (EKF) est implémenté afin d'estimer les six
états du quadricoptère à partir des mesures de position \(x\) et \(z\).

À chaque période d'échantillonnage, l'algorithme réalise :

la prédiction de l'état à partir du modèle non linéaire ;
le calcul du Jacobien du modèle ;
la prédiction de la covariance ;
le calcul du gain de Kalman ;
le calcul de l'innovation ;
la correction de l'état estimé.
Résultats de l'estimation des etats à partir de l'EKF

Les résultats permettent de comparer les états estimés par l'EKF avec
les états issus du modèle Simulink.

Par exemple, la vitesse angulaire \(\dot{\theta}\) est comparée entre la
valeur réelle et la valeur reconstruite par l'EKF.

La figure met en évidence une bonne tendance générale de l'estimation,
mais également un écart entre la vitesse angulaire réelle et son
estimation sur une partie de la simulation.


Cette différence constitue un point d'analyse intéressant pour améliorer
le réglage du filtre et les hypothèses du modèle.
