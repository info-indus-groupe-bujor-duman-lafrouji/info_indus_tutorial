.. _contribuer:

####################################
 Comment nous travaillons ensemble
####################################

Cette page décrit l'organisation du groupe. Elle suit la méthode du cours
(`Coopérer grâce aux pull requests <https://yguel.github.io/informatique_industrielle_avec_ROS2/c01_create_and_publish_doc/p01s06_cooperate_by_pull_request.html>`_).

Dépôts
======

.. list-table::
   :widths: 30 70

   * - Dépôt principal (``upstream``)
     - https://github.com/info-indus-groupe-bujor-duman-lafrouji/info_indus_tutorial
   * - Documentation publiée
     - https://info-indus-groupe-bujor-duman-lafrouji.github.io/info_indus_tutorial/
   * - Fork de chaque membre (``origin``)
     - ``https://github.com/<login>/info_indus_tutorial``

Membres
=======

.. list-table::
   :header-rows: 1

   * - Nom
     - Login GitHub
     - Rôle
   * - Zilan DUMAN
     - ``zduman01``
     - responsable (administre l'organisation)
   * - Andreea BUJOR
     - ``andreeabb07``
     - membre
   * - *Prénom* LAFROUJI
     - *à compléter*
     - membre

Mettre en place son poste (une seule fois)
==========================================

.. code-block:: bash

   cd ~/info_indus
   git clone git@github.com:MON_LOGIN/info_indus_tutorial.git
   cd info_indus_tutorial
   git remote add upstream https://github.com/info-indus-groupe-bujor-duman-lafrouji/info_indus_tutorial.git
   git remote -v

Construire la documentation en local :

.. code-block:: bash

   cd doc/sphinx
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   make html
   xdg-open build/html/index.html

Cycle de travail pour chaque tâche
==================================

#. Une tâche = une *issue* sur le dépôt principal, assignée à quelqu'un. On note le numéro de l'issue dans le tableau d'avancement (:ref:`pentographe`).
#. Se mettre à jour depuis le dépôt principal :

   .. code-block:: bash

      git switch rolling
      git pull upstream rolling
      git push origin rolling

#. Créer une branche dédiée, nommée ``login/sujet`` :

   .. code-block:: bash

      git switch -c andreeabb07/doc_materiel

#. Travailler, vérifier que ``make html`` passe sans erreur, puis commiter par petites étapes avec des messages clairs (``[docs] ...``, ``[urdf] ...``, ``[fix] ...``).
#. Pousser sur son fork et ouvrir une pull request vers ``info-indus-groupe-bujor-duman-lafrouji/info_indus_tutorial`` branche ``rolling``, en remplissant le modèle de description et en écrivant ``Closes #N`` pour lier l'issue.
#. **Au moins une autre personne relit** la pull request (onglet *Files changed*), commente et approuve. Les corrections sont poussées sur la même branche.
#. Le responsable intègre la pull request. Chacun se remet à jour (étape 2).

Règles communes
===============

* On ne pousse jamais directement sur ``rolling``.
* Une pull request = un sujet. Petites pull requests, relues rapidement.
* Les fichiers lourds (modèles CAO, vidéos) ne vont pas dans la documentation : on les met dans le package ROS2 concerné ou on les référence par un lien.
* Toute page de la documentation indique ses responsables et son statut en tête de page.
