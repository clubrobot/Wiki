# Abstract Move Strategy

Cette page a pour vocation de presenter le fonctionnement des AbstractMoveStrategy mise en place sur la base roulante. Avant de commencer, il faut bien comprendre que le rôle d’une AbstractMoveStrategy est de générer en temps reel des consignes de vitesses pour les moteurs, afin de pouvoir diriger le robot vers la position souhaitée. Pour atteindre cette mission, l’AbstractMoveStrategy dispose en temps réel de la position du robot. Le but pour créer une AbstractMoveStrategy est définir des algorithmes les plus simples possible pour définir des vitesses associées.

## TurnOnTheSpot

TurnOnTheSpot a pour but de faire tourner le robot sur place. Cela n’a l’air de rien mais si on se contente de mettre une simple vitesse angulaire, le robot va naturement diverger de sa position initiale. Cela est dû aux imperfections du robot et ne peux pas être corigé par un simple asservissement en vitesse linéaire à 0.

Pour résoudre le problème, TurnOnTheSpot utilise un algorithme simple. Le principe de cet algorithme est de determiner à partir de l’angle courant la vitesse linéaire grâce à un changement de base et la vitesse angulaire avec un asservissement proportionnel. Pour determiner la vitesse angulaire, TurnOnTheSpot prend la différence entre l’angle courant et l’angle souhaité puis la multiplie par un coefficient proportionnel. Le calcul de la vitesse linéaire est un peu plus compliqué. Pour determiner cette vitesse, on calcule la distance entre le centre du robot et la projection du point de rotation sur l’axe des x du robot.

Voici un shéma représentant les deux bases :

![TurnOnTheSpot](../images/abstractMoveStrategy/shema_turonthespot.png)

On peux voir la base normale (x,y) et la base du robot (u,v).

La base du robot est déterminé à partir d’une rotation alpha (angle du robot) de la base (x,y).
On peut donc facilement écrire :

`u = cos(alpha).x + sin(alpha).y`

`v = sin(alpha).x - cos(alpha).y`

La distance entre le centre du robot et la projection du centre de rotation sur l’axe u est égale à la composante u du vecteur robot -> point. Donc cette distance est égal à `cos(alpha).dx + sin(alpha).dy` avec `dx = x_point - x_robot` et `dy = y_point - y_robot`. On multiplie cette distance avec un coefficient proportionnel pour obtenir notre vitesse linéaire.

Voici un exemple de fonctionnement:

![TurnOnTheSpot](../images/abstractMoveStrategy/gif_turnonthespot.gif)

## PurePursuit

Purpursuit a pour but de suivre une trajectoire sous la forme de ligne brisée.

La trajectoire a la particularité de suivre cette ligne avec des trajectoires courbes en essayant de respecter au mieux la ligne consigne. Pour faire le calcul des vitesses cet algorithme est divisé en 4 sous algorithme.

Le principe de Purpursuit est de générer un point à faire suivre au robot. Ce point est situé devant la projection du robot sur la droite. La distance de ce point cible est apellé lookahead. Une fois le point calculé, on en déduit les vitesses pour le rejoindre. L’astuce réside dans le fait que ce point va avancer en même temps que le robot jusqu’à la fin de la ligne brisée.

![PurePusuit](../images/abstractMoveStrategy/gif_purpursuit.gif)

Nous avons donc besoin d’un algorithme pour le calcul de ce point intermédaire puis d’un autre pour calculer les vitesses associés. Comme je l’ai dit précedement, Purpursuit possède 4 algorithmes. En effet, il y a bien un algorithme pour le calcul des vitesses et pour la determination du point projeter sur la droite, mais nous avons aussi besoin d’un algorithme pour determiner un point sur la droite si le robot est trop loin de cette dernière, car aucun point ne respectera la distance limite entre lui et le robot. Purpursuit va aussi avoir besoin de la distance qui sépare le point intermédaire du point final pour le calcul des vitesses et pour renseigner l’utilisateur de l’avancée du suivi de ligne.

Pour commencer, nous avons la méthode basique `computeVelSetpoints(float timestep)`, c’est cette méthode qui va calculer les vitesses pour le robot. Dans le début de son appel, `computeVelSetpoints` va calculer à l’aide des méthodes secondaire, un point intermédiaire pour faire ses calculs.

Elle va en premier appeler `checkLookAheadGoal` qui va rendre vrai si un point intermédaire a été trouver et faux si non. Si `checkLookAheadGoal` n’a pas pu determiner de point, computeVelSetpoints va apeller `checkProjectionGoal`. Cette méthode va obligatoirement trouver le point le plus proche du robot sur la ligne brisée. Une fois le point trouvé, on va determiner les vitesses angulaires et linéaires.

Pour determiner la vitesse linéaire, on fait la saturation de la multiplication de la distance par un coefficient proportionnel par la vitesse linéaire max.
Pour la vitesse angulaire, on sature la multiplication entre l’angle formé par l’axe du robot et le vecteur robot point, par la vitesse angulaire max.

Un problème va rapidement arriver si on ne modifie par les vitesses maximales. Car si notre robot désire rejoindre la ligne car il est désaxé mais que ça vitesse linéaire est trop importante par rapport à sa vitesse angulaire, il ne pourra alors jamais rejoindre la ligne rapidement car l’angle de courbure sera trop grand. Pour résoudre ce problème, avant de calculer les vitesses, on calcule des vitesses maximales pour permettre au robot d’atteindre un rayon de courbure satisfaisant.
Ce rayon est le rayon du cercle formé par le point intermédaire et la tangente qu’elle doit faire avec le robot.

![PurePusuit](../images/abstractMoveStrategy/schema_purpursuit.png)

Le rayon est donc égal à `2*sin(alpha)/chord` .

L’idée est de determiner la vitesse angulaire max pour avoir cette courbure. On utilise le fait que le rayon de courbure est égal à la vitesse linéaire sur la vitesse angulaire. On obtient donc la vitesse angulaire max égal à : `LinVelMax * abs(2.sin(delta)) / chord`.

Il reste à régler le problème que si la nouvelle vitesse angulaire max calculée dépasse la vitesse angulaire max du robot, le robot ne pourra pas atteindre le rayon de courbure désiré. Si tel est le cas, on procède à l’envers. C’est à dire que l’on va determiner la vitesse linéaire max à partir de la vitesse angulaire max avec la formule : `AngVelMax * chord / abs(2.sin(delta))`.

Le calcul du point intermédiaire se passe en deux étapes. On commence par faire la projection du robot sur le segment courant. On utilise le produit scalaire pour connaitre la distance entre la projection et le debut du segment. Une fois que ce point est calculé, on calcule la distance entre ce point et le robot avec un produit vectoriel. Cette distance nous permet ensuite de determiner la distance entre le point intermédiaire final et la projection du robot à l’aide d’un théorème de Pythagore et de la valeur du LookaHead.

<aside class="notice">
Il faut savoir que dans l’algorithme ces points sont stockés sous la forme d’un rapport entre la distance de ce point avec le début de segment et la taille du segment.
</aside>

## API

| Members                                                              | Descriptions                                   |
| -------------------------------------------------------------------- | ---------------------------------------------- |
| `class`[`AbstractMoveStrategy`](#class_abstract_move_strategy)       | Interface de Stratégie de mouvement.           |
| `class`[`PurePursuit`](#class_pure_pursuit)                          | Trajectoire courbe le long d'une ligne brisée. |
| `class`[`TurnOnTheSpot`](#class_turn_on_the_spot)                    | Rotation du robot sans translations.           |
| `struct`[`PurePursuit::Waypoint`](#struct_pure_pursuit_1_1_waypoint) | Structure d'un point de passage de Purpursuit. |

# class `AbstractMoveStrategy`

Interface de Stratégie de mouvement.

Interface à implémenter pour réaliser une classe de strategie de mouvement.

## Summary

| Members                                                                                                                                                   | Descriptions                                         |
| --------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| `protected`[`PositionController`](#class_position_controller)`*`[`m_context`](#class_abstract_move_strategy_1a0a0cc07cae1127a7d7dc454d240d48ba)           | Pointeur du PositionControlleur associé.             |
| `protected void`[`computeVelSetpoints`](#class_abstract_move_strategy_1a8bed0fd2cf5af27a7804d1be57823a24)`(float timestep)`                               | Calcul les nouvelles vitesses désirer.               |
| `protected bool`[`getPositionReached`](#class_abstract_move_strategy_1a36fbdf31d04778a013fbbd160ff2ed07)`()`                                              | Indique si la position désirée est atteinte.         |
| `protected inline const`[`Position`](#struct_position)`&`[`getPosInput`](#class_abstract_move_strategy_1a24e6faf227a28c534322f0918b4cb732)`() const`      | Retourne la position du robot.                       |
| `protected inline const`[`Position`](#struct_position)`&`[`getPosSetpoint`](#class_abstract_move_strategy_1ae3d80bae8f22c55ff91ca3dc508d7434)`() const`   | Retourne la position à atteindre.                    |
| `protected inline void`[`setVelSetpoints`](#class_abstract_move_strategy_1a9be3e48b174c6f084e84af1e903c78a4)`(float linVelSetpoint,float angVelSetpoint)` | Charge une nouvelle vitesse pour le robot.           |
| `protected inline float`[`getLinVelKp`](#class_abstract_move_strategy_1ad2cdd59254831d777e276c053d93e507)`() const`                                       | Retourne le coef proportionnel de vitesse linéaire.  |
| `protected inline float`[`getAngVelKp`](#class_abstract_move_strategy_1a34e828c9189a02c92889a476254d95da)`() const`                                       | Retourne le coef proportionnel de vitesse angulaire. |
| `protected inline float`[`getLinVelMax`](#class_abstract_move_strategy_1a37f4a119ef96ba81c3621d6f640039b0)`() const`                                      | Retourne vitesse linéaire max.                       |
| `protected inline float`[`getAngVelMax`](#class_abstract_move_strategy_1a8e71820d1f3dfea99ec35cf3aaf1319a)`() const`                                      | Retourne vitesse angulaire max.                      |
| `protected inline float`[`getLinPosThreshold`](#class_abstract_move_strategy_1a4ac627813184bbde3b3031c3575f260f)`() const`                                | Retourne la précision cartésienne à atteindre.       |
| `protected inline float`[`getAngPosThreshold`](#class_abstract_move_strategy_1aeb5f02ec6bc18993066712b728d901c5)`() const`                                | Retourne la précision angulaire à atteindre.         |

## Members

#### `protected`[`PositionController`](#class_position_controller)`*`[`m_context`](#class_abstract_move_strategy_1a0a0cc07cae1127a7d7dc454d240d48ba)

Pointeur du PositionControlleur associé.

#### `protected void`[`computeVelSetpoints`](#class_abstract_move_strategy_1a8bed0fd2cf5af27a7804d1be57823a24)`(float timestep)`

Calcul les nouvelles vitesses désirer.

Méthode à implémenter pour réaliser une [AbstractMoveStrategy](#class_abstract_move_strategy). Cette méthode calcul à partir de la position du robot des vitesses à suivre pour le robot.

#### Parameters

- `timestep` Temps depuis le dernier appel en secondes.

#### `protected bool`[`getPositionReached`](#class_abstract_move_strategy_1a36fbdf31d04778a013fbbd160ff2ed07)`()`

Indique si la position désirée est atteinte.

Calcul la distance entre la position du robot et la position désirée selon le mode de calcul de l'[AbstractMoveStrategy](#class_abstract_move_strategy).

#### Returns

true Si la position est atteinte.

#### Returns

false Si la position n'est pas atteinte.

#### `protected inline const`[`Position`](#struct_position)`&`[`getPosInput`](#class_abstract_move_strategy_1a24e6faf227a28c534322f0918b4cb732)`() const`

Retourne la position du robot.

Retourne la position du robot stocker dans le [PositionController](#class_position_controller).

#### Returns

La position du robot sous la struct [Position](#struct_position).

#### `protected inline const`[`Position`](#struct_position)`&`[`getPosSetpoint`](#class_abstract_move_strategy_1ae3d80bae8f22c55ff91ca3dc508d7434)`() const`

Retourne la position à atteindre.

#### Returns

[Position](#struct_position) à atteindre.

#### `protected inline void`[`setVelSetpoints`](#class_abstract_move_strategy_1a9be3e48b174c6f084e84af1e903c78a4)`(float linVelSetpoint,float angVelSetpoint)`

Charge une nouvelle vitesse pour le robot.

#### Parameters

- `linVelSetpoint` Vitesse linéaire en mm/s.

- `angVelSetpoint` Vitesse angulaire en rad/s.

#### `protected inline float`[`getLinVelKp`](#class_abstract_move_strategy_1ad2cdd59254831d777e276c053d93e507)`() const`

Retourne le coef proportionnel de vitesse linéaire.

#### Returns

Coefficient proportionnel (sans unité).

#### `protected inline float`[`getAngVelKp`](#class_abstract_move_strategy_1a34e828c9189a02c92889a476254d95da)`() const`

Retourne le coef proportionnel de vitesse angulaire.

#### Returns

Coefficient proportionnel (sans unité).

#### `protected inline float`[`getLinVelMax`](#class_abstract_move_strategy_1a37f4a119ef96ba81c3621d6f640039b0)`() const`

Retourne vitesse linéaire max.

#### Returns

Vitesse en mm/s.

#### `protected inline float`[`getAngVelMax`](#class_abstract_move_strategy_1a8e71820d1f3dfea99ec35cf3aaf1319a)`() const`

Retourne vitesse angulaire max.

#### Returns

Vitesse en rad/s.

#### `protected inline float`[`getLinPosThreshold`](#class_abstract_move_strategy_1a4ac627813184bbde3b3031c3575f260f)`() const`

Retourne la précision cartésienne à atteindre.

#### Returns

Précision en mm.

#### `protected inline float`[`getAngPosThreshold`](#class_abstract_move_strategy_1aeb5f02ec6bc18993066712b728d901c5)`() const`

Retourne la précision angulaire à atteindre.

#### Returns

Précision en rad.
