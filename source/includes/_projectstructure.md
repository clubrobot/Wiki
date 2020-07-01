# Structure du projet

Notre projet est hébergé sur github à l'adresse suivante : https://github.com/clubrobot/

Chaque année, un nouveau repo est crée, celui de l'année 2020 est dispnible [ici](https://github.com/clubrobot/team-2020).

Le projet se strcture de la facon suivante :

```
team-2020
+- arduino
| |
| +- common
| | +- xxx.h
| | +- xxx.cpp
| |
| +- wheeledbase
| | +- main.ino
| | +- instructions.h
| | +- instructions.cpp
| | +- makefile
| |
| +- actionneur_name
| | +- main.ino
| | +- instructions.h
| | +- instructions.cpp
| | +- makefile
| |
| +- Module.mk
|
+- esp32
| |
| +- common
| | +- xxx.h
| | +- xxx.cpp
| |
| +- wheeledbase
| | +- main.ino
| | +- instructions.h
| | +- instructions.cpp
| | +- makefile
| |
| +- actionneur_name
| | +- main.ino
| | +- instructions.h
| | +- instructions.cpp
| | +- makefile
| |
| +- Module.mk
|
+- raspberrypi
| |
| +- common
| +- daughter_cards
| +- robots
```

### Arduino

Cette section contient tout le code relatifs au cartes embarquées utillisant une ardino nano.

Le dossier `common` contient toutes les librairies du club (Ex. `Odometry`, `AX12`, `Motor`, ...).

Le dossier `wheelebase` est le code de la carte actionneur wheeledbase. Elle contient un `main.ino`,

TODO

### ESP32

TODO

### Raspberrypi

TODO
