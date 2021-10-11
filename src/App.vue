<template>
  <div id="app">
    TEST KEY IMPLEMENTATION
  </div>
</template>

<script>
export default {
  name: 'App',

  async mounted () {
    await this.runKeyTest()
  },

  methods: {
    async createPublicKeyCredential (userId = '') {
      return navigator.credentials.create({
        publicKey: {
          challenge: window.crypto.getRandomValues(new Uint8Array(10)),

          rp: {
            name: 'Test'
          },

          user: {
            id: Uint8Array.from(userId, c => c.charCodeAt(0)),
            name: 'Test Person',
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
    },

    async runKeyTest () {
      if (!window.PublicKeyCredential) {
        return alert(`Your web browser does not meet our security requirements.  Please
        upgrade to Google Chrome or contact MeaVitae support for help.`)
      }

      try {
        console.log('------BEGIN CREATE CREDENTIALS 1------')

        const firstUserId = `STATIC:${window.crypto.getRandomValues(new Uint32Array(1))[0].toString(36).substring(0, 4)}`

        console.log('First userHandle:', firstUserId)

        const firstCreatedCredential = await this.createPublicKeyCredential(firstUserId)

        const firstCredentialIdAsBase64 = btoa(String.fromCharCode.apply(null, new Uint8Array(firstCreatedCredential.rawId)))

        console.log('------END CREATE CREDENTIALS 1------')


        console.log('------BEGIN CREATE CREDENTIALS 2------')

        const secondUserId = `STATIC:${window.crypto.getRandomValues(new Uint32Array(1))[0].toString(36).substring(0, 4)}`

        console.log('Second userHandle:', secondUserId)

        const secondCreatedCredential = await this.createPublicKeyCredential(secondUserId)

        const secondCredentialIdAsBase64 = btoa(String.fromCharCode.apply(null, new Uint8Array(secondCreatedCredential.rawId)))

        console.log('------END CREATE CREDENTIALS 2------')


        console.log('------BEGIN GET CREDENTIALS FROM KEY------')

        const retrievedCredentialFromKey = await navigator.credentials.get({
          publicKey: {
            challenge: window.crypto.getRandomValues(new Uint8Array(10)),
            timeout: 60000, // 1 minutes to open vault, or you need a new challenge.
            allowCredentials: [
              {
                type: 'public-key',
                id: Uint8Array.from(atob(firstCredentialIdAsBase64), c => c.charCodeAt(0))
              },
              {
                type: 'public-key',
                id: Uint8Array.from(atob(secondCredentialIdAsBase64), c => c.charCodeAt(0))
              }
            ]
          }
        })

        console.log('Stored credential retrieved from key: ', retrievedCredentialFromKey.response)

        const retrievedUserHandleAsString = String.fromCharCode.apply(null, new Uint8Array(retrievedCredentialFromKey.response.userHandle))

        console.log('Success: retrieved userHandle = ', retrievedUserHandleAsString)

        console.log('------END GET CREDENTIALS FROM KEY------')
      } catch (error) {
        console.error(error)
      }
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
