# Abstract et Plan : HQFBP aux Rencontres Spatiales Radioamateur de l'Amsat-F

**Titre :** Hamradio Quick File Broadcasting Protocol (HQFBP)
**Sous-titre :** Comment réinventer la roue ...mais en se couchant moins bête !

## Abstract

Comment transmettre de manière fiable des fichiers depuis un satellite quand le canal radio est bruité et qu'aucun retour n'est possible ? Le projet HQFBP, développé pour la mission FOSM-1, propose une solution de diffusion asynchrone optimisée pour les liens satellites.

Cette présentation retrace l'aventure technique : des simulations Monte-Carlo pour trouver la meilleure combinaison d'encodages, la découverte d'un bug subtil dans RaptorQ (les "Fountain Codes"), et comment l'efficacité varie selon la taille des fichiers (un SMS vs une photo HD).

Au-delà du transfert de fichiers, HQFBP ouvre la voie à la "Radio Cognitive" : des récepteurs qui s'adaptent automatiquement au protocole annoncé par l'émetteur. Une histoire de bits, de bruit, et de plaisir à comprendre pourquoi "faire simple" est complexe !

---

## Plan de la présentation (20mn)

### 1. Introduction (2mn)
* Le contexte : Mission FOSM-1 et le besoin de diffuser des données depuis l'espace.
* Pourquoi un nouveau protocole ? (Limitations de l'existant : overhead, mode connecté).
* Le titre : Pourquoi "réinventer la roue" ?

### 2. Anatomie d'HQFBP (3mn)
* En-têtes compacts : L'apport de CBOR (RFC 8949).
* Flexibilité : La pile d'encodages (`Content-Encoding`).
* Asynchronisme total : Pas de handshake, juste de la diffusion.

### 3. L'Histoire des Explorations (7mn)
* **La quête de la pile idéale** : Simulations de Monte-Carlo et exploration des différents empilements.
* **Le piège de l'empoisonnement** : 
    * Retour d'expérience sur RaptorQ (Fountain Codes).
    * L'importance cruciale du CRC PDU-level (conversion d'erreur en effacement).
* **Efficacité vs Taille** : 
    * Pourquoi la répétition/RS gagne sur les petits fichiers.
    * Pourquoi RaptorQ devient imbattable quand le fichier grossit.

### 4. Vers la Radio Cognitive : Le protocole d'annonce (3mn)
* Le concept de "Hailing" : Annoncer la pile protocolaire pour être compris.
* Décomposition explicite de la pile (Modulation, Codec, Protocole).
* Vers des radios qui "comprennent" ce qu'elles entendent sans configuration préalable.

### 5. Conclusion et Questions/Réponses (5mn)
* État actuel des implémentations (Rust/Python).
* Prochaines étapes sur FOSM-1.
* Échange avec la salle.
