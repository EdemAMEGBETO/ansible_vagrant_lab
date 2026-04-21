"# ansible_vagrant_lab"  

1. Lancement de l’infrastructure Vagrant
> vagrant up

Nous devons ensuite vérifier que les 3 machines sont up :
> vagrant status

2. Préparation des clés SSH pour Ansible

Vagrant génère une clé SSH privée unique par VM. Il faut regrouper la clé la plus récente
(ou copier celle de n'importe quelle VM, elles sont identiques avec les boxes récentes) dans
un fichier commun référencé dans `ansible.cfg`.

# Récupérer la clé SSH générée par Vagrant (identique pour toutes les VM avec bento/ubuntu) sur mon WSL car j'ai install
> mkdir -p ~/vagrant-keys
> cp .vagrant/machines/web/virtualbox/private_key ~/vagrant-keys/web.key
> cp .vagrant/machines/db/virtualbox/private_key ~/vagrant-keys/db.key
> cp .vagrant/machines/lb/virtualbox/private_key ~/vagrant-keys/lb.key
> chmod 600 ~/vagrant-keys/*.key


3. Vérification de la connectivité Ansible


# Ping sur tous les hôtes
> ansible all -m ping

# Ping par groupe
> ansible web -m ping
> ansible db  -m ping
> ansible lb  -m ping


Résultat attendu pour chaque hôte :
```yaml
web | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

# Vérifier les informations système de toutes les VM
> ansible all -m setup -a "filter=ansible_distribution*"

4. Configuration du serveur Web (Nginx)

4.1 — Mise à jour du cache APT et installation de NGINX

> ansible web -i inventory -b -m apt -a "name=nginx state=present update_cache=yes"

4.2 — Vérification que Nginx écoute sur le port 80

> ansible web -i inventory -m shell -a "ss -tlnp | grep ':80'"

4.3 — Création de la page HTML personnalisée

> ansible web -i inventory -b -m copy -a "src=templates/index.html.j2 dest=/var/www/html/index.html"

4.4 — Vérification du contenu de la page

> ansible web -i inventory -m shell -a "cat /var/www/html/index.html"

5. Configuration du serveur DB (MariaDB)

# Definition des variables ansible et 
> export ANSIBLE_BECOME=True
> export ANSIBLE_BECOME_USER=root
> export ANSIBLE_BECOME_METHOD=sudo
> export ANSIBLE_BECOME_ASK_PASS=False
> touch /tmp/empty.cfg
> mv ansible.cfg ansible.cfg.backup
> export ANSIBLE_CONFIG=/tmp/empty.cfg

5.1 — Mise à jour du cache APT

> ansible db -i inventory -m apt -a "update_cache=yes"

5.2 — Installation de MariaDB et du module Python requis

> ansible db -i inventory -m apt -a "name=mariadb-server state=present"
> ansible db -i inventory -m apt -a "name=python3-pymysql state=present"

5.3 — Démarrage et activation de MariaDB au boot

> ansible db -i inventory -m service -a "name=mariadb state=started enabled=yes"

5.4 — Vérification que MariaDB écoute sur le port 3306

> ansible db -i inventory -m shell -a "ss -tlnp | grep ':3306'"

5.5 — Création de la base de données `devsecopsdojo`

> ansible db -i inventory -m mysql_db -a "name=devsecopsdojo state=present login_unix_socket=/run/mysqld/mysqld.sock"

5.6 — Création de l'utilisateur dédié à la base

> ansible db -i inventory -m mysql_user -a "name=dojo_user password=Dojo@2024! priv='devsecopsdojo.*:ALL' host='%' state=present login_unix_socket=/run/mysqld/mysqld.sock"

5.7 — Autoriser MariaDB à écouter sur toutes les interfaces (pas seulement localhost)

Par défaut MariaDB n'écoute que sur `127.0.0.1`. Il faut modifier `bind-address` pour que
HAProxy puisse atteindre le port 3306 depuis `192.168.56.12`.

# Créer le fichier de configuration override
> ansible db -i inventory -m copy -a "content='[mysqld]\nbind-address = 0.0.0.0\n' dest=/etc/mysql/mariadb.conf.d/99-bind.cnf owner=root group=root mode=0644"

# Redémarrer MariaDB pour appliquer le changement
ansible db -i inventory -m service -a "name=mariadb state=restarted"

5.8 — Vérification finale

# MariaDB écoute maintenant sur 0.0.0.0:3306
> ansible db -i inventory -m shell -a "ss -tlnp | grep ':3306'"

# Vérifier la présence de la base devsecopsdojo
> ansible db -i inventory -m shell -a "mysql -u root -e 'SHOW DATABASES;'"



