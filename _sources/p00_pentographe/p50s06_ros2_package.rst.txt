##################################################
 Package ROS 2 : nœud cinématique du pantographe
##################################################

Le package ``pentographe_kinematics`` traduit une position voulue du crayon en
angles pour les deux moteurs, en appliquant le :ref:`modèle géométrique inverse <kinematics>`.

Rôle et interface
=================

Le nœud ne commande pas directement les moteurs : il calcule les angles, qu'un
second nœud (le pilote des moteurs) se charge d'envoyer au matériel.

.. code-block:: text

   position (x, y)                angles (q1, q4)              moteurs
   ----------------> [ nœud cinématique ] ----------------> [ pilote ] --> AX-12A

.. list-table::
   :header-rows: 1
   :widths: 20 25 25 30

   * - Sens
     - Topic
     - Type de message
     - Contenu
   * - entrée
     - ``pen_target``
     - ``geometry_msgs/Point``
     - position du crayon en **mètres** (z ignoré)
   * - sortie
     - ``joint_commands``
     - ``sensor_msgs/JointState``
     - angles ``q1`` et ``q4`` en **radians**, mesurés depuis l'axe x0

Les angles produits sont absolus (depuis x0) : leur conversion en consignes
moteur (calibration) est décrite plus bas.

Prise en main (pas à pas)
=========================

Pour utiliser le package à partir de zéro :

#. **Compiler** le package, depuis la racine de l'espace de travail ROS 2 :

   .. code-block:: bash

      cd ros2_ws
      colcon build --packages-select pentographe_kinematics

#. **Activer** l'espace de travail (à refaire dans chaque nouveau terminal) :

   .. code-block:: bash

      source install/setup.bash

#. *(optionnel)* **Ajuster les dimensions** dans
   ``src/pentographe_kinematics/config/pentographe_kinematics.yaml`` si elles ont
   changé (remesure, autre robot).

#. **Lancer** le nœud :

   .. code-block:: bash

      ros2 launch pentographe_kinematics kinematics.launch.py

#. Dans **un autre terminal** (où l'on a aussi fait ``source install/setup.bash``),
   **envoyer une position** du crayon (ici x = 0,05 m, y = 0,18 m) :

   .. code-block:: bash

      ros2 topic pub --once /pen_target geometry_msgs/Point "{x: 0.05, y: 0.18, z: 0.0}"

#. **Lire les angles** calculés :

   .. code-block:: bash

      ros2 topic echo /joint_commands

#. **Pour piloter le vrai robot** : remplir ``config/calibration.yaml`` (voir plus
   bas) et lancer le nœud pilote, qui lit ``joint_commands`` et commande les moteurs.

Réglage des dimensions
======================

Toutes les dimensions sont dans le fichier ``config/pentographe_kinematics.yaml``.
Pour les changer, il suffit d'éditer ce fichier : **aucune modification du code
n'est nécessaire**.

.. code-block:: yaml

   pentographe_kinematics:
     ros__parameters:
       l0: 137.0   # entraxe des moteurs
       l1: 86.0    # bras gauche
       l2: 150.0   # avant-bras gauche
       l3: 150.0   # avant-bras droit
       l4: 85.0    # bras droit
       e_par: -16.0   # décalage du crayon le long de link2
       e_perp: 26.0   # décalage perpendiculaire

Calibration (du modèle aux moteurs)
===================================

Le nœud produit des angles :math:`q_1` et :math:`q_4` mesurés depuis l'axe x0.
Un moteur AX-12A, lui, attend un entier de 0 à 1023 compté depuis son propre
zéro. Le pilote des moteurs doit donc appliquer, pour chaque moteur :

.. math::

   \theta_\text{moteur}[°] = s \,(q - q_0)
   \qquad
   \text{position} = \operatorname{round}\!\left(\frac{\theta_\text{moteur}}{300} \times 1023\right)

où :math:`q_0` est l'angle du bras quand le moteur indique zéro et :math:`s = \pm 1`
le sens de rotation. Ces deux valeurs s'obtiennent par une calibration :

#. placer les bras dans une **position de référence connue** (par exemple bras gauche vertical) ;
#. lire la valeur renvoyée par chaque moteur dans cette position ;
#. en déduire :math:`q_0` et :math:`s`.

Comme pour les dimensions, ces valeurs se règlent dans un fichier YAML dédié,
``config/calibration.yaml``, sans toucher au code :

.. code-block:: yaml

   pilote_moteurs:
     ros__parameters:
       moteur_gauche:
         id: 0          # identifiant Dynamixel, à relever
         q0_deg: 0.0    # angle du bras à la position moteur 0, à mesurer
         s: 1           # sens de rotation : +1 ou -1
       moteur_droit:
         id: 0
         q0_deg: 0.0
         s: 1

À quoi sert concrètement cette calibration
------------------------------------------

Sans calibration, on connaît la *forme* du mouvement, mais pas sa *position* : le
dessin sort décalé, tourné ou en miroir par rapport à ce qui était prévu. La
calibration fixe deux choses :

* :math:`q_0` **recale l'origine** : il garantit que « crayon au point (x, y) »
  dans le modèle correspond bien au même point sur la feuille ;
* :math:`s` **fixe le sens** : il évite que le dessin parte à l'envers ou en miroir.

C'est donc elle qui relie le repère du modèle au repère réel de la feuille.

Précision du dessin
===================

La résolution de l'AX-12A est de :math:`300° / 1024 \approx 0{,}29°` par pas : le
moteur ne peut pas se placer entre deux pas. Comme le crayon est au bout d'un bras
d'environ 200 mm, un pas déplace la pointe d'environ :

.. math::

   200 \text{ mm} \times 0{,}29° \times \frac{\pi}{180} \approx 1 \text{ mm}

Le crayon ne peut donc pas être positionné plus finement que de l'ordre du
millimètre. Cette limite, ajoutée à la qualité de la calibration, détermine la
précision des dessins réalisables.

.. note::

   À compléter au prochain accès au robot : les valeurs de calibration
   (:math:`q_0`, :math:`s`) dans ``config/calibration.yaml`` et les identifiants
   des moteurs, à relever une fois le Raspberry Pi accessible.
