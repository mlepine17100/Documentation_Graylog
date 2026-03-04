# Documentation d'utilisation : Graylog (7.0.4)

## 1 Mise en place de l'Index Set (Stockage & Rétention)

L'Index Set définit la manière dont les messages sont écrits sur le disque et combien de temps ils sont conservés avant d'être purgés.

### 1. Mise en place d'un template

Pour faciliter la mise en place des index avec les bonnes pratiques à respecter sur la conservation des logs.

#### 1.1 Créer le template

Pour cela, il faut se rendre sur **`System`** > **`Indices`** > **`Index Set Templates`** > **`Create Template`**


#### 1.2 Paramétrage technique (Optimisation des ressources)
Pour garantir les performances du serveur virtuel sans surcharger l'hyperviseur :
* **Index Shards :** `1` (Optimisé pour un serveur unique).
* **Index Replicas :** `0` (Évite de doubler l'espace disque inutilement).
* **Index Optimization :** `Activé` (Laisser décochée la case, compresse les anciens logs pour gagner de la place).

#### 1.3 Politique de Rétention (Objectif : 9 mois)
Conformément à la politique de sécurité de l'infrastructure, nous configurons un cycle de vie de **270 jours**.

| Paramètre | Valeur | Explication |
| :--- | :--- | :--- |
| **Max. days in storage** | `275` | Suppression définitive après ce délai (en jours). |
| **Min. days in storage** | `270` | Durée minimale de conservation garantie (9 mois = 270 jours). |

> **⚠️ Attention :** Le stockage de 9 mois de logs peut prendre une place assez conséquente sur le disque, penser à augmenter la partition LVM (root) pour augmenter la taille du / .

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

### Isoler les logs SSH

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

# Documentation Graylog – Mise en place des Pipelines

---

# 1. Introduction

Les **Pipelines** dans Graylog permettent de traiter, transformer, enrichir et router les messages avant leur indexation.

On peut utiliser les Pipelines pour :

- Normaliser les champs
- Enrichir les messages
- Ajouter des tags
- Router dynamiquement vers des Streams
- Nettoyer les données inutiles (les supprimer)

---

# 2. Fonctionnement général

## 2.1 Ordre de traitement d’un message

1. Message reçu via Input
2. Stockage temporaire dans le Journal
3. Exécution des Pipelines
4. Évaluation des Streams
5. Indexation dans Graylog Datanode

Les Pipelines s'exécutent **avant l’assignation finale aux Streams**.

---

# 3. Structure d’une Pipeline

Une Pipeline est composée de :

- Stages
- Règles
- Connexion à un Stream

---

## 3.1 Stages

Un stage représente une étape logique.

Exemple :

- Stage 0 → Extraire une IP
- Stage 1 → Utiliser le nouveau champ où est extrait l'IP
- Stage 2 → Créer un nouveau champ se servant de l'IP

Les stages s’exécutent dans l’ordre numérique.

---

## 3.2 Structure d’une règle

Une règle contient :

- `when` → condition
- `then` → actions
- `end`

Exemple simple :

```json
rule "add environment"
when
    has_field("source") && contains(to_string($message.source), "SVP")
then
    set_field("environnement", "PROD");
end
```

---

# 4. Mise en place d’une Pipeline

## Étape 1 : Créer une règle

Menu :

`System → Pipelines → Manage Rules`

Créer une nouvelle règle.

---

## Étape 2 : Créer une Pipeline

`System → Pipelines → Manage Pipelines`

- Ajouter un Stage
- Associer les règles

---

## Étape 3 : Connecter la Pipeline à un Stream

`System → Pipelines → {new_pipeline} → Edit Connections`

Associer la pipeline au Stream souhaité.

**Sans connexion à un Stream, la Pipeline ne s’exécute pas.**

---

# 5. Fonctions principales utilisées

## 5.1 Manipulation de champs

| Fonction | Description |
|----------|------------|
| set_field() | Ajouter/modifier un champ |
| remove_field() | Supprimer un champ |
| rename_field() | Renommer un champ |
| has_field() | Vérifier existence |

---

## 5.2 Manipulation de texte

| Fonction | Description |
|----------|------------|
| contains() | Vérifie présence texte |
| regex() | Correspondance regex |
| lowercase() | Met en minuscule |
| uppercase() | Met en majuscule |
| split() | Découpe chaîne |

---

# 6. Exemples pratiques

---

## 6.1 Tag sécurité automatique

```javascript
rule "tag failed login"
when
    contains(to_string($message.message), "Failed password")
then
    set_field("log_type", "SECURITY");
    set_field("severity", "HIGH");
end
```

---

## 6.2 Routing dynamique vers un Stream

```javascript
rule "route security stream"
when
    contains(to_string($message.message), "Failed password")
then
    route_to_stream("{nom_stream_voulu}");
end
```

---

# Documentation Graylog – Mise en place des Alertes
## Configuration de base de l’Alerting

---

## 1. Introduction

Le système d’alerting de Graylog permet de détecter automatiquement des événements spécifiques dans les logs et d’envoyer des notifications (email, Slack, webhook, etc.).

L’alerting repose sur :

- Event Definitions
- Conditions (filtres + seuils)
- Notifications
- Event Streams

---

## 2. Architecture simplifiée de l’Alerting

Flux de fonctionnement :

1. Un message arrive
2. Il correspond à un Stream
3. Une Event Definition analyse ce Stream
4. Si la condition est remplie
5. Une Notification est envoyée

---

## 3. Prérequis

- Un Stream existant (ex : SECURITY-STREAM)
- Serveur SMTP configuré si alertes email
- Permissions administrateur

---

## 4. Configuration d’une alerte simple (exemple)

### Cas d'exemple :
Détecter plus de 5 échecs SSH en 5 minutes.

---

## 5. Étape 1 : Créer une Event Definition

Menu :

`Alerts → Event Definitions → Create Event Definition`

---


### Titre

```
Multiple SSH Failed Login
```

### Condition Type

```
Filter & Aggregation
```

---

#### Search Query

Exemple simple :

```
message:"Failed password"
```

---

#### Streams 

Sélectionner le stream voulu.


**Important pour éviter de scanner tous les logs.**

---

#### Search within the last

```
5 minutes
```
*Cherche dans les 5 dernières minutes*

---

#### Execute search every

```
1 minute
```

Permet une détection plus réactive, regarde toutes les minutes les logs.

---

### Create Events for Definition if...

**Ne PAS laisser** :

```
Filter has results
```

Cela déclenche une alerte à chaque log.

---

Sélectionner :

```
Aggregation of results reaches a threshold
```

Envoi une liste de logs en une seule alerte

---

Configurer :

##### Group By (recommandé)

```
source - string
```

Permet de déclencher une alerte par IP.

---

##### Function

```
count()
```

---

##### Condition

```
>
```

---

##### Threshold

```
5
```


## 6. Étape 2 : Créer une Notification

Menu :

`Alerts → Notifications → Create Notification`

---

## 6.1 Exemple Notification Email

### Type :

```
Email Notification
```

### Paramètres :

- Recipients : test@groupecgo.fr (mail où envoyer l'alerte)
- Subject :

```
ALERTE SSH - ${event.source}
```

- Body :

```
Une tentative de brute force SSH a été détectée.

Source : ${event.source}
Nombre d'échecs : ${event.fields.count}
```

---

## 7. Lier la Notification à l’Event

Dans l’Event Definition :

Section :

```
Notifications → Add Notification
```

Sélectionner la notification créée.

---

## 8. Test de l’alerte

Utiliser :

```
Alerts → Event Definitions → Execute Test
```

Ou générer volontairement des logs correspondant à la condition.

---