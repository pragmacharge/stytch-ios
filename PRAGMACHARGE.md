# PragmaCharge fork of stytch-ios

Upstream: https://github.com/stytchauth/stytch-ios, tag `0.105.0`.
Branch `pcce/0.105.0`, tagged `0.105.0-pcce.1`, is what the PragmaCharge driver app pins.

## The one patch

`StytchB2BClient.passwords.discovery.resetByEmail` refuses to run without a PKCE pair stored on the
device and throws `StytchSDKError.missingPKCE` before any request is made. A set-password link the
backend starts with `passwords.discovery.email.resetStart` (driver invitations, admin resets) has no
PKCE challenge, so the device never has a pair and the link can never be completed on iOS.

stytch-android sends the same call without a verifier, and Stytch accepts it. The patch does the
same: with no stored pair, the request goes out without `pkce_code_verifier`. Resets started in the
app keep their pair and their verifier, and Stytch still enforces the challenge for those on the
server, so nothing that PKCE protects is weakened.

File: `Sources/StytchCore/StytchB2BClient/StytchB2BClient+Passwords/StytchB2BClient+Passwords+Discovery.swift`

## Moving to a new upstream release

    git fetch upstream --tags
    git switch -c pcce/<version> <version>
    git cherry-pick <the patch commit on the previous pcce branch>
    git tag <version>-pcce.1 && git push origin pcce/<version> <version>-pcce.1

Then point the app's package reference at the new tag. Drop the fork once upstream sends the reset
without a verifier, as stytch-android does.
