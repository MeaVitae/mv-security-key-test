# Security Key Authenticator Test

Code in src/App.vue


## Issue

Solo Key version 1 behaves differently to Solo Key version 2 and Yubi Key

Scenario:

* Store 2 credentials on the same key with different userId/userHandle using WebAuthn navigator.credentials.create() credential type PublicKey

* Use WebAuthn navigator.credentials.get() passing in an array of the two returned credential ids

* Expected behaviour: security authenticator key to authenticate and returned the 1st matched credential stored on the key. Solo Key V2 and Yubi Key do this.

* Actual behaviour: Solo Key v1 throws an error:
```
DOMException: The operation either timed out or was not allowed. See: https://www.w3.org/TR/webauthn-2/#sctn-privacy-considerations-client.
```

