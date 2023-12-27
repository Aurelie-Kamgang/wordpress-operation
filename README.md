# Automatisation des actions de gestion d'un site wordpress avec Ansible

Objectif:

Automatiser les actions primaires sur le site EAZYtraining

Prérequis: 

- Installé chez le client git; zip; python3.8; wp-cli
- Installé dans le conteneur wordpress wp-cli

## Version 1

- Installation et activation des plugins et thèmes en utilisant wp-cli et les rôles ansibles
- Désinstallation et désactivation des plugins et thèmes en utilisant wp-cli et les rôles ansibles

### Procédure d'exécution:

 1. Déploiement de l'application EAZYtraining:
	  - Git cloner le repos
      - Se rendre dans le repertoire: `cd /home/user/Role-wordpress`
      - Taper la commande: 
      `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags deploy`

**NB: BECOME password=${user_passord} dans mon cas c'est vagrant et Vault password=vagrant qui est le password définit lors de la création de la vault key**

 2. Installation et activation des plugins et thèmes
      
      - Se rendre dans le fichier :
   
         `vi roles/wordpress/vars/plugins_to_install.yml `
 ou
          `vi roles/wordpress/vars/themes_to_install.yml `

           Pour spécifier la liste des plugins et thèmes à installer
           
      -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande:
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags install_plugins`
ou
          `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags install_themes`

      - Tester l'installation des plugins et thèmes chez le client:

        `wp plugin list  --allow-root --ssh=docker:wordpress`
ou
        `wp theme list  --allow-root --ssh=docker:wordpress`


3. Avoir la liste des plugins installés:

      -   Se rendre dans le repertoire: `cd /home/user/Role-wordpress`
      - Taper la commande: 

		`ansible-playbook -i hosts.yml --ask-vault-pass -vvv   wordpress_operations.yml --tags plugins_list`

      -  Aller chez le client ansible ouvrir le fichier: `plugins_list.json` Pour consulter la liste des plugins installés
      
5.  Désinstallation des plugins et thèmes

      - Se rendre dans le fichier :
   
         `vi roles/wordpress/vars/plugins_to_uninstall.yml `
 ou
          `vi roles/wordpress/vars/themes_to_uninstall.yml `

           Pour spécifier la liste des plugins ou thèmes à désinstaller
           
      -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags uninstall_plugins`
ou
          `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags uninstall_themes`


##  Version 2

- Backup globale en local (avec une organisation de dossier bien spécifique)
- Restauration globale en local

### Procédure d'exécution:

#### Backup Globale en local

   Se rassurer que les repertoires suivant sont créés chez le client: 
         global_backup_site; global_backup_db; global_backup_plugins; global_backup_themes.

1. Backup globale en local (de tout le site)
         
      -   Se rendre dans le repertoire: `cd /home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags global_backup` 
         
      - Se rendre chez le client dans le repertoire : `ls /home/user/global_backup_site/` 
    pour consulter le fichier globale zippé sous cette forme: 
    `backup_2023-11-29-150808Z_EAZYTraining_RJ80HnU9RO_global.zip`

2. Backup de la base de donnée

      -   Se rendre dans le repertoire: `ls /home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags global_backup_db` 
         
      - Se rendre chez le client dans le repertoire : `ls /home/user/global_backup_db/` 
    pour consulter le fichier globale zippé sous cette forme: 
    `backup_2023-11-29-150808Z_EAZYTraining_RJ80HnU9RO_global_db.sql`

3. Backup globale de tous les plugins :

      -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags plugins_backup` 
         
      - Se rendre chez le client dans le repertoire : `ls /home/user/global_backup_plugins` 
    pour consulter le fichier globale zippé sous cette forme: 
    `backup_2023-11-29-151045Z_EAZYTraining_JJlXbmRtxY_plugins.zip`

4. Backup globale de tous les themes :

      -   Se rendre dans le repertoire: `cd /home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags themes_backup` 
         
      - Se rendre chez le client dans le repertoire : `ls /home/user/global_backup_themes` 
    pour consulter le fichier globale zippé sous cette forme: 
    `backup_2023-11-29-151307Z_EAZYTraining_DivZbqEMYk_themes.zip`
    
#### Restauration Globale en locale

   Se rassurer que les repertoires suivant sont créés chez le client: 
         global_backup_site; global_backup_db; global_backup_plugins; global_backup_themes.

1. Restauration  globale en local (de tout le site)

      - Se rendre dans le fichier :
   
         `vi roles/wordpress/vars/restore_file_name.yml`

           Pour spécifier le fichier archivé du site à restaurer et le repertoire en local où il est situé (`global_backup_site` dans notre cas):
           
           `backup_pattern: "backup_2023-12-11-154653Z_EAZYTraining_PppBXRbPPy_global.zip"
backup_files_path: "{{ compose_project_dir }}/global_backup_site/"`
           
      -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags global_restore`

2. Restauration  de la base de données en local

      - Se rendre dans le fichier :
   
         `vi roles/wordpress/vars/restore_db_name.yml`

           Pour spécifier le fichier  archivé de la base de données à restaurer et le repertoire en local où il est situé (`global_backup_db` dans notre cas):
           
           `restore_pattern: "backup_2023-12-08-120021Z_EAZYTraining_HNXXlsUKhE_global_db.sql"
restore_files_path: "{{ compose_project_dir }}/global_backup_db"`
           
      -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags global_restore_db`

3. Restauration globale de tous les plugins en locale

 Se rendre dans le fichier :
   
   `vi roles/wordpress/vars/restore_plugins_name.yml`
   
Pour spécifier le fichier  archivé du site à restaurer et le repertoire en local où il est situé (`global_backup_plugins` dans notre cas):    

`restore_pattern: "backup_2023-12-12-152806Z_EAZYTraining_8dd5OjIj9H_plugins.zip"
restore_files_path: "{{ compose_project_dir }}/global_backup_plugins"`    

   -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
   - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags global_restore_plugins`

4. Restauration globale de tous les themes en locale

 Se rendre dans le fichier :
   
   `vi roles/wordpress/vars/restore_themes_name.yml`

Pour spécifier le fichier  archivé du site à restaurer et le repertoire en local où il est situé (`global_backup_themes` dans notre cas): 

`restore_pattern: "backup_2023-12-08-132228Z_EAZYTraining_RuOWL6ErRk_themes.zip"
restore_files_path: "{{ compose_project_dir }}/global_backup_themes"`

   -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
   - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags global_restore_themes`

##  Version 3

- Mise à jour et Backup plugin spécifique et theme specifique
- restauration plugin spécifique (rollback)

### Procédure d'exécution

a. Mise à jour d'un plugin spécifique

**NB:** 

**- Lors de la mise à jour d'un plugin spécifique le backup de ce plugin là est effectué en arrière plan en local et sur le bucket de manière automatique.**

**-Au même moment, un fichier yaml est créé automatiquement portant le nom du plugin à l'intérieur du quel nous avons les informations sur le nom du plugin, la version avant la mise à jour ; le nom du plugin, la version après la mise à jour et enfin la date à la quelle le plugins a été mise à jour**

**exemple:** 

**![](https://lh7-us.googleusercontent.com/DyCF7nxO9Go5B6jJzvnYEuoNtksRD2YQ4IlJHsgHM_ub8jop6z4KjymzIbpEusx7_AAyOkW8eC_1VnJCQE8vQiUwGCvt-SIxb5166fWKoTLMEeRrD4qE6jfOe-6pJZU63-g8o-V2rqw6LdS9mCVCsDI)**

  - Se rendre dans le fichier : `vi roles/wordpress/vars/plugin_to_update.yml`
  pour spécifier le nom du plugin à mettre à jour et sa nouvelle version
  
  -  Mettre à jour le plugin
        
        - Consulté la version du plugin actuelle avec la commande:
        `wp plugin get nom_plugin --allow-root --ssh=docker:wordpress`
        
        -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande: 
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags update_plugin`
         
     - Consulté à nouveau la version du plugin avec la commande:
        `wp plugin get nom_plugin --allow-root --ssh=docker:wordpress`
     - Aller chez le client dans le fichier :  `vi specific_backup_plugins/nom_plugin.yaml`
     pour avoir les informations de mise à jour du plugin
     - Consulter chez le client le repertoire suivant pour voire le fichier archivé du plugin avant sa mise à jour: `ls specific_backup_plugins/`

**Remarque: Pour faire le backup et la restauration  d'un plugin spécifique indépendamment de sa mise à jour suivre les étapes ci-après:**

#### Backup spécifiques

a. Backup du plugin spécifique en local
   - Se rendre dans le fichier : `vi roles/wordpress/vars/plugin_to_update.yml`
  pour spécifier le nom du plugin à backuper et version
  
-  Se rendre dans le repertoire: `/home/user/Role-wordpress`
- Taper la commande: 
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags plugin_specific_backup`

b. Backup du plugin spécifique sur le bucket s3
   - Se rendre dans le fichier : `vi roles/wordpress/default/main.yml`
  pour spécifier chez le client le repertoire des plugins spéciques à synchroniser sur le bucket:
  `specific_backup_plugins: "{{ compose_project_dir }}/specific_backup_plugins/"`
  
-  Se rendre dans le repertoire: `/home/user/Role-wordpress`
- Taper la commande: 
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags s3_specific_plugins_backup
  
  #### Restauration spécifique
 
a. Restauration du plugin spécifique en local
   - Se rendre dans le fichier : `vi roles/wordpress/vars/specific_plugin_name_restore.yml`
  
  pour spécifier le nom de l'archive  du plugin à restaurer et le repertoire où il est situé.
  
-  Se rendre dans le repertoire: `/home/user/Role-wordpress`
- Taper la commande: 
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags plugin_specific_restore`

b. Restauration du plugin spécifique depuis le bucket s3
   - Télécharger l'archive du plugin à restaurer depuis le bucket:
	 - Se rendre dans le fichier: `vi roles/wordpress/default/main.yml` pour spécifier le nom du fichier archivé du plugin à télécharger:
`s3_filename_specific_plugins: "backup_2023-12-08-134144Z_EAZYTraining_rhrwTMKhZe_updraftplus_1.23.12.zip"`

     - Et fichier sera stocker dans le repertoire `s3_specific_plugin_restore` chez le client
  
-  Se rendre dans le repertoire: `/home/user/Role-wordpress`
- Taper la commande: 
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags restore_specific_plugins_s3`

b. Mise à jour d'un theme spécifique

**NB:** 

**- Lors de la mise à jour d'un thème spécifique le backup de ce thème là est effectué en arrière plan en local et sur le bucket de manière automatique.**

**-Au même moment, un fichier yaml est créé automatiquement portant le nom du thème à l'intérieur du quel nous avons les informations sur le nom du thème, la version avant la mise à jour ; le nom du thème, la version après la mise à jour et enfin la date à la quelle le thème a été mise à jour**

**exemple:** 

**![](https://lh7-us.googleusercontent.com/MrW3vpZAGURftPtWLF_I9YpFLgK9gLrjqjZkGB79XFQdA88aL3HvGcW2nw4pKKhf4eZ9_Tv-inGRBcJz1qNUwQZiP7C7ACDsltVVznM7a5wdaJxOZWLhMh5TVqrdVdiDm6Pa5jkWNi-Z9c3P1UtgGcc)**

  - Se rendre dans le fichier : `vi roles/wordpress/vars/theme_to_update.yml`
  pour spécifier le nom du theme à mettre à jour et sa nouvelle version
  
  -  Mettre à jour le theme
        
        - Consulté la version du theme actuelle avec la commande:
        `wp theme get nom_plugin --allow-root --ssh=docker:wordpress`
        
        -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande: 
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags update_theme`
         
     - Consulté à nouveau la version du plugin avec la commande:
        `wp theme get nom_plugin --allow-root --ssh=docker:wordpress`
     - Aller chez le client dans le fichier :  `vi specific_backup_themes/nom_theme.yaml`
     pour avoir les informations de mise à jour du theme
     - Consulter chez le client le repertoire suivant pour voire le fichier archivé du theme avant sa mise à jour: `ls specific_backup_themes/`

**Remarque: Pour faire le backup et la restauration  d'un theme spécifique indépendamment de sa mise à jour suivre les étapes ci-après:**

#### Backup spécifiques

a. Backup du theme spécifique en local
   - Se rendre dans le fichier : `vi roles/wordpress/vars/theme_to_update.yml`
  pour spécifier le nom du theme à backuper et version
  
-  Se rendre dans le repertoire: `/home/user/Role-wordpress`
- Taper la commande: 
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags theme_specific_backup`

b. Backup du theme spécifique sur le bucket s3
   - Se rendre dans le fichier : `vi roles/wordpress/default/main.yml`
  pour spécifier chez le client le repertoire des plugins spéciques à synchroniser sur le bucket:
  `specific_backup_themes: "{{ compose_project_dir }}/specific_backup_themes/"`
  
-  Se rendre dans le repertoire: `/home/user/Role-wordpress`
- Taper la commande: 
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags s3_specific_themes_backup`
  
  #### Restauration spécifique
 
a. Restauration du plugin spécifique en local
   - Se rendre dans le fichier : `vi roles/wordpress/vars/specific_theme_name_restore.yml`
  
  pour spécifier le nom de l'archive  du theme à restaurer et le repertoire où il est situé.
  
-  Se rendre dans le repertoire: `/home/user/Role-wordpress`
- Taper la commande: 
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags theme_specific_restore`

b. Restauration du theme spécifique depuis le bucket s3
   - Télécharger l'archive du theme à restaurer depuis le bucket:
	 - Se rendre dans le fichier: `vi roles/wordpress/default/main.yml` pour spécifier le nom du fichier archivé du plugin à télécharger:
`s3_filename_specific_themes: "backup_2023-12-09-171502Z_EAZYTraining_7ybgX73bqz_twentytwenty_1.1.zip"`

     - Et ce fichier sera stocker dans le repertoire `s3_specific_theme_restore` chez le client
  
-  Se rendre dans le repertoire: `/home/user/Role-wordpress`
- Taper la commande: 
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags restore_specific_themes_s3`


## Version 4

- Backup globale sur google drive, S3
- Restauration globale sur google drive, S3

### Procédure d'exécution:

#### Backup Globale sur le bucket s3
   Se rassurer que les repertoires suivant sont créés chez le client: 
         global_backup_site; global_backup_db; global_backup_plugins; global_backup_themes.

1. Backup globale sur le bucket s3 (de tout le site)
         
      -   Se rendre dans le repertoire: `cd /home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags backup_site_s3` 
         
      - Se rendre sur le bucket de la console aws et choisir le nom du bucket où le backup a été fait pour consulter le fichier globale zippé sous cette forme: 
   `backup_2023-11-29-150808Z_EAZYTraining_RJ80HnU9RO_global.zip`

2. Backup de la base de données sur le bucket s3

      -   Se rendre dans le repertoire: `ls /home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags backup_db_s3` 
         
      - Se rendre sur le bucket de la console aws et choisir le nom du bucket où le backup a été fait pour consulter le fichier globale zippé sous cette forme: 
    `backup_2023-11-29-150808Z_EAZYTraining_RJ80HnU9RO_global_db.sql`

3. Backup globale de tous les plugins sur le bucket s3:

      -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags s3_global_plugins_backup` 
         
      - Se rendre sur le bucket de la console aws et choisir le nom du bucket où le backup a été fait pour consulter le fichier globale zippé sous cette forme: 
    `backup_2023-11-29-151045Z_EAZYTraining_JJlXbmRtxY_plugins.zip`

4. Backup globale de tous les themes sur le bucket s3:

      -   Se rendre dans le repertoire: `cd /home/user/Role-wordpress`
      - Taper la commande: 
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags s3_global_themes_backup` 
         
      - Se rendre sur le bucket de la console aws et choisir le nom du bucket où le backup a été fait pour consulter le fichier globale zippé sous cette forme: 
    `backup_2023-11-29-151307Z_EAZYTraining_DivZbqEMYk_themes.zip`

#### Restauration Globale sur le bucket s3
   Se rassurer que les repertoires suivant sont créés chez le client: 
         s3_site_restore; s3_db_restore; s3_global_plugin_restore; s3_global_theme_restore.

1. Restauration  globale à partir du bucket s3 (de tout le site)

    a. Télécharger le fichier zipper du site à restaurer dans le repertoire `s3_site_restore`
      
      - Se rendre dans le fichier :
   
         `vi roles/wordpress/default/main.yml`

           Pour spécifier le fichier archivé du site à restaurer :
           
           `s3_filename_site: "backup_2023-12-11-154653Z_EAZYTraining_PppBXRbPPy_global.zip"`
         
      b. Restaurer le site en recuperant l'archive dans le repertoire `s3_site_restore`
  
   -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags restore_site_s3`

2. Restauration  de la base de données à partir du bucket s3

      a. Télécharger le fichier zipper de la base de données à restaurer dans le repertoire `s3_db_restore`
      
      - Se rendre dans le fichier :
   
         `vi roles/wordpress/default/main.yml`

           Pour spécifier le fichier archivé du site à restaurer :
           
           `s3_filename_db: "backup_2023-12-08-120021Z_EAZYTraining_HNXXlsUKhE_global_db.sql"`
         
      b. Restaurer la bd en recuperant l'archive dans le repertoire `s3_db_restore`
  
   -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags restore_db_s3`
         
3. Restauration globale de tous les plugins à partir du bucket s3

      a. Télécharger le fichier zipper des plugins à restaurer dans le repertoire `s3_global_plugin_restore`
      
      - Se rendre dans le fichier :
   
         `vi roles/wordpress/default/main.yml`

           Pour spécifier le fichier archivé du site à restaurer :
           
           `s3_filename_plugins: "backup_2023-12-08-122719Z_EAZYTraining_haEVITTfu9_plugins.zip"`
         
      b. Restaurer les plugins en recuperant l'archive dans le repertoire `s3_global_plugin_restore`
  
   -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags restore_plugins_s3`

4. Restauration globale de tous les themes à partir du bucket s3

 
      a. Télécharger le fichier zipper des plugins à restaurer dans le repertoire `s3_global_theme_restore`
      
      - Se rendre dans le fichier :
   
         `vi roles/wordpress/default/main.yml`

           Pour spécifier le fichier archivé du site à restaurer :
           
           `s3_filename_themes: "backup_2023-12-08-132228Z_EAZYTraining_RuOWL6ErRk_themes.zip"`
         
      b. Restaurer les plugins en recuperant l'archive dans le repertoire `s3_global_theme_restore`
  
   -   Se rendre dans le repertoire: `/home/user/Role-wordpress`
      - Taper la commande: 
      
         `ansible-playbook -i hosts.yml --ask-vault-pass -vvv wordpress_operations.yml --tags restore_themes_s3`

## Version 5

- intégration de molecule
- Intégration des  pré-commit hooks

##  Version 6

- libraison du rôle sur la galaxie
