# Feedback : Présentation HQFBP pour Amsat-F

**Date du feedback** : 2026-02-01  
**Présentation** : Hamradio Quick File Broadcasting Protocol (HQFBP)  
**Événement** : Rencontres Spatiales Radioamateur de l'Amsat-F (2026-03-06)  
**Durée** : 20 minutes

---

## Analyse Critique de la Présentation

### ✅ **Points Forts**

1. **Titre accrocheur** : "Comment réinventer la roue ...mais en se couchant moins bête !" est excellent - il capte l'attention et désamorce avec humour la critique classique du "NIH syndrome" (Not Invented Here).

2. **Structure narrative solide** : La progression du contexte → technique → innovation est bien pensée pour un public radioamateur spatial.

3. **Contenu technique pertinent** : Les points sur l'empoisonnement RaptorQ et le trade-off efficacité/taille sont des éléments différenciants forts.

4. **Ouverture vers la radio cognitive** : C'est un excellent pont vers l'innovation et l'avenir.

5. **Abstract simplifié** : ✅ La version révisée est plus accessible tout en conservant la profondeur technique.

---

## ⚠️ **Points à Améliorer**

### **1. Adéquation avec le public Amsat-F**

**Contexte** : Le public Amsat-F est très hétérogène :
- Des radioamateurs passionnés de satellites mais pas tous experts en protocoles
- Des ingénieurs spatiaux
- Des enthousiastes qui veulent comprendre "comment ça marche"

**Risque** : La section 3 (7 minutes sur les explorations techniques) pourrait perdre une partie de l'audience si elle devient trop théorique.

**Recommandations** : 
- Ajouter des **exemples concrets** : "Pour transmettre une photo de 100 Ko depuis l'ISS avec 20% de perte de paquets..."
- Utiliser des **visuels/graphiques** : Courbes d'efficacité, schémas de l'empoisonnement RaptorQ
- Prévoir une **slide de secours** simplifiée si tu sens que tu perds l'audience

---

### **2. Timing Serré**

**Problème** : 
- Section 3 : 7 minutes pour 3 sous-points techniques complexes = ~2min20 par point
- Section 5 : 5 minutes de Q&R peuvent être insuffisantes si l'audience est engagée

**Plan Révisé Suggéré** :

```markdown
### Plan de la présentation (20mn)

1. Introduction (2mn)
   * Contexte FOSM-1 et besoin de diffusion depuis l'espace
   * Pourquoi un nouveau protocole ?
   * Accroche : "Réinventer la roue"

2. Anatomie d'HQFBP (3mn)
   * En-têtes compacts (CBOR)
   * Flexibilité de la pile d'encodages
   * Asynchronisme total

3. L'Histoire des Explorations (5-6mn)
   * La quête de la pile idéale + Le piège de l'empoisonnement (3mn)
     - Simulations Monte-Carlo
     - Bug RaptorQ : l'importance du CRC PDU-level
   * Efficacité vs Taille : Démonstration par l'exemple (2-3mn)
     - Répétition/RS pour petits fichiers
     - RaptorQ pour gros fichiers

4. Vers la Radio Cognitive (3-4mn)
   * Définition simple : "Une radio qui 'comprend' ce qu'elle reçoit 
     sans configuration préalable"
   * Le concept de "Hailing"
   * Exemple concret : réception automatique d'un satellite inconnu

5. Conclusion et Q&R (6-7mn)
   * État des implémentations (Rust/Python)
   * Retour sur FOSM-1 : "Quand FOSM-1 enverra sa première image..."
   * Échange avec la salle
```

**Justification** : Le public Amsat adore poser des questions - prévoir plus de temps pour les Q&R est stratégique.

---

### **3. Manque de Contexte sur la Radio Cognitive**

**Problème** : Très peu de gens connaissent la radio cognitive. La section 4 risque d'être trop courte (3mn) pour introduire ET expliquer ce concept.

**Recommandations** :

1. **Ajouter une définition simple** en début de section 4 :
   > "La Radio Cognitive, c'est une radio qui 'comprend' ce qu'elle reçoit sans configuration préalable, en s'adaptant dynamiquement aux paramètres annoncés."

2. **Donner un exemple concret** :
   > "Imaginez : votre station reçoit un signal d'un satellite inconnu. Au lieu de devoir configurer manuellement la modulation, le débit, le protocole... votre radio décode l'annonce HQFBP et s'adapte automatiquement !"

3. **Expliquer le lien avec le Hailing** :
   - Le Hailing annonce la pile protocolaire
   - Le récepteur décode cette annonce
   - Auto-configuration de la chaîne de réception

---

### **4. Lien avec FOSM-1 à Renforcer**

**Problème** : FOSM-1 est mentionné mais pas assez exploité comme "fil rouge".

**Recommandations** :

1. **Ouvrir avec une accroche FOSM-1** (Section 1) :
   - "Dans quelques mois, FOSM-1 sera en orbite. Comment va-t-il communiquer avec nous ?"
   - Montrer une photo/schéma du satellite

2. **Revenir sur FOSM-1 en conclusion** (Section 5) :
   - "Quand FOSM-1 enverra sa première image, c'est HQFBP qui la transportera."
   - "Vous pourrez suivre les transmissions et même décoder les fichiers avec nos outils open-source."

3. **Utiliser FOSM-1 comme exemple concret** dans la section technique :
   - "Pour FOSM-1, avec un budget de X kbps et Y% de pertes..."

---

## 📊 **Suggestions de Slides Clés**

Pour maximiser l'impact, prévoir ces visuels :

### Slide 1 : "Le Problème"
- Schéma satellite → Terre avec pertes de paquets visualisées
- Texte : "Pas de retour possible, canal bruité, comment garantir la réception ?"

### Slide 2 : "Empoisonnement RaptorQ"
- Graphique Avant/Après avec CRC
- Visualisation de l'effet d'un paquet corrompu sur le décodeur
- Message clé : "Un seul paquet corrompu peut empoisonner tout le décodage"

### Slide 3 : "Efficacité vs Taille"
- Courbe comparative (Répétition vs RaptorQ)
- Axe X : Taille du fichier (10 bytes → 10 MB)
- Axe Y : Efficacité (% de bande passante utilisée)
- Point de croisement mis en évidence

### Slide 4 : "Radio Cognitive - Le Concept"
- Schéma de la pile protocolaire auto-configurée
- Flux : Réception → Décodage Hailing → Auto-configuration → Décodage données
- Comparaison : Avant (configuration manuelle) vs Après (auto-adaptation)

### Slide 5 : "FOSM-1 en Action"
- Photo du satellite (si disponible)
- Exemple de transmission : "Photo de la Terre, 150 Ko, transmise en 3 passes"
- Lien vers les outils de décodage

### Slide 6 : "Résultats des Simulations"
- Tableau comparatif des encodages testés
- Meilleurs résultats par cas d'usage (petit fichier, gros fichier, canal très bruité)

---

## 🎯 **Verdict Final**

### Le sujet est-il adapté ?

**OUI, absolument !** HQFBP est parfaitement dans le scope d'Amsat-F :
- ✅ Innovation technique pour les satellites
- ✅ Retour d'expérience concret et honnête (bugs, explorations, échecs)
- ✅ Ouverture vers l'avenir (radio cognitive)
- ✅ Lien direct avec une mission réelle (FOSM-1)
- ✅ Aspect pédagogique : "pourquoi faire simple est complexe"

### Faut-il changer quelque chose ?

#### **Changements Essentiels** (Déjà fait ✅)
1. ✅ **Abstract simplifié** : Version plus accessible adoptée
2. 🔲 **Définition de "Radio Cognitive"** : À ajouter dans la section 4
3. 🔲 **Exemples concrets** : À préparer pour la section 3

#### **Changements Recommandés**
4. ⚡ **Ajuster le timing** : Réduire section 3 (5-6mn), augmenter Q&R (6-7mn)
5. ⚡ **Renforcer le fil rouge FOSM-1** : Ouvrir et conclure avec le satellite
6. ⚡ **Préparer des visuels percutants** : Courbes, schémas, animations
7. ⚡ **Version "simplifiée"** : Avoir une explication de secours si la section technique perd l'audience

---

## 💡 **Ton Atout Principal**

L'histoire du **"pourquoi RaptorQ s'empoisonne"** est un **excellent storytelling technique** :
- C'est contre-intuitif (un code de correction d'erreur qui aggrave les erreurs !)
- C'est un vrai bug découvert par l'expérience
- C'est le genre d'anecdote que les gens retiennent et racontent après
- Ça montre l'importance de la validation expérimentale

**Conseil** : Construire cette partie comme une mini-enquête :
1. "On pensait que RaptorQ allait résoudre tous nos problèmes..."
2. "Mais les simulations montraient des résultats catastrophiques..."
3. "Après investigation : un paquet corrompu empoisonne tout le décodeur !"
4. "Solution : un CRC avant RaptorQ pour convertir les erreurs en effacements"

---

## 📝 **Actions Suggérées**

### Court terme (avant la présentation)
- [ ] Ajouter la définition de "Radio Cognitive" dans la section 4
- [ ] Préparer 2-3 exemples concrets avec chiffres (taille fichier, pertes, efficacité)
- [ ] Créer les slides visuelles (courbes, schémas)
- [ ] Répéter en chronométrant chaque section
- [ ] Préparer des photos/schémas de FOSM-1

### Pendant la présentation
- [ ] Surveiller l'engagement de l'audience dans la section 3
- [ ] Avoir une version simplifiée prête si nécessaire
- [ ] Encourager les questions tout au long (pas seulement à la fin)

### Après la présentation
- [ ] Recueillir les questions/feedbacks
- [ ] Partager les slides et les outils de décodage
- [ ] Documenter les points d'intérêt soulevés par l'audience

---

## 🎤 **Message Final**

Cette présentation a tout pour être un **succès** :
- Sujet original et pertinent
- Retour d'expérience authentique
- Ouverture vers l'innovation
- Lien avec une mission concrète

Le principal défi sera de **doser la profondeur technique** pour garder tout le monde à bord tout en satisfaisant les experts. Les exemples concrets et les visuels seront tes meilleurs alliés.

**Bonne chance pour la présentation ! 🚀**
