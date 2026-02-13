# sdk-client-asafe-v.1.0 (PHP)

Petite lib **TOTP (RFC 6238)** compatible **Google Authenticator / Authy**.

## Installation

```bash
composer require sdk-client-asafe-v.1.0
```

## Utilisation

### Exemple basique

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use Asafe2FA\Asafe2FA;

$a2fa = new Asafe2FA();

// Générer une clé secrète (base32)
$secret = $a2fa->generateSecret();

// Créer l’URL OTP Auth pour le scan QR code
$url = $a2fa->getOtpAuthUrl("alice@example.com", "MyCompany", $secret);

// Obtenir le code OTP actuel
$otp = $a2fa->getCurrentOtp($secret);

// Vérifier le code OTP
$ok = $a2fa->verifyKey($secret, $otp, 1);
```

### Comprendre le paramètre `window`

Le paramètre `window` dans `verifyKey()` permet de tolérer un léger décalage horaire. Les codes TOTP changent toutes les 30 secondes (par défaut) ; une différence d’horloge entre le serveur et l’appareil de l’utilisateur peut faire échouer la vérification.

**Fonctionnement :**
- `window = 0` : n’accepte que le code de la période courante
- `window = 1` (par défaut) : accepte les codes de la période courante, précédente et suivante (±30 secondes)
- `window = 2` : accepte les codes sur ±2 périodes (±60 secondes)

**Exemple :**

```php
// Vérification stricte - uniquement le code actuel
$strict = $a2fa->verifyKey($secret, $userInput, 0);

// Par défaut - tolérance ±30 secondes (recommandé)
$normal = $a2fa->verifyKey($secret, $userInput, 1);

// Plus souple - tolérance ±60 secondes
$lenient = $a2fa->verifyKey($secret, $userInput, 2);
```

**Quand utiliser quelle valeur :**
- `window = 0` : sécurité maximale (mais peut échouer en cas de décalage d’horloge)
- `window = 1` : cas général (bon compromis sécurité / utilisabilité)
- `window = 2` ou plus : si les horloges peuvent être mal synchronisées

## API

### `generateSecret(length?: int): string`
Génère une clé secrète aléatoire encodée en base32.
- `length` : optionnel. Nombre de caractères (défaut : 32, ~160 bits)

### `getOtpAuthUrl(account: string, issuer: string, secret: string): string`
Crée une URL `otpauth://` compatible Google Authenticator, Authy et autres apps TOTP.
- `account` : identifiant utilisateur (ex. adresse email)
- `issuer` : nom du service (ex. "MyCompany")
- `secret` : clé secrète générée par `generateSecret()`

### `getCurrentOtp(secret: string, options?: array): string`
Génère le code TOTP actuel pour la clé donnée.
- `secret` : la clé secrète
- `options` : configuration TOTP optionnelle (`period`, `digits`, `algorithm`)

### `verifyKey(secret: string, token: string, window?: int, options?: array): bool`
Vérifie si un token TOTP est valide.
- `secret` : la clé secrète
- `token` : le code OTP à vérifier (saisie utilisateur)
- `window` : optionnel. Tolérance temporelle en périodes (défaut : 1). Voir [Comprendre le paramètre `window`](#comprendre-le-paramètre-window) ci-dessus
- `options` : configuration TOTP optionnelle
- Retourne : `true` si le token est valide, `false` sinon

## Licence

MIT
