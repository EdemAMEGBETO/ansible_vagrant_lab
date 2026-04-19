"# ansible_vagrant_lab"  

1. Lancement de l’infrastructure Vagrant
> vagrant up

Nous devons ensuite vérifier que les 3 machines sont up :
> vagrant status

2. Configuration Ansible
- Nous allons pinger toutes les machines
> ansible all -i inventory -m ping

- Ensuite Récupérer les facts
> ansible all -i inventory -m setup

3. Configuration du serveur web (Nginx)
- Nous allons procéder à l'installation de Nginx
> ansible web -i inventory -m apt -a "name=nginx state=present update_cache=yes" --become

- Ensuite procéder à l'activation du service
> ansible web -i inventory -m service -a "name=nginx state=started enabled=yes" --become

- Déploiement de la page personnalisée
> ansible web -i inventory -m template -a "src=templates/index.html.j2 dest=/var/www/html/index.nginx-debian.html" --become

- Et pour finir effectuer la vérification
>ansible web -i inventory -a "curl http://localhost"

4. Configuration du serveur DB (MariaDB)
- Nous allons procéder à l'installation
> ansible db -i inventory -m apt -a "name=mariadb-server state=present update_cache=yes" --become

- Ensuite à l'activation du service
> ansible db -i inventory -m service -a "name=mariadb state=started enabled=yes" --become

- Création de la base "devsecopsdojo"
> ansible db -i inventory -m mysql_db -a "name=devsecopsdojo state=present" --become

- Et pour finir effectuer la vérification
> ansible db -i inventory -a "mysql -e 'SHOW DATABASES;'"


