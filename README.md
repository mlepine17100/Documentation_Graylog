# Pré-requis

### Avant de lancer les conteneurs, il faut générer le .env avec les valeurs

#### 

##### Pour générer le secret de minimum 6 caractères :
```bash
echo "GRAYLOG_PASSWORD_SECRET=\"$(pwgen -N 1 -s 96)\"" >> /opt/graylog/.env
```

##### Pour générer le mot de passe de l'utilisateur admin chiffré : 
```bash
echo "GRAYLOG_ROOT_PASSWORD_SHA2=\"$(echo -n 'mot_de_passe' | shasum -a 256 | awk '{print $1}')\"" >> /opt/graylog/.env
```

### Il faut aussi mettre en place certains dossiers qui seviront de volumes avec le docker-compose
#### Dans le dossier /opt/graylog

```bash
mkdir -p .mongodb_data .mongodb_config .graylog_data .graylog-datanode
```