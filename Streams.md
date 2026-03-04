# Documentation Graylog – Mise en place des Streams

## 1. Introduction

Dans **Graylog**, les *Streams* permettent de router, organiser et filtrer les messages entrants selon des règles définies et les lier à des index précis.

Ils constituent un élément central pour :

- Séparer les logs par application
- Gérer la rétention différente par type de données
- Appliquer des alertes spécifiques

Un message peut appartenir à **plusieurs streams** simultanément.

---

## 2. Prérequis

- Un input configuré (minimum)
- Droits de création des streams

---

## 3. Fonctionnement des Streams

Lorsqu’un message arrive dans Graylog :

1. Il est d'abord placé dans le **Default Stream**.
2. Graylog évalue les règles des autres streams.
3. Si le message correspond aux règles définies :
   - Il est assigné au stream correspondant.
   - Il peut être retiré du Default Stream (optionnel).

Les streams utilisent des **règles basées sur les champs des messages**, par exemple :

- `source`
- `message`
- `level`

---

## 4. Création d’un Stream

### Étape 1 : Accéder au menu

`Streams`

Cliquez sur **Create Stream**.

### Étape 2 : Paramètres principaux

| Paramètre | Description |
|-----------|-------------|
| Title | Nom du stream |
| Description | Description fonctionnelle |
| Index Set | Groupe d’index associé |
| Remove matches from Default Stream | Retire les messages du flux par défaut |

Cliquez sur **Save**.

---

## 5. Ajout de règles au Stream

Une fois le stream créé :

1. Cliquez sur le stream
2. Sélectionnez **Manage Rules**
3. Ajoutez une règle

### Types de règles disponibles

| Type de règle | Description |
|---------------|-------------|
| Match exactly | Correspondance exacte |
| Match regular expression | Correspondance par regex |
| Greater than | Supérieur à |
| Smaller than | Inférieur à |
| Field present | Champ présent |

---

## 6. Exemple pratique

### Cas : Isoler les logs SSH

Pour isoler des logs SSH de serveur Linux.

Règle possible :

- Field: `message`
- Type: `Match regular expression`
- Value: `sshd`


---


## 7. Références officielles

- Documentation Graylog – Streams  
  https://go2docs.graylog.org/

- Documentation Index Sets  
  https://go2docs.graylog.org/current/setting_up_graylog/index_model.html

---
