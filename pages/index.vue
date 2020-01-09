<template lang="pug">
  div
    section.hero(v-if="!userFilled")
      .hero-body
        .container
          .section
            h2.subtitle Ingrese Su nombre
          .section
            .field.has-addons
              .control
                input.input.is-expanded(
                  type="text"
                  placeholder="Ingrese su nombre"
                  v-model="user"
                  v-on:keyup.enter="setUser"
                  )
              .control
                a.button(
                  @click.stop="setUser"
                  )
                  | Enviar
    template(v-else)
      section.section
        template(v-for="message in messages")
          p.message(:ref="message.messageId") {{ message.user }}: {{ message.text }}
      nav.navbar.is-fixed-bottom
        .container
          .field.has-addons
            p.control.is-expanded
              input.input(
                ref="message"
                type="text"
                placeholder="Ingrese su mensaje"
                v-model="message"
                v-on:keyup.enter="sendMsg"
                v-bind:class="connectedClass"
                :disabled="!connected"
                )
            p.control
              a.button(
                @click.stop="sendMsg"
                v-bind:class="connectedClass"
                :disabled="!connected"
                )
                | Enviar
</template>

<script>
const io = require('socket.io-client')

const socket = io('https://personal-development-217118.appspot.com/')

export default {
  data () {
    return {
      userFilled: false,
      user: '',
      messages: [],
      message: '',
      connected: false
    }
  },
  computed: {
    connectedClass () {
      const currentClass = this.connected ? 'is-success' : 'is-danger'
      return currentClass
    }
  },
  mounted () {
    socket.on('connect', this.handleConnect)
    socket.on('disconnect', this.handleDisconnect)
    socket.on('chat message', (message) => {
      this.handleNewMessage(message)
    })
  },
  methods: {
    sendMsg () {
      this.messageFocus()
      if (this.connected) {
        const date = new Date().valueOf()
        socket.emit('chat message', {
          text: this.message,
          user: this.user,
          messageId: date
        })
        this.message = ''
      }
    },
    handleNewMessage (message) {
      this.messages.push(message)
      setTimeout(() => {
        this.focusMessage(message.messageId)
      }, 50)
    },
    handleConnect () {
      this.connected = true
    },
    handleDisconnect () {
      this.connected = false
    },
    focusMessage (messageId) {
      this.$refs[messageId][0].scrollIntoView()
    },
    setUser () {
      this.userFilled = true
      setTimeout(() => {
        this.messageFocus()
      }, 50)
    },
    messageFocus () {
      this.$refs.message.focus()
    }
  }
}
</script>

<style>
</style>
