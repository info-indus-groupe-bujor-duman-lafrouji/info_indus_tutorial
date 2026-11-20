.. _urdf:

#######################################################
 Génération de la description URDF du système 5 barres
#######################################################

.. admonition:: Suivi de l'étape 3

   :Responsable(s): *à compléter*
   :Statut: à faire
   :Issue / PR: *à compléter*

Nous allons utiliser le modèle 3D au format step conçu et réalisé par M. Olivier PICCIN pour générer la description URDF du système 5 barres.

.. figure:: resources/img/pantograph_step_model.png
   :align: center

Il s'agit d'un assemblage comportant 5 pièces principales :

  #. BATI_ASM: le bâti de la strucuture du système 5 barres avec les 2 moteurs pas à pas. Dans l'URDF nous allons le nommer ``base``.
  #. LINK1_ASM: le bras gauche du système 5 barres. Dans l'URDF nous allons le nommer ``link1``.
  #. LINK2_ASM: l'avant bras gauche du système 5 barres. Dans l'URDF nous allons le nommer ``link2``.
  #. LINK3_ASM: l'avant bras droit du système 5 barres. Dans l'URDF nous allons le nommer ``link3``.
  #. LINK4_ASM: le bras droit du système 5 barres. Dans l'URDF nous allons le nommer ``link4``.

`maquette-5-barres_asm.stp <https://github.com/yguel/informatique_industrielle_avec_ROS2/raw/rolling/doc/sphinx/source/p00_5_bars_system_pencil_holder_with_rpi_and_dynamixel/resources/cad/maquette-5-barres_asm.stp>`__

Du point de vue URDF, nous aurons aussi besoin de définir l'outil de travail: le crayon et donc de le référencer par rapport à son support qui se trouve être sur l'avant bras gauche du pentographe (``link2``).

Dans un fichier URDF, les modèles 3D sont utilisés à partir de fichiers collada (extension .dae).
Il va donc falloir convertir notre modèle 3D au format step en un ensemble de 5 modèles 3D au format collada.

==================================
Extraction des modèles 3D en step
==================================

Pour extraire les modèles 3D au format step, vous pouvez utiliser le logiciel freecad.
   
   #. `base.stp <https://github.com/yguel/informatique_industrielle_avec_ROS2/raw/rolling/doc/sphinx/source/p00_5_bars_system_pencil_holder_with_rpi_and_dynamixel/resources/cad/base.stp>`__
   #. `link1.stp <https://github.com/yguel/informatique_industrielle_avec_ROS2/raw/rolling/doc/sphinx/source/p00_5_bars_system_pencil_holder_with_rpi_and_dynamixel/resources/cad/link1.stp>`__
   #. `link2.stp <https://github.com/yguel/informatique_industrielle_avec_ROS2/raw/rolling/doc/sphinx/source/p00_5_bars_system_pencil_holder_with_rpi_and_dynamixel/resources/cad/link2.stp>`__
   #. `link3.stp <https://github.com/yguel/informatique_industrielle_avec_ROS2/raw/rolling/doc/sphinx/source/p00_5_bars_system_pencil_holder_with_rpi_and_dynamixel/resources/cad/link3.stp>`__
   #. `link4.stp <https://github.com/yguel/informatique_industrielle_avec_ROS2/raw/rolling/doc/sphinx/source/p00_5_bars_system_pencil_holder_with_rpi_and_dynamixel/resources/cad/link4.stp>`__

=====================================
Conversion des modèles 3D en collada
=====================================

Pour convertir les modèles 3D en collada (dae), vous pouvez utiliser le logiciel freecad.
   
   #. `base.dae <https://github.com/yguel/informatique_industrielle_avec_ROS2/raw/rolling/doc/sphinx/source/p00_5_bars_system_pencil_holder_with_rpi_and_dynamixel/resources/cad/base.dae>`__
   #. `link1.dae <https://github.com/yguel/informatique_industrielle_avec_ROS2/raw/rolling/doc/sphinx/source/p00_5_bars_system_pencil_holder_with_rpi_and_dynamixel/resources/cad/link1.dae>`__
   #. `link2.dae <https://github.com/yguel/informatique_industrielle_avec_ROS2/raw/rolling/doc/sphinx/source/p00_5_bars_system_pencil_holder_with_rpi_and_dynamixel/resources/cad/link2.dae>`__
   #. `link3.dae <https://github.com/yguel/informatique_industrielle_avec_ROS2/raw/rolling/doc/sphinx/source/p00_5_bars_system_pencil_holder_with_rpi_and_dynamixel/resources/cad/link3.dae>`__
   #. `link4.dae <https://github.com/yguel/informatique_industrielle_avec_ROS2/raw/rolling/doc/sphinx/source/p00_5_bars_system_pencil_holder_with_rpi_and_dynamixel/resources/cad/link4.dae>`__

=========================
Création du fichier URDF
=========================

Dans un fichier URDF les modèles 3D sont référencés par des balises ``<mesh>``.