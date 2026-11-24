###########################################################
 Description du matériel de la plateforme système 5 barres
###########################################################

Cette page décrit le matériel électronique qui pilote le système 5 barres :
le calculateur, l'interface et l'alimentation des moteurs, et les moteurs eux-mêmes.

Vue d'ensemble
==============

La chaîne de commande est la suivante :

.. code-block:: text

   Raspberry Pi 5  --USB-->  U2D2  -->  U2D2 Power Hub Board  -->  moteurs Dynamixel AX-12A
                                              ^
                                              |
                                        alimentation

Le Raspberry Pi exécute ROS 2 et envoie les ordres. Le U2D2 convertit l'USB en
liaison série Dynamixel. La Power Hub Board distribue cette liaison et
l'alimentation aux moteurs, branchés en chaîne.

Raspberry Pi 5
==============

Calculateur de la plateforme, dans un boîtier imprimé en 3D, avec ventilateur
actif et une extension de stockage **Pimoroni NVMe Base** (disque SSD).

* Documentation : `Raspberry Pi 5 <https://www.raspberrypi.com/products/raspberry-pi-5/>`__
* Extension stockage : `Pimoroni NVMe Base <https://shop.pimoroni.com/products/nvme-base>`__

.. note::

   À compléter au prochain accès au robot : quantité de RAM, système installé
   (Ubuntu 24.04 attendu) et version de ROS 2 (``lsb_release -a`` ; ``printenv ROS_DISTRO``).

Interface et alimentation Dynamixel
===================================

* **U2D2** : interface USB vers liaison série Dynamixel —
  `documentation <https://emanual.robotis.com/docs/en/parts/interface/u2d2/>`__
* **U2D2 Power Hub Board** : distribution de la liaison et de l'alimentation —
  `documentation <https://emanual.robotis.com/docs/en/parts/interface/u2d2_power_hub_board/>`__

.. note::

   À compléter : référence et caractéristiques de l'alimentation (tension, courant),
   et le port USB du Raspberry Pi utilisé.

Moteurs
=======

Les deux articulations actives (:math:`A_1` et :math:`A_5`) sont entraînées par
des servomoteurs **Dynamixel AX-12A**.

* Documentation : `Dynamixel AX-12A <https://emanual.robotis.com/docs/en/dxl/ax/ax-12a/>`__

.. list-table::
   :header-rows: 1
   :widths: 40 60

   * - Caractéristique
     - Valeur
   * - Débattement
     - 300° (résolution 0,29° sur 1024 pas)
   * - Position commandée
     - entier de 0 à 1023 (0 à 300°)
   * - Protocole
     - Protocol 1.0
   * - Débit par défaut
     - 1 000 000 bps
   * - Interface
     - TTL half-duplex (3 fils)

.. important::

   Le débattement limité à 300° est une contrainte pour l'espace de travail :
   toutes les positions du crayon ne sont pas atteignables. Voir
   :ref:`l'espace de travail et les singularités <kinematics>`.

.. note::

   À compléter : les identifiants (ID) des deux moteurs et le débit réellement
   configuré, à relever avec le logiciel une fois le Raspberry Pi accessible.
