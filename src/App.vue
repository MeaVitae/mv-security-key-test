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
    consoleLogLastMessage () {
      console.log(this.messages[this.messages.length - 1])
    },

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

      this.messages.push('credentialIdAsBase64:' + credentialIdAsBase64)
      this.consoleLogLastMessage()
    },

    async create () {
      if (!window.PublicKeyCredential) {
        return alert(`Your web browser does not meet our security requirements.  Please
        upgrade to Google Chrome or contact MeaVitae support for help.`)
      }

      try {
        this.messages.push('------BEGIN CREATE CREDENTIALS 1------')
        this.consoleLogLastMessage()

        const firstUserId = `STATIC:${window.crypto.getRandomValues(new Uint32Array(1))[0].toString(36).substring(0, 4)}`
        this.messages.push('First userHandle:' + firstUserId)
        this.consoleLogLastMessage()

        await this.createPublicKeyCredential(firstUserId)

        this.messages.push('------END CREATE CREDENTIALS 1------')
        this.consoleLogLastMessage()

        this.messages.push('------BEGIN CREATE CREDENTIALS 2------')
        this.consoleLogLastMessage()

        const secondUserId = `STATIC:${window.crypto.getRandomValues(new Uint32Array(1))[0].toString(36).substring(0, 4)}`
        this.messages.push('Second userHandle:' + secondUserId)
        this.consoleLogLastMessage()

        await this.createPublicKeyCredential(secondUserId)
        
        this.messages.push('------END CREATE CREDENTIALS 2------')
        this.consoleLogLastMessage()

      } catch (error) {
        this.error = `${error.name} - ${error.message} - ${error.stack}`
        console.error(error)
      }
    },

    async get () {
      try {
        this.messages.push('------BEGIN GET CREDENTIALS FROM KEY------')
        this.consoleLogLastMessage()

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
        this.consoleLogLastMessage()

        this.messages.push('------END GET CREDENTIALS FROM KEY------')
        this.consoleLogLastMessage()
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
