# CiK V1 — App Privacy / questionnaire App Store Connect

Base : build candidat V1 (Bloc 4, 18 septembre 2026), `docs/CIK_V1_DATA_INVENTORY.md`, documentation officielle consultée le 18 septembre 2026 :
- Apple — App privacy details : https://developer.apple.com/app-store/app-privacy-details/
- RevenueCat — Apple App Privacy : https://www.revenuecat.com/docs/platform-resources/apple-platform-resources/apple-app-privacy
- PostHog — Privacy : https://posthog.com/docs/privacy

Recalculé le 20 septembre 2026 contre l'état de `main` : code réel de PostHog (`store/analytics.ts`, `store/analyticsContract.ts`), RevenueCat (`store/purchases.ts`), identité pseudonyme (`store/identity.ts`), consentement (`store/analyticsConsent.ts`, `components/cik/AnalyticsTracker.tsx`) et couche Supabase (`supabase/functions`, `supabase/migrations`). Les trois réponses ci-dessous sont inchangées par ce recalcul.

## Réponse générale
- Vous ou vos partenaires collectez-vous des données depuis cette app ? **Oui**
- Tracking (au sens Apple) : **Non** — la définition Apple couvre le rapprochement de données de l'app avec des données tierces à des fins publicitaires, ou le partage avec un data broker. Aucun des deux ne se produit : RevenueCat traite l'abonnement pour son propre compte, PostHog ne reçoit rien sans consentement et ne partage rien à des fins publicitaires, Supabase ne reçoit qu'une copie interne.
- La mesure d'usage (PostHog) n'est active qu'après consentement explicite dans l'app ; Apple demande de déclarer aussi les données collectées sur consentement.

## Types déclarés

### Identifiers → User ID
- Collecté : Oui
- Finalités : App Functionality ; Analytics
- Linked to User : **Yes**
- Tracking : **No**
- Justification : identifiant pseudonyme CiK (UUID v4, Keychain), App User ID RevenueCat et distinct_id PostHog, clé de jointure Supabase. Aucune identité civile. L'identifiant est affiché à l'utilisateur (« Identifiant d'assistance », Réglages) et sert à traiter une demande sur ses propres données pseudonymes : il identifie un utilisateur particulier au sens Apple, même sans nom ni e-mail.
- Citation Apple (linked) : « Data collected from an app is often linked to the user's identity, unless specific privacy protections are put in place before collection to de-identify or anonymize it. » — aucune de ces protections n'est en place : l'identifiant est stable, réutilisé entre RevenueCat, Supabase et PostHog, et communiqué à l'utilisateur lui-même.

### Usage Data → Product Interaction
- Collecté : Oui, uniquement sur consentement explicite (retiré à tout moment dans Réglages)
- Finalités : Analytics
- Linked to User : **Yes**
- Tracking : **No**
- Justification : les 8 events du contrat fermé (`store/analyticsContract.ts`) — progression onboarding, affichage et action du paywall, étapes du wizard, calcul du Flex (sans montant), ouverture de l'app — envoyés à PostHog liés à l'identifiant pseudonyme.

### Purchases → Purchase History
- Collecté : Oui
- Finalités : App Functionality ; Analytics
- Linked to User : **Yes**
- Tracking : **No**
- Justification : App Functionality — abonnement vérifié et restauré via RevenueCat. Analytics — suivi commercial interne (copie minimale des événements d'abonnement vers Supabase, `commercial_events` / `analytics_users`), pour tous les abonnés, indépendamment du consentement analytics (décision D6). Aucun montant ni donnée de paiement brute.
- Citation RevenueCat (finalités) : « Select both "Analytics" and "App Functionality" » pour Purchase History — confirmé par la documentation officielle RevenueCat elle-même.
- Lié : la documentation RevenueCat indique que seul un App User ID anonyme, jamais rattaché à un e-mail ou une autre coordonnée, autoriserait « Non ». Ce n'est pas le cas ici : le même identifiant est affiché à l'utilisateur comme identifiant d'assistance et sert à traiter ses demandes — il reste donc « Lié » au sens Apple, sans jamais être une identité civile.

## Types non déclarés
- Financial Info — jamais transmis ; les montants et soldes restent sur l'appareil et dans la sauvegarde iCloud privée de l'utilisateur.
- Location (Precise / Coarse) — GeoIP désactivé côté SDK PostHog et côté projet, adresse IP non conservée (Task 8) ; aucune autre source de localisation.
- Identifiers → Device ID — non déclaré : l'audit du SDK (`posthog-react-native` / `@posthog/core`) montre que `$device_id` n'est jamais renseigné (aucun appel ne fixe la propriété persistée `DeviceId`) ; la clé part donc `undefined` et disparaît de la charge JSON envoyée. Aucun identifiant d'installation persistant n'est donc réellement transmis par CiK.
- Diagnostics (Other Diagnostic Data) — aucun SDK de crash ou de performance ; PostHog ne capture ni logs de console, ni performance, ni erreurs, ni session replay.
- Contact Info, Health & Fitness, Sensitive Info, Contacts, User Content, Browsing/Search History, Other Data — aucune collecte, aucun SDK concerné.
- Usage Data → Advertising Data / Other Usage Data — aucune collecte : la liste des events est fermée (`store/analyticsContract.ts`), aucune capture automatique n'est activée.
- **Aucune donnée, déclarée ou non, n'est utilisée pour du Tracking au sens Apple.** Aucun identifiant publicitaire, aucun App Tracking Transparency, aucun SDK publicitaire, aucun partage avec un data broker, aucun rapprochement avec des données tierces. `NSPrivacyTracking` vaut `false` dans le manifeste applicatif (`app.json`).

## Points vérifiés
- Données financières jamais transmises ; sauvegarde iCloud dans le container privé de l'utilisateur, inaccessible à CiK depuis un serveur.
- Face ID : authentification entièrement gérée par iOS ; CiK ne reçoit qu'un succès ou un échec.
- IP non conservée par PostHog, GeoIP désactivé (SDK et projet) ; rétention PostHog : 12 mois (plan Free du projet CiK, confirmé le 18 septembre 2026 — aucun passage à un plan payant connu à cette date).
- Métadonnées techniques ajoutées automatiquement à chaque event par `posthog-react-native` / `@posthog/core` : `$device_type`, `$app_build`, `$app_name`, `$app_namespace`, `$app_version`, `$device_manufacturer`, `$device_name` (nom générique du modèle depuis iOS 16, jamais le nom personnalisé de l'appareil), `$os_name`, `$os_version`, `$is_emulator`, `$locale`, `$timezone`, `$session_id` (identifiant de session tournant), `$lib`, `$lib_version` ; et, une seule fois, sur l'event technique `$identify` : `$anon_distinct_id` (identifiant anonyme interne généré par le SDK, distinct de l'identifiant pseudonyme CiK). Aucune de ces valeurs ne porte de donnée financière ou de texte libre.

## Vérifications finales avant publication

Ces trois points ne sont pas vérifiables depuis le dépôt : ils dépendent d'une configuration de
tableau de bord ou du binaire réellement construit. Ils doivent être confirmés avant de figer les
réponses ci-dessus dans App Store Connect.

1. **RevenueCat — aucune intégration publicitaire ou d'attribution activée.** Vérifier dans le
   tableau de bord RevenueCat qu'aucune intégration d'attribution ou de réseau publicitaire n'est
   branchée (AppsFlyer, Adjust, Branch, Singular, Meta, Google Ads, Apple Search Ads). C'est le seul
   chemin qui ferait basculer la réponse Tracking à **Yes** et apparaître un **Device ID** : le code
   applicatif, lui, n'appelle ni `collectDeviceIdentifiers`, ni `setAttributes`, ni `setEmail`, ni
   `setDisplayName`, ni `setPhoneNumber`, ni `logIn`.

2. **PostHog — GeoIP / IP et fonctions non utilisées conformes à la configuration retenue.** Le SDK
   coupe déjà côté client (`disableGeoip: true`, `captureAppLifecycleEvents: false`,
   `enableSessionReplay: false`, `preloadFeatureFlags: false`, `disableRemoteFeatureFlags: true`,
   `disableSurveys: true`, host `https://eu.i.posthog.com`). Confirmer côté **projet** PostHog :
   adresse IP non conservée, enrichissement GeoIP désactivé, session replay / surveys / error
   tracking inactifs, rétention 12 mois. Une GeoIP active côté projet rendrait **Coarse Location**
   déclarable.

3. **Manifeste RevenueCat et privacy report agrégé vérifiés sur le build candidat.** Ce dépôt ne pin
   que `react-native-purchases` 10.9.0 → `PurchasesHybridCommon` 18.33.1 ; il ne contient ni `ios/`
   ni `Podfile.lock`, donc la version RevenueCat effective est résolue au build EAS. Sur le build
   candidat : relire le `PrivacyInfo.xcprivacy` du pod RevenueCat (attendu : `PurchaseHistory` seul —
   toute autre entrée, notamment un identifiant d'appareil, ajouterait une catégorie), puis générer
   le privacy report agrégé depuis Xcode et le comparer aux trois types déclarés ici.

## À revalider en continu
- Tout ajout de SDK réseau, d'event ou de propriété rend ce questionnaire obsolète.
- Build candidat : aucun e-mail ITMS-91053 à l'upload.
