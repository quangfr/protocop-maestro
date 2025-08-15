# 🚀 Protocop GO-LIVE : prompt GPT pour créer un prototype HTML

**🎯 But du doc :** cadrer un **prototype HTML autonome** (offline, sans dépendance)

**📌 Objectifs d'un prototype HTML :**

- 💡 Proposer et valider une vision produit
- 🗂️ Aider à structurer une feuille de route et à prioriser les fonctionnalités à haute valeur ajoutée
- 🖥️ Discuter et valider des idées d'interface ou un modèle de données
- 📑 Illustrer et préciser une spécification avec les développeurs comme les usagers

**🛠 Usages :**

- 🖼️ Copie d’écran
- 🎥 Enregistrement vidéo court (1 min)
- 📊 Illustration des données et des cas d'exemple
- 👥 Tests utilisateurs (UI/UX)

**📥 Dépôt :**
- readme.md : questionnaire (et ex synthétique MAESTRO) à mettre dans GPT pour créer son prompt
- prompt.md : exemple de prompt complet MAESTRO à mettre dans GPT pour générer le prototype
- prototype.html : exemple de prototype MAESTRO à télécharger et à ouvrir dans un navigateur


**🤖 Astuces**

- 🔍 Prototyper petit d’abord
- ✏️ Demander à GPT de te poser des questions pour préciser ton prompt
- 🆘 Demander à GPT de te proposer des pistes d'améliorations
- 🔄 Mettre le plus de demandes possibles dans ton prompt avant régénération
- 📥 Demander à GPT de re-décrire ton prompt à partir du code ou de tes demandes additionnelles
- 📋 Plus ton prompt est précis, plus ton prototype sera fidèle mais plus la génération prend du temps
  
---

## 1️⃣ Identité

❓ *Quel nom afficher dans le bandeau en haut ?*  
❓ *Quel style général ?*  
❓ *En une phrase, à quoi sert cet outil ?*

**Exemple : MAESTRO**

- **Nom affiché :** MAESTRO
- **Style :** Thème clair SAP‑like
- **Objectif :** Créer/éditer des demandes et opérations de maintenance sur les moteurs d'avion, piloter les capacités et afficher des KPIs, le tout offline dans un fichier HTML unique sans dépendances externes.

---

## 2️⃣ Modèle de données

❓ *Quels sont les objets métiers de l'application ? (ex. Demande, Opération, Ressource, Calendrier…)*  
❓ *Pour chaque objet : propriétés, validations, permissions ?*  
❓ *Relations entre objets : cardinalités, contraintes (suppression en cascade, restrictions, compatibilités, relations obligatoires, limites quantitatives, contraintes temporelles, unicité, synchronisation d’état)*  
❓ *Quelles données en exemple ?*

**Exemple : MAESTRO**

- **Objets principaux :** Demande, Opération, Listes maîtres, Mapping, Durées, Capacités atelier
- **Relations :** Demande 1..N Opération, Shop 1..N Operation Types, Request Type 1..4 Operation Types
- **Données d’exemple :** Urgency (4), Status (4), 10 shops, 5 locations, 12 ops types, 5 modèles moteur, 5 clients, 3 request types (4 ops chacune), durées 1–5 semaines, 10 demandes, 30 opérations

---

## 3️⃣ Navigation et écrans

❓ *Quels onglets/écrans veux-tu ?*  
❓ *Pour chaque onglet : données, actions, aides ?*  
❓ *Comment naviguer entre les onglets ?*  
❓ *Comportement entre écrans ?*

**Exemple : MAESTRO**

- **Nouvelle demande** : création avec helper panel (4 ops requises, capacity lookup)
- **Éditer opérations** : ajout/édition avec helper (opérations autorisées par shop)
- **Demandes** : tableau filtrable, IDs cliquables vers opérations
- **Opérations** : tableau filtrable, IDs cliquables vers demandes
- **Listes maîtres** : édition des listes, mapping, durées, JSON Viewer, Apply changes
- **KPI** : On-time % urgent + heatmap

---

## 4️⃣ Techniques

❓ *Quelles fonctionnalités avancées ?*  
❓ *Quelles contraintes techniques ?*

**Exemple : MAESTRO**

- HTML unique, offline, Vanilla JS, dropdowns dynamiques, import/export JSON, recherche, entêtes sticky, thème clair SAP-like
