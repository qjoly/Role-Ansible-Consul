Role Ansible Consul Server
=========
## Introduction
Projet permettant de déployer un serveur Consul, puis d'y configurer des clients. 
Le role est conçu de manière modulaire permettant d'ajouter ses propres services (en dehors de ceux par défaut). 

Variables pour configurer Consul.

| Variable             | Description                                                              | Emplacement       | valeur par défaut        |
|----------------------|--------------------------------------------------------------------------|-------------------|--------------------------|
| consul_data_dir      |  Dossier contenant les fichiers de données                                                                        | defaults/main.yml | /var/lib/consul          |
| consul_config_dir    | Fichiers de configuration ( + services) de consul                        | defaults/main.yml | /etc/consul.d            |
| consul_datacenter    | Nom du datacenter Consul                                                 | defaults/main.yml | datacenter               |
| consul_log_level     | Niveau de log de Consul                                                  | defaults/main.yml | INFO                     |
| consul_enable_syslog | Activer syslog                                                           | defaults/main.yml | true                     |
| consul_group         | Groupe qui sera utilisé par Consul                                       | defaults/main.yml | consul                   |
| consul_user          | Utilisateur qui sera utilisé par Consul                                  | defaults/main.yml | consul                   |
| consul_encrypt_key   | Clé partagée de chiffrement pour Consul (à générer avec "consul keygen") | vars/main.yml     | TeLbPpWX41zMM3vfLwHHfQ== |
| consul_tmpl_dir      | Dossier utilisé pour déposer les templates                               | defaults/main.yml | /var/consul_tmpl         |

Variables d'installation de Consul.

| Variable               | Description                                                                                                 | valeur par défaut                                                          |
|------------------------|-------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------|
| consul_dl_url          | (Optionnel) URL pour télécharger le binaire (si non précisé, le role enverra le binaire dans files/consul)  | https://releases.hashicorp.com/consul/1.12.1/consul_1.12.1_linux_amd64.zip |
| dir_consul_binary      | Emplacement de dépot du binaire consul                                                                      | /usr/local/bin/                                                            |
| service_consul_systemd | Fichier contenant le service (systemd) de consul                                                            | /etc/systemd/system/consul.service                                         |
| checksum_consul_binary | (Utilisé uniquement si consul_dl_url n'est pas défini)                                                      | 6efa2544faf05835663803522c8f0e0f77c58d05                                   |

## Ajouter ses propres roles

Le role incorpore un déploiement des services Consul automatique. Celui-ci va récupérer la liste des programmes installés sur les machines clientes, et va y configurer les services. Pour configurer le service "Nginx", il faudra : 

- Créer le JSON avec les metadonnées du service (dans *files/consul_services*) [**à nommer nginx.json**]
```json
{"service":
    {
    "name": "nginx", 
    "tags": ["web"], 
    "port": 80,
    "check": {
       "id": "nginx",
       "name": "Checking if nginx is running",
       "args": ["cat", "/var/run/nginx.pid"],
       "interval": "10s",
       "timeout": "5s"
     }
   }
   }
```
- (Si Consul-template) Ajouter les variables nécéssaires pour la template (dans *vars/main.yml*)

```yaml
  - service_name: "nginx"
    config_location: "/etc/nginx/sites-available/default"
    reload_command: "service nginx restart"
```

- (Si Consul-template) Ajouter une template 
```nginx
server {
{{ with service "nginx" }}
{{ with index . 0 }}
  listen {{ with index (service "nginx") 0 }}{{ .Port }}{{ end }} default_server;
{{ end }}{{ end }}
  
	root /var/www/html;
	index index.html index.htm index.nginx-debian.html;

	server_name _;
	location / {
		try_files $uri $uri/ =404;
	}

}
```

