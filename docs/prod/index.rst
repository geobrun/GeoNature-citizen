====================================
Installation de GeoNature-citizen 
====================================


Prérequis
=========

Application développée et installée sur un serveur Debian 9.

Ce serveur doit aussi disposer de : 

- sudo (``apt-get install sudo``)
- un utilisateur (``monuser`` dans cette documentation) appartenant au groupe ``sudo`` (pour pouvoir bénéficier des droits d'administrateur)

**1. Créer un utilisateur Debian :**

::

  adduser nom_utilisateur (geonatadmin) entrez un mot de passe (****)
  usermod -aG sudo nom_utilisateur (geonatadmin)
  su - nom_utilisateur (geonatadmin)

**2. Vérifier que l’utilisateur est correctement créé :**

::

  sudo -l (entrez un mot de passe) : vérifier que ALL
  sudo whoami : ok si on peut faire un sudo

**3. Changer la locale en fr (il faut être root) :**

::

  sudo dpkg-reconfigure locales
    
INSTALLATION Automatique
========================

:notes:

 - Bien vérifier de ne pas être en ``root`` :

  .. code-block:: bash

    su - nom_utilisateur (geonatadmin)

 - S'assurer d'avoir le projet Geonature-citizen dans ce dossier ainsi que d'etre propriétaire du dossier et de ses dépendances

 - Se rendre sur la Home de votre utilisateur

  .. code-block:: bash

    cd

Lancer le script d'installation :

.. code-block:: bash

  ./install/install_app.sh

- Le script créera un fichier de config settings.ini, il faut alors le compléter avec les informations de votre installation.

.. code-block:: bash

  editor ./config/settings.ini


- Complétez également les fichiers de config Apache situés dans config/apache
- Relancer le script :

.. code-block:: bash

  ./install/install_app.sh

Le script crééra la base de données, configurera taxhub si l'installation est demandée, configurera apache et installera toutes les dépendances du projet Geonature-Citizen.



INSTALLATION DE TAXHUB
======================

Pour plus de détails, lien officiel pour l’installation de TaxHub :
https://taxhub.readthedocs.io/fr/latest/

Configurer le serveur :
https://taxhub.readthedocs.io/fr/latest/serveur.html#installation-et-configuration-du-serveur

Configurer PostgreSQL :
https://taxhub.readthedocs.io/fr/latest/serveur.html#installation-et-configuration-de-posgresql

Configuration et installation de l’application :
https://taxhub.readthedocs.io/fr/latest/installation.html

:notes:
 - Bien vérifier de ne pas être en ``root`` :

  ::

    su - nom_utilisateur (geonatadmin)

 - Pour avoir les caractéristiques de votre instance :

  ::

    lsb_release -a
    uname -a


INSTALLATION DE GEONATURE-CITIZEN
=================================

**Etape 1 : Configurer PostgreSQL :**

:notes:

 - Cette étape n'est nécessaire que si TaxHub n’est pas installé
 - voir init_launch_db.rst : https://github.com/PnX-SI/GeoNature-citizen/blob/master/docs/devs/init_launch_db.rst

::

  sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ stretch-pgdg main" >>/etc/apt/sources.list.d/postgresql.list'
  sudo wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add
  sudo apt update
  sudo apt install postgresql-10 postgresql-10-postgis-2.5 postgresql-10-postgis-2.5-scripts git
  sudo -u postgres createuser -e -E -P dbuser (geonatadmin) (Entrez le password) 
  sudo -u postgres createdb -e -E UTF8 -O dbuser (geonatadmin) dbname (geonature2db) 

:notes:

 - ls /etc/init.d/ : pour lister les services
 - sudo service restart postgresql : vérification 

**Etape 2 : Installer python3, pip et virtualenv :**

::

  python3 -m pip install --upgrade --user virtualenv
  sudo apt install python3-pip

installer virtualenv :

::

 export PATH=/home/username/.local/bin:$PATH (username = geonatadmin)
 echo $PATH
 virtualenv -p /usr/bin/python3 venv
 source venv/bin/activate
 
**Etape 3 : Installation du backend et de la base des données :**

:notes:

 - init_launch_backend.rst 
 - creation referentiel géo
 - voir : https://github.com/PnX-SI/GeoNature-citizen/blob/master/docs/devs/init_launch_db.rst

Cloner le dépôt Github de GeoNature-citizen

::

 sudo apt install git
 git clone name (citizen)
 git checkout branch_name
 cd citizen/backend


Création du référentiel des géométries communales :

::

 wget https://github.com/PnX-SI/GeoNature/raw/master/data/core/public.sql -P /tmp
 wget https://github.com/PnX-SI/GeoNature/raw/master/data/core/ref_geo.sql -P /tmp
 wget https://github.com/PnX-SI/GeoNature/raw/master/data/core/ref_geo_municipalities.sql -P /tmp

 psql -d geonature2db -h localhost -p 5432 -U geonatadmin -f /tmp/public.sql
 sed 's/MYLOCALSRID/2154/g' /tmp/ref_geo.sql > /tmp/ref_geo_2154.sql
 psql -d geonature2db -h localhost -p 5432 -U geonatadmin -f /tmp/ref_geo_2154.sql


Pour restaurer en cas de besoin :

::

 psql -d geonature2db -h localhost -U geonatadmin -f ~/citizen_taxhub_l_areas_dump.sql
 if [ ! -f '/tmp/communes_fr_admin_express_2019-01.zip' ]
 then
    wget  --cache=off http://geonature.fr/data/ign/communes_fr_admin_express_2019-01.zip -P /tmp
 else
    echo "/tmp/communes_fr_admin_express_2019-01.zip already exist"
 fi
 unzip /tmp/communes_fr_admin_express_2019-01.zip -d /tmp/
 psql -d geonature2db -h localhost -p 5432 -U geonatadmin -f /tmp/fr_municipalities.sql
 psql -d geonature2db -h localhost -p 5432 -U geonatadmin -c "ALTER TABLE ref_geo.temp_fr_municipalities 
 OWNER TO geonatadmin;"
 psql -d geonature2db -h localhost -p 5432 -U geonatadmin -f /tmp/ref_geo_municipalities.sql
 psql -d geonature2db -h localhost -p 5432 -U geonatadmin -c "DROP TABLE ref_geo.temp_fr_municipalities;"

Lancement du backend pour générer les schémas :

En mode debug :

::

 export FLASK_ENV=development; export FLASK_DEBUG=1; export FLASK_RUN_PORT=5002; export FLASK_APP=wsgi;
 python -m flask run --host=0.0.0.0

Enregistrement du module principal :

::

 insert into gnc_core.t_modules values (1, 'main', 'main', 'main', NULL, false, '2019-05-26 09:38:39.389933', '2019-05-26 09:38:39.389933');

Enregistrement d’un programme exemple

::

 psql -d geonature2db -h localhost -p 5432 -U geonatadmin -c "INSERT INTO gnc_core.t_programs VALUES (1, 'Au 68', 'inventaire  du 68', 'desc', NULL,	NULL,	1,	1,	't', '0106000020E6100000010000000103000000010000000500000001000070947C154042CA401665A5454001000070EE7C15402235D7E667A54540010000D81C7D1540AFBA27365AA5454000000040C47C1540DD9BD74A58A5454001000070947C154042CA401665A54540',	'2019-05-26 09:38:39.389933', '2019-05-26 09:38:39.389933');"

**Etape 4 : éditer le fichier de config :**

:notes:

 - voir : https://github.com/PnX-SI/GeoNature-citizen/blob/dev/docs/devs/config_files.rst

::

 cd ../config
 editer les paramètres dans config.toml

 -SQLALCHEMY_DATABASE_URI :"postgresql+psycopg2://dbuser(geonatadmin):password(***)@127.0.0.1:5432/dbname(geonature2db)"
 -URL_APPLICATION : 'https://ipserveur:4200/'
 -API_ENDPOINT : 'https://ipserveur:5002/api'
 -API_TAXHUB : 'http://ipserveur/taxhub/api/'
 
 - Pour configurer du serveur Smtp renseigner les différents paramètres de votre serveur 
 dans la partie [MAIL] ( MAIL_HOST,MAIL_PORT ..) 
  # La partie [RESET_PASSWD] correspond à la configuration du texte du mail a envoyé pour la restauration
   du mot de passe oublié 
  # La partie [CONFIRM_EMAIL] correspond à la configuration du texte du mail a envoyé pour l’activation
  du compte et la confirmation de l’adresse mail de l’utilisateur   

**Etape 5 : configuration des badges :**

- voir : https://github.com/PnX-SI/GeoNature-citizen/blob/dev/docs/devs/badges.rst

**Etape 6 : configuration du supervisor :**

::

 /etc/supervisor/conf.d/geonature-citizen-service.conf
 [program:citizen]
 command = /home/geonatadmin/citizen/backend/start_gunicorn.sh
 autostart=true
 autorestart=true
 stdout_logfile = /var/log/supervisor/citizen.log
 redirect_stderr = true

**Etape 7 : Installation du frontend :**

:notes:

 - voir : https://github.com/PnX-SI/GeoNature-citizen/tree/dev/docs/devs/init_launch_frontend.rst

::

 cd citizen/frontend/
 nvm use --lts  # Now using node v10.16.0 (npm v6.9.0)
 si pas installé : nvm install --lts (remplacer lts par la dernière version)
 cp -v src/assets/badges_* ../media/


Editer la conf :

::

 cp src/conf*.ts.sample src/conf/  # ajuster la conf
 # copier le template css alternatif
 cp src/custom/custom.css.template src/custom/custom.css
 # Pour configurer le lien externe de la fiche détaillée de l'espèce, éditer l'entrée suivante:
 details_espece_url: "<url_inpn_or_atlas>/cd_nom/" // !! garder bien le cd_nom/ dans l'url


Lancer le front :

.. code-block:: sh

    npm run build:i18n-ssr && npm run serve:ssr
