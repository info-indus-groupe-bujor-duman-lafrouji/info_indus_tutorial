##################################################
 Package ROS 2 : nœuds cinématiques du pantographe
##################################################

Le package ``pentographe_kinematics`` calcule les angles du pantographe à partir
de la position du crayon, en appliquant les :ref:`modèles géométriques <kinematics>`.
Les équations sont écrites une seule fois dans ``kinematics.py`` (Python pur,
sans ROS) ; les deux nœuds du package ne font que les appeler.

Les deux nœuds
==============

.. list-table::
   :header-rows: 1
   :widths: 25 37 38

   * -
     - ``kinematics_node``
     - ``rviz_joint_state_node``
   * - Pour
     - le pilotage des moteurs
     - la simulation RViz
   * - Entrée
     - ``pen_target`` (x, y)
     - ``pen_target`` (x, y)
   * - Publie
     - 2 angles moteurs (q1, q4)
     - 4 orientations absolues (theta1..theta4)
   * - Topic de sortie
     - ``joint_commands``
     - ``pantograph_angles``
   * - Dimensions
     - valeurs mesurées
     - valeurs du modèle Creo

Dans les deux cas, la position du crayon est donnée en **mètres** sur le topic
``pen_target`` (type ``geometry_msgs/Point``, z ignoré).

Nœud de pilotage (``kinematics_node``)
======================================

Il applique le modèle géométrique inverse et publie les deux angles moteurs, en
radians, sur ``joint_commands`` (type ``sensor_msgs/JointState``, noms ``q1`` et
``q4``). Ces angles, absolus (mesurés depuis x0), sont ensuite convertis en
consignes moteur par le nœud pilote (voir la calibration plus bas).

Nœud de simulation (``rviz_joint_state_node``)
==============================================

Pour afficher le mécanisme complet dans RViz, il faut aussi les angles des deux
avant-bras. Ce nœud publie donc les **quatre orientations absolues** des barres,
en radians, sur ``pantograph_angles`` (type ``sensor_msgs/JointState``) :

.. list-table::
   :header-rows: 1
   :widths: 20 25 55

   * - Angle
     - Barre
     - Orientation de
   * - ``theta1``
     - link1 (:math:`A_1 \to A_2`)
     - bras gauche (moteur)
   * - ``theta2``
     - link2 (:math:`A_2 \to A_3`)
     - avant-bras gauche
   * - ``theta3``
     - link3 (:math:`A_4 \to A_3`)
     - avant-bras droit
   * - ``theta4``
     - link4 (:math:`A_5 \to A_4`)
     - bras droit (moteur)

Fournir les quatre angles évite au nœud de visualisation (côté URDF) de
recalculer le point de fermeture :math:`A_3` : les équations ne sont écrites
qu'une seule fois. La conversion de ces angles vers le repère de l'URDF, de la
forme :math:`\alpha_i - \theta_i`, dépend de l'URDF et reste réalisée du côté de
sa description.

Comme les meshes de l'URDF sont exportés du modèle Creo, ce nœud utilise par
défaut les dimensions du modèle Creo (fichier ``config/pentographe_creo.yaml``),
et non les valeurs mesurées sur le robot réel.

Prise en main (pas à pas)
=========================

#. **Compiler**, depuis la racine de l'espace de travail ROS 2 :

   .. code-block:: bash

      cd ros2_ws
      colcon build --packages-select pentographe_kinematics

#. **Activer** l'espace de travail (à refaire dans chaque nouveau terminal) :

   .. code-block:: bash

      source install/setup.bash

#. **Lancer** le nœud voulu :

   .. code-block:: bash

      ros2 launch pentographe_kinematics kinematics.launch.py        # pilotage
      ros2 launch pentographe_kinematics rviz_joint_state.launch.py  # simulation RViz

#. Dans un autre terminal, **envoyer une position** du crayon :

   .. code-block:: bash

      ros2 topic pub --once /pen_target geometry_msgs/Point "{x: 0.02, y: 0.17, z: 0.0}"

#. **Lire la sortie** :

   .. code-block:: bash

      ros2 topic echo /joint_commands      # angles moteurs (pilotage)
      ros2 topic echo /pantograph_angles   # 4 orientations (RViz)

Réglage des dimensions
======================

Les dimensions sont dans des fichiers YAML, modifiables **sans toucher au code** :

* ``config/pentographe_kinematics.yaml`` : valeurs mesurées (pilotage) ;
* ``config/pentographe_creo.yaml`` : valeurs du modèle Creo (simulation RViz).

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

Le nœud de pilotage produit des angles :math:`q_1` et :math:`q_4` mesurés depuis
l'axe x0. Un moteur AX-12A, lui, attend un entier de 0 à 1023 compté depuis son
propre zéro. Le pilote des moteurs applique donc, pour chaque moteur :

.. math::

   \theta_\text{moteur}[°] = s \,(q - q_0)
   \qquad
   \text{position} = \operatorname{round}\!\left(\frac{\theta_\text{moteur}}{300} \times 1023\right)

où :math:`q_0` est l'angle du bras quand le moteur indique zéro et :math:`s = \pm 1`
le sens de rotation. Ces valeurs se règlent dans ``config/calibration.yaml`` :

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
dessin sort décalé, tourné ou en miroir. La calibration fixe deux choses :

* :math:`q_0` **recale l'origine** : « crayon au point (x, y) » dans le modèle
  correspond bien au même point sur la feuille ;
* :math:`s` **fixe le sens** : le dessin ne part pas à l'envers ou en miroir.

Précision du dessin
===================

La résolution de l'AX-12A est de :math:`300° / 1024 \approx 0{,}29°` par pas. Comme
le crayon est au bout d'un bras d'environ 200 mm, un pas déplace la pointe de :

.. math::

   200 \text{ mm} \times 0{,}29° \times \frac{\pi}{180} \approx 1 \text{ mm}

Le crayon ne peut donc pas être positionné plus finement que de l'ordre du
millimètre. Cette limite, avec la qualité de la calibration, détermine la
précision des dessins réalisables.

.. note::

   À compléter au prochain accès au robot : les valeurs de calibration
   (:math:`q_0`, :math:`s`) dans ``config/calibration.yaml`` et les identifiants
   des moteurs.
