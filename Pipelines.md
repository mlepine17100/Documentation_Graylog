# Graylog – Documentation complète des Pipelines

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