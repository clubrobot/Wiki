# Structure du projet

Notre projet est hébergé sur github à l'adresse suivante : https://github.com/clubrobot/

Chaque année, un nouveau repo est crée, celui de l'année 2020 est dispnible [ici](https://github.com/clubrobot/team-2020).

Le projet se strcture de la facon suivante :

```
team-2020
+- arduino : le code des cartes avec arduino
| |
| +- common : contient toutes les libs du club.
| | +- xxx.h
| | +- xxx.cpp
| |
| +- wheeledbase : exemple de carte actionneur : la base roulante.
| | +- main.ino
| | +- instructions.h
| | +- instructions.cpp
| | +- makefile
| |
| +- actionneur_name : carte actionneur lambda
| | +- main.ino
| | +- instructions.h
| | +- instructions.cpp
| | +- makefile
| |
| +- Module.mk
|
+- esp32 : le code des cartes avec esp32
| |
| +- common : contient toutes les libs du club.
| | +- xxx.h
| | +- xxx.cpp
| |
| +- wheeledbase : exemple de carte actionneur : la base roulante.
| | +- main.ino
| | +- instructions.h
| | +- instructions.cpp
| | +- makefile
| |
| +- actionneur_name : carte actionneur lambda
| | +- main.ino
| | +- instructions.h
| | +- instructions.cpp
| | +- makefile
| |
| +- Module.mk
|
+- raspberrypi : le code sur raspberrypi
| |
| +- common
| +- daughter_cards
| +- robots
```
