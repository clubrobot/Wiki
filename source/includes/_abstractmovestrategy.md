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

[Purepursuit](includes/API/html/class_pure_pursuit.html)

# Summary

| Members                                                                            | Descriptions                                                                                                                                                                                                                  |
| ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `class`[`AbstractCodewheel`](#class_abstract_codewheel)                            | Classe abstraite d'une roue codeuse.                                                                                                                                                                                          |
| `class`[`AbstractMotor`](#class_abstract_motor)                                    | Instance de moteur.                                                                                                                                                                                                           |
| `class`[`AbstractMoveStrategy`](#class_abstract_move_strategy)                     | Interface de Stratégie de mouvement.                                                                                                                                                                                          |
| `class`[`Adafruit_TCS34725`](#class_adafruit___t_c_s34725)                         |
| `class`[`BrushlessMotor`](#class_brushless_motor)                                  |
| `class`[`ButtonCard`](#class_button_card)                                          |
| `class`[`Clock`](#class_clock)                                                     | Utilitaire pour gérer le temps dans vos programmes Arduino.                                                                                                                                                                   |
| `class`[`Codewheel`](#class_codewheel)                                             | Fait la passerelle entre les roues codeuses et le compteur.                                                                                                                                                                   |
| `class`[`ColorSensor`](#class_color_sensor)                                        |
| `class`[`CRC16`](#class_c_r_c16)                                                   |
| `class`[`DCMotor`](#class_d_c_motor)                                               | Pilotage de moteur continu.                                                                                                                                                                                                   |
| `class`[`DCMotorsDriver`](#class_d_c_motors_driver)                                | Utilisation des drivers moteurs.                                                                                                                                                                                              |
| `class`[`DifferentialController`](#class_differential_controller)                  | Controle les moteurs.                                                                                                                                                                                                         |
| `class`[`EchoHandler`](#class_echo_handler)                                        | Gère le pin de reception du capteur.                                                                                                                                                                                          |
| `class`[`EndStop`](#class_end_stop)                                                | Capteur fin de course est une classe permettant d'utiliser les capteurs fins de courses (clic de souris/bouton poussoir) Pour utiliser cette classe le bouton doit être d'un côté relié à la masse et de l'autre à l'arduino. |
| `class`[`FullSpeedServo`](#class_full_speed_servo)                                 | Pilotage de Servomoteur particulier.                                                                                                                                                                                          |
| `class`[`Lsm303`](#class_lsm303)                                                   |
| `class`[`MagneticCompas`](#class_magnetic_compas)                                  |
| `class`[`NonCopyable`](#class_non_copyable)                                        | Classe a hériter pour empécher la copie de cette dernière.                                                                                                                                                                    |
| `class`[`Odometry`](#class_odometry)                                               | Calcule la position en temps réel du robot.                                                                                                                                                                                   |
| `class`[`SerialTalks::ostream`](#class_serial_talks_1_1ostream)                    | Stream virtuel pour les erreurs et autre. est un outils pour permettre de mieux transmettre les erreurs rencontrées et les STD::OUT.                                                                                          |
| `class`[`PeriodicProcess`](#class_periodic_process)                                | Classe à implémenter pour gérer les appels dans la loop.                                                                                                                                                                      |
| `class`[`PID`](#class_p_i_d)                                                       | Classe d'asservissement.                                                                                                                                                                                                      |
| `class`[`PositionController`](#class_position_controller)                          | Classe support des objets [AbstractMoveStrategy](#class_abstract_move_strategy).                                                                                                                                              |
| `class`[`PurePursuit`](#class_pure_pursuit)                                        | Trajectoire courbe le long d'une ligne brisée.                                                                                                                                                                                |
| `class`[`QueueArray`](#class_queue_array)                                          |
| `class`[`RobotArm`](#class_robot_arm)                                              |
| `class`[`SensorListener`](#class_sensor_listener)                                  |
| `class`[`SerialTalks`](#class_serial_talks)                                        | Object de communication serial avec un ordinateur.                                                                                                                                                                            |
| `class`[`SerialTopics`](#class_serial_topics)                                      |
| `class`[`ShiftDynamixelClass`](#class_shift_dynamixel_class)                       |
| `class`[`ShiftRegAX12`](#class_shift_reg_a_x12)                                    |
| `class`[`ShiftRegDCMotor`](#class_shift_reg_d_c_motor)                             |
| `class`[`ShiftRegDCMotorsDriver`](#class_shift_reg_d_c_motors_driver)              |
| `class`[`ShiftRegister`](#class_shift_register)                                    |
| `class`[`SoftwareSerial`](#class_software_serial)                                  |
| `class`[`StepByStepMotor`](#class_step_by_step_motor)                              |
| `class`[`TrigHandler`](#class_trig_handler)                                        | Gère le pin d'émission du capteur.                                                                                                                                                                                            |
| `class`[`TurnOnTheSpot`](#class_turn_on_the_spot)                                  | Rotation du robot sans translations.                                                                                                                                                                                          |
| `class`[`TwoWire`](#class_two_wire)                                                |
| `class`[`UltrasonicSensor`](#class_ultrasonic_sensor)                              | Permet d'utiliser des capteurs de distance ulrasons.                                                                                                                                                                          |
| `class`[`Vector`](#class_vector)                                                   |
| `class`[`VelocityController`](#class_velocity_controller)                          | Objet de controle de la vitesse.                                                                                                                                                                                              |
| `class`[`VelocityServo`](#class_velocity_servo)                                    | Objet permettant de controller la vitesse d'un servomoteur.                                                                                                                                                                   |
| `struct`[`_DELAY_TABLE`](#struct___d_e_l_a_y___t_a_b_l_e)                          |
| `struct`[`Lsm303::AccelData`](#struct_lsm303_1_1_accel_data)                       |
| `struct`[`Deserializer`](#struct_deserializer)                                     | Objet destiné à extraire des variables d'un flux en octet.                                                                                                                                                                    |
| `struct`[`Lsm303::MagData`](#struct_lsm303_1_1_mag_data)                           |
| `struct`[`Position`](#struct_position)                                             | Structure de position.                                                                                                                                                                                                        |
| `struct`[`Serializer`](#struct_serializer)                                         | Objet destiné à creer un flux de sortie pour les programme cpp.                                                                                                                                                               |
| `struct`[`SerialTopics::subscription_t`](#struct_serial_topics_1_1subscription__t) | Subscription context structure.                                                                                                                                                                                               |
| `struct`[`PurePursuit::Waypoint`](#struct_pure_pursuit_1_1_waypoint)               | Structure d'un point de passage de Purpursuit.                                                                                                                                                                                |
