# Projet de Réconciliation et d'Audit du Référentiel TVA — Meridian Distribution

Ce dépôt consigne l'analyse exploratoire, le nettoyage structurel, l'automatisation des appels à l'API VIES et les recommandations stratégiques pour la validation d'un référentiel de $10\,000$ numéros de TVA, dans le cadre de la sécurisation des flux de facturation intracommunautaire (art. 262 ter I du CGI).

## 📂 Structure du Dépôt

```text
.
├── notebooks/
│   ├── 01_exploration.ipynb       # Phase 1 : Exploration, bruit, doublons et analyse des canaux
│   └── 02_analyse.ipynb           # Phases 2, 3 & 4 : Validation structurelle, entonnoir, API VIES et réconciliation
├── starter-kit/
│   ├── data/                      # Jeux de données bruts (numeros_tva.csv & numeros_tva.xlsx)
│   └── validation_structure.py    # Module de validation structurelle fourni
├── vies_collecte.log              # Journal d'exécution de la campagne API VIES (module logging)
├── resultats_vies.csv             # Résultats exportés des appels API
└── README.md                      # Documentation du projet
```

## 🚀 Méthodologie et Synthèse des Phases

### Phase 1 : Exploration et Analyse Initiale (`01_exploration.ipynb`)
* **Comparaison des sources :** Vérification de l'identité parfaite entre les formats CSV et XLSX (le format CSV a été retenu comme source de vérité).
* **Anomalies et Pépites :**
  * **Bruit :** $1\,335$ lignes ($13.35\ \%$) contenant des caractères parasites (espaces, points, tirets).
  * **Valeurs vides :** $201$ valeurs manquantes au total ($146$ NaNs, $55$ tirets, $59$ espaces/vides).
  * **Diversité géographique :** $15$ pays déclarés, dont $5$ hors périmètre ou anomalies (GB, UK, ZZ, QQ, XX).
  * **Divergences CRM :** Mise en évidence de $647$ décalages entre le pays déclaré dans le CRM (`pays_declare`) et le préfixe réel du numéro de TVA.

### Phase 2 : Nettoyage, Filtrage et Entonnoir Chiffré (`02_analyse.ipynb`)
Intégration du module `validation_structure.py` pour purifier le portefeuille :
1. **Volume initial brut :** $10\,000$ lignes
2. **Moins les valeurs vides :** $-260$
3. **Moins les clés mathématiques fausses (`cle_invalide`) :** $-2\,605$ (éliminées sans appel API)
4. **Moins les doublons stricts normalisés :** $-438$ (définis par une normalisation en majuscules sans espaces, points ni tirets)
5. **Volume cible éligible (`ok`) :** $\mathbf{6\,083 \text{ numéros}}$ structurellement valides prêts pour l'API VIES.

### Phase 3 : Interrogation de l'API VIES et Journalisation
* **Tests manuels :** Analyse approfondie des réponses de l'API au-delà du booléen (`isValid`, `name`, `address`) et identification des limites de concurrence (`MS_MAX_CONCURRENT_REQ`).
* **Script de collecte robuste :** Implémentation d'une boucle avec le module **`logging`** de Python et une temporisation de sécurité (`time.sleep`) pour éviter la saturation des serveurs européens.
* **Chiffrage de la campagne :** Estimation de la durée totale pour balayer le référentiel éligible :
  $$\text{6 083 numéros} \times 3,5 \text{ secondes} \approx 21\,290 \text{ secondes} \approx \mathbf{5,9 \text{ hours}}$$
* **Validité juridique :** Une réponse de l'API n'étant valide qu'à l'instant $T$, une campagne de ré-attestation trimestrielle est indispensable.

### Phase 4 : Analyse des Divergences CRM et Recommandations
* **Analyse par source de saisie :** Preuve d'un taux d'erreur homogène (entre $14\ \%$ et $16\ \%$) sur l'ensemble des canaux (`crm`, `import_fournisseur`, `portail_client`, `reprise_erp`, `saisie_manuelle`), mettant en évidence un défaut global de contrôle à l'entrée.
* **Règle des non-tranchés :** Aucun flux ne doit être exonéré de TVA sans validation manuelle comptable et justificatif officiel.

## 🛠️ Guide d'Exécution

1. **Prérequis :** Assurez-vous d'avoir Python $3$ et installez les dépendances :
   ```bash
   pip install pandas openpyxl requests jupyter
   ```
2. **Lancement :** Ouvrez le dossier dans **VS Code** et exécutez les notebooks dans l'ordre :
   * `notebooks/01_exploration.ipynb` pour l'exploration et les anomalies.
   * `notebooks/02_analyse.ipynb` pour le nettoyage, l'entonnoir, les tests API et la réconciliation.