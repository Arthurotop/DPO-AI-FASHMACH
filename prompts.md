```yaml
Agent_Assistant:
  description: "Agent chargé de gérer l’inscription, le paiement et la transmission sécurisée des données du client."
  
  entrees:
    - donnees_client_completes:
        description: "Ensemble des informations fournies par le client lors de l'inscription, de la commande, du paiement et de l'historique de discusion"
        provenance: "Formulaire client + validation de recommandation envoyée par l’Agent Artisan."
        champs:
          identite:
            - nom:
                type: string
                exemple: "Dupont"
                usage: "Identification et génération d’étiquette de livraison."
            - prenom:
                type: string
                exemple: "Camille"
                usage: "Identification complémentaire du client."
            - genre:
                type: enum
                valeurs_possibles: ["Homme", "Femme", "Non spécifié"]
                usage: "Adaptation du style et du modèle recommandé."
            - date_naissance:
                type: date
                format: "YYYY-MM-DD"
                usage: "Estimation de l’âge, vérification de majorité légale."
          contact:
            - email:
                type: string
                format: "exemple@domaine.com"
                usage: "Communication transactionnelle et confirmation de commande."
            - telephone:
                type: string
                format: "+33XXXXXXXXX"
                usage: "Contact pour la livraison ou support client."
          adresse_livraison:
            - rue:
                type: string
                exemple: "12 rue des Lilas"
                usage: "Adresse postale de livraison."
            - complement:
                type: string
                nullable: true
                usage: "Complément d'adresse (bâtiment, étage, etc.)."
            - code_postal:
                type: string
                exemple: "75011"
                usage: "Code postal pour expédition."
            - ville:
                type: string
                exemple: "Paris"
                usage: "Ville de livraison."
            - pays:
                type: string
                exemple: "France"
                usage: "Pays pour transporteur."
          informations_paiement:
            - type_carte:
                type: enum
                valeurs_possibles: ["Visa", "MasterCard", "Amex"]
                usage: "Identification du mode de paiement."
            - dernieres_chiffres:
                type: string
                exemple: "XXXX-XXXX-XXXX-1234"
                usage: "Conservation partielle pour justificatif."
            - transaction_id:
                type: string
                usage: "Identifiant de transaction pour suivi et traçabilité."
            - montant_total:
                type: float
                unite: "EUR"
                usage: "Montant total payé par le client."
          donnees_physiques:
            - taille_cm:
                type: float
                unite: "cm"
                usage: "Paramètre de calcul d’IMC et recommandation de tailles."
            - poids_kg:
                type: float
                unite: "kg"
                usage: "Paramètre de calcul d’IMC et ajustement morphologique."
            - imc:
                type: float
                formule: "poids_kg / (taille_cm/100)^2"
                usage: "Indicateur morphologique (sera recalculé/anonymisé plus tard)."
            - morphologie:
                type: enum
                valeurs_possibles: ["A", "V", "X", "O", "H"]
                usage: "Aide à la personnalisation stylistique."
          preferences:
            - style_prefere:
                type: list
                exemple: ["Casual", "Chic", "Streetwear"]
                usage: "Filtrage des produits recommandés."
            - palette_couleurs:
                type: list
                exemple: ["Bleu marine", "Beige", "Blanc"]
                usage: "Recommandation de teintes adaptées."
            - budget_max:
                type: float
                unite: "EUR"
                usage: "Filtre budgétaire appliqué aux recommandations."
          historique_commandes:
            - id_commande:
                type: string
                usage: "Identifiant unique de la commande."
            - produits_achetes:
                type: list
                champs:
                  - id_produit: string
                  - nom_produit: string
                  - prix: float
                  - date_achat: date
                usage: "Historique d’achat utile pour la recommandation future."
          images_profil:
            - photo_profil:
                type: image
                format: "jpeg/png"
                usage: "Analyse morphologique (sera anonymisée ensuite)."
            - photo_silhouette:
                type: image
                format: "jpeg/png"
                usage: "Utilisée pour la détection de forme corporelle (crop/flou ensuite)."

  traitement:
    - vérifie_et_valide_les_donnees_obligatoires
    - stocke_les_donnees_necessaires_a_la_livraison
    - chiffre_les_donnees_sensibles_temporairement
    - transmet_les_donnees_a_l_agent_d_anonymisation_pour_traitement_RGPD

  sorties:
    - donnees_pour_livraison:
        description: "Sous-ensemble des données nécessaires à l’expédition."
        champs:
          - nom
          - prenom
          - adresse_complète
          - email
          - telephone
          - id_commande
          - montant_total
    - donnees_a_anonymiser:
        description: "Sous-ensemble transmis à l’Agent d’Anonymisation pour suppression ou pseudonymisation."
        champs:
          - nom
          - prenom
          - email
          - telephone
          - adresse
          - images_profil
          - donnees_physiques (taille, poids, morphologie, imc)
          - historique_commandes
          - preferences (style, couleurs)

```

```yaml
Agent_Anonymisation:
  description: "Agent en charge de la suppression ou transformation des données identifiantes."
  entrees:
    - donnees_a_anonymiser
  traitement:
    - détecte_les_champs_personnels
    - pseudonymise_nom_prenom_email
    - floute_ou_coupe_les_images
    - calcule_IMC_a_partir_de_la_taille_et_du_poids
    - crée_un_identifiant_anonyme_unique
  sorties:
    - donnees_anonymisees:
        champs:
          - id_client: string
          - genre: string
          - imc: float
          - morphologie: string
          - style_prefere: list
          - historique_achat: list(id_produit)
          - images_anonymisees: list(url)

```

```yaml
Agent_FashiMatch:
  description: "IA de recommandation personnalisée utilisant un LLM et les données anonymisées client."
  entrees:
    - base_produits:
        description: "Catalogue complet des produits des artisans."
        champs:
          - id_produit: string
          - nom_produit: string
          - type: string
          - genre: [Homme, Femme, Enfant, Mixte]
          - taille_disponible: [XS, S, M, L, XL]
          - couleurs: list
          - matiere: string
          - style: string
          - prix: float
          - images_produit: list(url)
          - disponibilité: boolean
    - donnees_client_anonymisees:
        description: "Profil client anonymisé issu de la base client."
        champs:
          - id_client: string
          - genre: [Homme, Femme]
          - morphologie: string
          - imc: float
          - tailles_vetements: dict(top: string, bottom: string, shoes: string)
          - preferences_style: list (ex: casual, chic, bohème)
          - palette_couleur_preferee: list
          - historique_commandes: list(id_produit)
          - comportement_achat: dict(frequence: int, budget_moyen: float)
  traitement:
    - utilise_un_LLM_pour_associer_le_profil_aux_produits
    - génère_une_recommandation_personnalisée
    - vérifie_la_disponibilité_du_produit
  sorties:
    - details_produit_ideal:
        champs:
          - id_produit
          - nom_produit
          - couleur
          - taille
          - prix
          - url_image
          - justification_recommandation
    - produit_inexistant:
        description: "Signal envoyé si aucun produit adéquat n'est trouvé."
        champs:
          - id_recherche
          - critères_recherchés
          - taux_similarité

```

```yaml
Agent_Artisan:
  description: "Interface entre la recommandation et les fournisseurs/artisans pour valider et expédier les produits."
  entrees:
    - details_produit_ideal:
        description: "Produit recommandé par FashiMatch."
        champs:
          - id_produit
          - couleur
          - taille
          - prix
          - id_client
    - base_artisans:
        description: "Liste des artisans et de leurs stocks disponibles."
        champs:
          - id_artisan
          - nom_artisan
          - stock_produits: list(id_produit)
  traitement:
    - vérifie_disponibilité_du_produit
    - si_indisponible_demande_nouvelle_recommandation
    - propose_recommandation_au_client
    - envoie_commande_aux_fournisseurs
  sorties:
    - proposition_client:
        champs:
          - id_recommandation
          - id_client
          - produit: id_produit
          - prix
          - delai_livraison
    - commande_fournisseur:
        champs:
          - id_commande
          - id_fournisseur
          - produit
          - quantite
          - adresse_livraison

```

```yaml
Client:
  description: "Utilisateur final recevant les recommandations et passant commande."
  entrees:
    - recommandation_produits
    - formulaire_inscription_paiement
  traitement:
    - accepte_ou_refuse_la_recommandation
    - remplit_les_donnees_personnelles_et_paiement
  sorties:
    - donnees_client_completes:
        champs:
          - nom: string
          - prenom: string
          - email: string
          - telephone: string
          - adresse:
              rue: string
              code_postal: string
              ville: string
              pays: string
          - informations_paiement:
              type_carte: string
              dernieres_chiffres: string
              transaction_id: string
          - donnees_physiques:
              taille_cm: float
              poids_kg: float
              imc: float
              morphologie: string
              genre: string
              age: int
          - preferences:
              style: list
              couleurs: list
              budget_max: float
```

```yaml
Base_Client_Anonymisee:
  description: "Stock central des données anonymisées pour apprentissage IA."
  entrees:
    - donnees_anonymisees
  traitement:
    - stocke_les_donnees_en_respectant_RGPD
    - indexe_les_caractéristiques_utiles_pour_le_modele
  sorties:
    - donnees_accessibles_pour_IA:
        champs:
          - id_client
          - style
          - imc
          - morphologie
          - historique_achat
          - préférences_couleurs

```

```yaml
Programme_Donnees_Evolutives:
  description: "Moteur d’amélioration continue de l’IA."
  entrees:
    - donnees_anonymisees_mises_a_jour
  traitement:
    - identifie_les_tendances_et_patterns
    - met_a_jour_les_modeles_IA
  sorties:
    - modele_IA_mis_a_jour

```

```yaml
Agent_Match:
  description: "Vérifie si les produits recommandés existent réellement et les ajoute si besoin."
  entrees:
    - produit_recommande_non_trouve
  traitement:
    - recherche_produit_similaire_dans_catalogue
    - si_similarite_supérieure_a_70_pourcent_=>_produit_reel
    - sinon_transmet_a_agent_concepteur
  sorties:
    - produit_reel_ajoute_au_catalogue
    - signalement_a_agent_concepteur

```

```yaml
Agent_Concepteur:
  description: "Crée de nouveaux produits à partir des tendances non couvertes."
  entrees:
    - resume_des_produits_inexistants
  traitement:
    - génère_nouveaux_modeles_de_produits
    - valide_avec_artisans
    - met_a_jour_le_catalogue
  sorties:
    - nouveaux_produits_catalogue

```

```yaml
Base_Artisans:
  description: "Référentiel central des artisans et de leurs stocks."
  entrees:
    - informations_stocks
    - produits_catalogue
  traitement:
    - rend_les_stocks_visibles_aux_agents
    - met_a_jour_la_disponibilite_en_temps_reel
  sorties:
    - catalogue_disponible_pour_FashiMatch_et_Artisan

```