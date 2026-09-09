# CiK V1 — App Privacy / QCM App Store Connect

Base : build CiK V1 figé au J+4-A + documentation officielle RevenueCat consultée le 9 septembre 2026.

## Réponse générale
- L’app / ses partenaires tiers collectent-ils des données ? **Oui**
- Les données financières saisies dans CiK et traitées uniquement sur l’iPhone : **ne pas déclarer comme “collectées”** au sens App Privacy Apple, puisqu’elles ne quittent pas l’appareil.

## Type à déclarer

### Purchases → Purchase History
- Collecté : **Oui**
- Finalités :
  - **App Functionality**
  - **Analytics**
- Lié à l’identité : **Non**, dans la configuration actuelle avec App User ID anonyme RevenueCat et sans compte CiK / custom user ID.
- Utilisé pour tracking : **Non**

## Types à ne pas sélectionner dans la configuration actuelle
Sous réserve que le build ne change pas avant soumission :
- Contact Info
- Health & Fitness
- Financial Info
- Location
- Sensitive Info
- Contacts
- User Content
- Browsing History
- Search History
- Identifiers
- Usage Data
- Diagnostics
- Other Data

## À revalider juste avant soumission
- J+4-D Face ID ne doit pas modifier cette liste de collecte : la biométrie doit rester gérée localement par iOS.
- J+5 doit vérifier le privacy manifest agrégé du build natif.
- Si PostHog, Supabase, Sentry ou tout autre SDK réseau est ajouté avant soumission, ce QCM devient obsolète et doit être recalculé.
