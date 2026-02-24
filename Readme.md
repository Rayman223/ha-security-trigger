# 🏠 Home Assistant — Système d'Alarme Sécurité

Automatisation complète d'une alarme de sécurité pour Home Assistant, avec gestion des horaires, blocage manuel, détection de mouvement et notifications Discord.

---

## 📋 Vue d'ensemble

Le système gère automatiquement l'activation et la désactivation d'une alarme selon une plage horaire configurable. Il envoie des notifications Discord et supporte un mode manuel qui suspend temporairement l'automatisation.

```
┌─────────────────────────────────────────────────────────┐
│                    FLUX PRINCIPAL                        │
│                                                         │
│  Heure d'activation ──► Alarme ON  ◄── Absent de chez soi │
│  Heure de désactivation ► Alarme OFF                    │
│  Action manuelle ──────► Blocage actif                  │
│  Retour à la maison ───► Réinitialisation du blocage    │
└─────────────────────────────────────────────────────────┘
```

---

## 🧩 Helpers requis

| Entité | Type | Rôle |
|--------|------|------|
| `input_boolean.statut_alarme_securite` | Boolean | État de l'alarme (ON/OFF) |
| `input_boolean.securite_blocage_manuel_actif` | Boolean | Verrou anti-automatisation |
| `input_datetime.securite_heure_activation` | Datetime (heure) | Heure de mise en service |
| `input_datetime.securite_heure_desactivation` | Datetime (heure) | Heure de mise hors service |

---

## ⚙️ Automatisations

### 1. `Alarme Sécurité - Activation Automatique`

**Rôle :** Cerveau du système. Active ou désactive l'alarme selon l'heure courante.

**Déclencheurs :**
- À l'heure d'activation
- À l'heure de désactivation
- Au démarrage de Home Assistant
- Lors d'un changement des heures configurées

**Logique :**
- Si l'heure courante est dans la plage d'activation → `statut_alarme_securite = ON`
- Sinon → `statut_alarme_securite = OFF`

**Condition bloquante :** Ne s'exécute pas si `securite_blocage_manuel_actif = ON`.

> ⚠️ Supporte les plages à cheval sur minuit (ex : 23:00 → 07:00).

---

### 2. `Alarme Sécurité - Blocage après Action Manuelle`

**Rôle :** Détecte quand l'utilisateur modifie l'alarme manuellement et suspend l'automatisation.

**Déclencheur :** Changement de `statut_alarme_securite` (ON ou OFF).

**Condition :** L'action doit provenir d'un utilisateur réel (`user_id` présent et `parent_id` nul — ce qui exclut les automatisations).

**Actions :**
1. Active `securite_blocage_manuel_actif`
2. Envoie une notification Discord confirmant la suspension

---

### 3. `Alarme Sécurité - Réinitialisation Mode Manuel`

**Rôle :** Lève le blocage manuel quand les conditions normales sont réunies.

**Déclencheurs :**
- Rayman rentre à la maison
- À l'heure de désactivation

**Conditions :**
- Rayman est à la maison
- L'heure courante est dans la plage de désactivation (jour)

**Actions :**
1. Éteint `statut_alarme_securite` et `securite_blocage_manuel_actif`
2. Envoie une notification Discord
3. Re-déclenche `Alarme Sécurité - Activation Automatique` après 2 secondes

---

### 4. `Discord - Mouvement Nuit avec Photo`

**Rôle :** Alerte en temps réel lors d'un mouvement détecté dans le salon quand l'alarme est active.

**Déclencheur :** `binary_sensor.salon_person_occupancy` passe à `on`.

**Condition :** `statut_alarme_securite = ON`.

**Actions :**
1. Capture un snapshot de `camera.salon`
2. Envoie une notification Discord avec la photo, l'heure et le nombre de personnes détectées

---

## 🔄 Schéma de fonctionnement

```
                         ┌──────────────────┐
                         │  HA démarre /    │
                         │  Heure change    │
                         └────────┬─────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │  Blocage Manuel Actif ?    │
                    └──────┬──────────┬──────────┘
                          OUI        NON
                           │          │
                    Aucune action   Dans la plage ?
                                  ┌───┴───┐
                                 OUI     NON
                                  │       │
                              Alarme ON  Alarme OFF
                                  │
                         Mouvement détecté ?
                                  │
                              Notification
                              Discord + Photo
                                  
──────────────────────────────────────────────────

  Action manuelle ──► Blocage activé ──► Notification Discord
  
  Rayman rentre ──► Blocage levé ──► Alarme OFF ──► Reprise auto
```

---

## 📦 Structure des fichiers

```
├── Automatisation/
│   ├── Alarme Sécurité - Activation Automatique.yaml
│   ├── Alarme Sécurité - Blocage après Action Manuelle.yaml
│   ├── Alarme Sécurité - Réinitialisation Mode Manuel.yaml
│   └── Discord - Mouvement Nuit avec Photo.yaml
├── Helpers/
│   └── List helpers.md
├── README.md
└── LICENSE
```

---

## 🔧 Configuration

1. Créer les helpers listés dans le tableau ci-dessus via l'interface Home Assistant.
2. Importer les fichiers YAML dans le répertoire `automations/` de votre configuration HA.
3. Configurer le `channel_id` Discord dans les automatisations (remplacer `1459865652679675915`).
4. Remplacer `person.rayman_2` par votre propre entité `person`.
5. Remplacer `camera.salon` et `binary_sensor.salon_person_occupancy` par vos entités de caméra/capteur.
6. Recharger les automatisations (Outils de développement → YAML → Recharger les automatisations).

---

## 📄 Licence

MIT © 2026 Rayman223