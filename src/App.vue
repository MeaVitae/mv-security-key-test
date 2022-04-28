<template>
  <div id="app">
    <div>TEST KEY IMPLEMENTATION - v1.01</div>

    <button
      v-if="!credentials.length"
      @click="create"
    >
      Create Authentication
    </button>

    <button
      v-else
      @click="get"
    >
      Check Authentication
    </button>

    <ol>
      <li v-for="message in messages" :key="message">
        {{ message }}
      </li>
    </ol>

    <div>{{error}}</div>
  </div>
</template>

<script>
export default {
  name: 'App',

  data() {
    return {
      rpId: 'security-key.meavitae.com',
      error: '',
      messages: [],
      credentials: []
    }
  },

  methods: {
    async createPublicKeyCredential (userId = '') {
      const createPublicKeyOptions = {
        // attestation: 'direct', // 

        challenge: window.crypto.getRandomValues(new Uint8Array(10)),

        rp: {
          name: 'Test',
          id: this.rpId
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
          residentKey: 'required', // authenticator must create/store a discoverable credential (a.k.a. resident key, userHandle)
          authenticatorAttachment: 'cross-platform'
        },

        timeout: 120000, // 2 minutes

        excludeCredentials: [],

        extensions: {
          loc: true
        }
      }

      console.log(createPublicKeyOptions)

      const credential = await navigator.credentials.create({
          publicKey: createPublicKeyOptions
        })

      const credentialIdAsBase64 = btoa(String.fromCharCode.apply(null, new Uint8Array(credential.rawId)))
      this.credentials.push({
        credentialId: credentialIdAsBase64,
        userHandle: userId
      })
    },

    async create () {
      if (!window.PublicKeyCredential) {
        return alert(`Your web browser does not meet our security requirements.  Please
        upgrade to Google Chrome or contact MeaVitae support for help.`)
      }

      try {
        this.messages.push('------BEGIN CREATE CREDENTIALS 1------')
        console.log(this.messages[0])

        const firstUserId = `STATIC:${window.crypto.getRandomValues(new Uint32Array(1))[0].toString(36).substring(0, 4)}`
        this.messages.push('First userHandle:' + firstUserId)
        console.log(this.messages[1])

        const firstCreatedCredential = await this.createPublicKeyCredential(firstUserId)
        const firstCredentialIdAsBase64 = btoa(String.fromCharCode.apply(null, new Uint8Array(firstCreatedCredential.rawId)))
        this.messages.push('First firstCredentialIdAsBase64:' + firstCredentialIdAsBase64)
        console.log(this.messages[2])

        this.messages.push('------END CREATE CREDENTIALS 1------')
        console.log(this.messages[3])

        this.messages.push('------BEGIN CREATE CREDENTIALS 2------')
        console.log(this.messages[4])

        const secondUserId = `STATIC:${window.crypto.getRandomValues(new Uint32Array(1))[0].toString(36).substring(0, 4)}`
        this.messages.push('Second userHandle:' + secondUserId)
        console.log(this.messages[5])

        const secondCreatedCredential = await this.createPublicKeyCredential(secondUserId)
        const secondCredentialIdAsBase64 = btoa(String.fromCharCode.apply(null, new Uint8Array(secondCreatedCredential.rawId)))
        this.messages.push('Second secondCredentialIdAsBase64:' + secondCredentialIdAsBase64)
        console.log(this.messages[6])

        this.messages.push('------END CREATE CREDENTIALS 2------')
        console.log(this.messages[7])

      } catch (error) {
        this.error = `${error.name} - ${error.message} - ${error.stack}`
        console.error(error)
      }
    },

    async get () {
      try {
        this.messages.push('------BEGIN GET CREDENTIALS FROM KEY------')
        console.log(this.messages[8])

        // const testCredentialIdRegisteredOnSecurityKey = 'Z3/fpqw31h7arCwBeQ6MOZOAqC23ybPOy8gAyvax6uhglyfIljPf7MbYm4C+vtZP'

        const getPublicKeyOptions = {
            rpId: this.rpId,
            challenge: window.crypto.getRandomValues(new Uint8Array(10)),
            timeout: 60000, // 1 minute
            userVerification: 'required',
            allowCredentials: this.credentials
              .map(({ credentialId }) => ({
                type: 'public-key',
                id: Uint8Array.from(atob(credentialId), c => c.charCodeAt(0))
              }))
          }

        this.messages.push(JSON.stringify(getPublicKeyOptions, null, 2))
        console.log(getPublicKeyOptions)

        const retrievedCredentialFromKey = await navigator.credentials.get({
          publicKey: getPublicKeyOptions
        })

        console.log('retrievedCredentialFromKey.response: ', retrievedCredentialFromKey.response)

        const retrievedUserHandleAsString = String.fromCharCode.apply(null, new Uint8Array(retrievedCredentialFromKey.response.userHandle))

        this.messages.push((retrievedUserHandleAsString && `Success: retrieved userHandle = ${retrievedUserHandleAsString}`) || 'FAIL, NO RETRIEVED USER userHandle')
        console.log(this.messages[10])

        this.messages.push('------END GET CREDENTIALS FROM KEY------')
        console.log(this.messages[11])
      } catch (error) {
        this.error = `${error.name} - ${error.message} - ${error.stack}`
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
