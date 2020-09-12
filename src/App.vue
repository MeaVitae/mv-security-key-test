<template>
  <div id="app">
    TEST KEY IMPLEMENTATION
  </div>
</template>

<script>

export default {
  name: 'App',

  created () {
    this.yubikey()
  },

  methods: {
    async yubikey () {
      if (!window.PublicKeyCredential) {
        return alert(`Your web browser does not meet our security requirements.  Please
        upgrade to Google Chrome or contact MeaVitae support for help.`)
      }

      console.log('------BEGIN CREATE CREDENTIALS------')

      const decryptKeyIntoUserIdOnCreate = 'STRING TO DECRYPT KEY WRAPPER'

      const createCredential = await navigator.credentials.create({
        publicKey: {
          challenge: window.crypto.getRandomValues(new Uint8Array(10)),

          rp: {
            name: 'dan'
          },

          user: {
            id: Uint8Array.from(decryptKeyIntoUserIdOnCreate, c => c.charCodeAt(0)),
            name: 'jim',
            displayName: 'jim display'
          },

          pubKeyCredParams: [
            { type: 'public-key', alg: -7 },
            { type: 'public-key', alg: -257 }
          ],

          authenticatorSelection: {
            userVerification: 'required',
            requireResidentKey: true,
            authenticatorAttachment: 'cross-platform'
          },

          timeout: 120000, // 2 minutes

          excludeCredentials: [],

          extensions: {
            loc: true
          }
        }
      })

      console.log('id as base64', createCredential.id)

      const rawIdAsBase64 = btoa(String.fromCharCode.apply(null, new Uint8Array(createCredential.rawId)))

      console.log('rawId as base64', rawIdAsBase64)

      const { authData } = window.CBOR.decode(createCredential.response.attestationObject)

      const dataView = new DataView(new ArrayBuffer(2))
      const idLenBytes = authData.slice(53, 55)
      idLenBytes.forEach((value, index) => dataView.setUint8(index, value))
      const credentialIdLength = dataView.getUint16()
      const credentialId = authData.slice(55, 55 + credentialIdLength)

      console.log('credentialId: ', credentialId)

      const credentialIdAsBase64 = btoa(String.fromCharCode.apply(null, credentialId))

      console.log('credentialIdAsBase64: ', credentialIdAsBase64)

      console.log('IS rawId===credentialId', !!(rawIdAsBase64===credentialIdAsBase64))

      const publicKeyBytes = authData.slice(55 + credentialIdLength)
      console.log('publicKeyBytes: ', publicKeyBytes)

      // authenticatorSelection.requireResidentKey = true makes CBOR.decode error
      // const publicKeyObject = window.CBOR.decode(publicKeyBytes.buffer)
      // console.log('publicKeyObject: ', publicKeyObject)

      console.log('------END CREATE CREDENTIALS------')

      const challenge = window.crypto.getRandomValues(new Uint8Array(10))
      const credentialIdAsArrayBuffer = Uint8Array.from(atob(rawIdAsBase64), c => c.charCodeAt(0))

      const credential = {
        publicKey: {
          challenge,
          timeout: 120000, // 2 minutes
          allowCredentials: [{
            type: 'public-key',
            id: credentialIdAsArrayBuffer
          }]
        }
      }

      console.log('------BEGIN GET CREDENTIAL 1ST TIME------')

      const getCredentialOne = await navigator.credentials.get(credential)
      console.log('One response:', getCredentialOne.response)
      const getCredentialOneSignatureAsBase64 = btoa(String.fromCharCode.apply(null, new Uint8Array(getCredentialOne.response.signature)))
      console.log('One Signature As Base64: ', getCredentialOneSignatureAsBase64)
      const getCredentialOneUserHandleAsString = String.fromCharCode.apply(null, new Uint8Array(getCredentialOne.response.userHandle))
      console.log('One User Handle As String: ', getCredentialOneUserHandleAsString)
      const getCredentialOneAuthenticatorDataAsBase64 = btoa(String.fromCharCode.apply(null, new Uint8Array(getCredentialOne.response.authenticatorData)))
      console.log('One Authenticator Data As Base64: ', getCredentialOneAuthenticatorDataAsBase64)
      console.log('One Authenticator Data: ', getCredentialOne.response.authenticatorData)

      console.log('------BEGIN GET CREDENTIAL 2ND TIME------')

      const getCredentialTwo = await navigator.credentials.get(credential)
      console.log('Two response:', getCredentialTwo.response)
      const getCredentialTwoSignatureAsBase64 = btoa(String.fromCharCode.apply(null, new Uint8Array(getCredentialTwo.response.signature)))
      console.log('Two Signature As Base64: ', getCredentialTwoSignatureAsBase64)
      const getCredentialTwoUserHandleAsString = String.fromCharCode.apply(null, new Uint8Array(getCredentialTwo.response.userHandle))
      console.log('Two User Handle As String: ', getCredentialTwoUserHandleAsString)
      const getCredentialTwoAuthenticatorDataAsBase64 = btoa(String.fromCharCode.apply(null, new Uint8Array(getCredentialTwo.response.authenticatorData)))
      console.log('Two Authenticator Data As Base64: ', getCredentialTwoAuthenticatorDataAsBase64)
      console.log('Two Authenticator Data: ', getCredentialTwo.response.authenticatorData)

      console.log(
        'Is .get() userHandle String the same every time as put into user.id on create(): ',
        getCredentialOneUserHandleAsString===getCredentialTwoUserHandleAsString && decryptKeyIntoUserIdOnCreate===getCredentialOneUserHandleAsString
      )
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
