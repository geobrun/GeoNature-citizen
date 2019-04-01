# GeoNature-citizen

English:

GeoNature citizen is a free and Open Source platform for citizen science projects for biodiversity data collection. It is fully customizable. Your program may be a single or a multiple program and be based on existing or adoc list of species.
The data collection is gamified to improve the user management using badges and scores. I can also be customized to accept new user to be created or not.
It is based on a fully open Source stack from PostgreSQL to Angular.


:bangbang: **git Attention, projet en cours de développement, non fonctionnel**

Portail d'inventaire participatif de la biodiversité à destination du grand public

Les développements de cette première version sur dans la branche [dev](https://github.com/PnX-SI/GeoNature-citizen/tree/dev).

La discussion de préfiguration du projet est [ici](https://github.com/PnX-SI/GeoNature-citizen/issues/2)

## Solutions logicielles

### Backend (API)

* Python 3.5
  * Flask (moteur de l'API)
  * Marshmallow
  * flask-jwt-extended (pour l'authentification)
  * requests (pour l'utilisation d'API externes comme les portails faune-xxx.org)
* PostgreSQL 10 / Postgis 2.4

### Frontend

* NodeJS 8
* Angular 6
* Leaflet
* Bootstrap 41

## L'origine du projet

Ce projet est initialement développé pour répondre aux besoins de collectes participatives dans le cadre des démarches d'atlas de biodiversité communal/territorial (ABC/ABT). 
La première version de ce projet est une démarche soutenue par l'agglomération de Valence Romans et développée par la LPO Ardèche.
    
![Logo Agglo Valence Romans](https://upload.wikimedia.org/wikipedia/fr/thumb/b/b6/Logo_Valence_Romans.jpg/251px-Logo_Valence_Romans.jpg)

Au delà des fonctionnalités principales de collecte de données naturalistes orienté grand public, des modules supplémentaires sont prévus :
* [gnc_events](https://github.com/lpofredc/gnc_events) Pour afficher une liste d'évènements en lien avec la biodiversté;
* [gnc_small_heritage](https://github.com/lpofredc/gnc_small_heritage) Pour faire l'inventaire du petit patrimoine (murs en pierre sèche, mares, arbres remarquables, etc.);
* Des fiches de présentations des espèces (basé sur [TaxHub](https://github.com/PnX-SI/TaxHub)).

# Suivi du projet du projet
L'état d'avancement du projet est [ici](https://github.com/PnX-SI/GeoNature-citizen/projects) (à venir).

