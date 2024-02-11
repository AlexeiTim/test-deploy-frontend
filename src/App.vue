<script setup lang="ts">
import { io } from 'socket.io-client'
import { ref } from 'vue'

interface Post {
  id: number
  created_at: string
  name: string
  description: string
}

const messages = ref([])

const sendMessage = () => {
  const newMessage = {
    text: 'Hello from client!!!' + ' ' + Date.now()
  }
  socket.emit('new-message', newMessage)
}

const socket = io('ws://127.0.0.1:8000/', {
  transports: ['websocket']
})
socket.on('connect', () => {
  console.log('socket connect')
})

socket.on('new-message', (data) => {
  messages.value.push(data)
  console.log('nget by socket ew-message', data)
})

const ownerVideo = ref()
const otherVideo = ref()

const servers: RTCConfiguration = {
  iceServers: [
    {
      urls: ['stun:stun2.l.google.com:19302']
    }
  ]
  // optional: [],
  // mandatory: {
  //   OfferToReceiveAudio: true,
  //   OfferToReceiveVideo: true
}
const offerOptions = {
  offerToReceiveAudio: true,
  offerToReceiveVideo: true
}

let localDescription = null
let remoteDescription = null
const userId = Date.now()

async function sendCall() {
  const mediaStream = await navigator.mediaDevices.getUserMedia({
    video: { height: 200, width: 200 },
    audio: true
  })
  ownerVideo.value.srcObject = mediaStream
  const myTrack = mediaStream.getTracks()[0]

  const localPeerConnection = new RTCPeerConnection(servers)

  localPeerConnection.addEventListener('icecandidate', (event) => {
    if (event.candidate) {
      console.log('local-candidate init')
      socket.emit('local-candidate', event.candidate)
    }
  })

  socket.on('remote-candidate', (candidate) => {
    localPeerConnection.addIceCandidate(new RTCIceCandidate(candidate))
  })

  socket.on('remote-description', (data) => {
    // if (userId == data.userId) return
    console.log('come remote description :', data.answer)
    localPeerConnection.setRemoteDescription(data.answer)
  })
  console.log(myTrack, mediaStream, 'test traack stream')
  localPeerConnection.addTrack(myTrack, mediaStream)

  localPeerConnection.createOffer().then((offer) => {
    localPeerConnection.setLocalDescription(offer)
    socket.emit('local-description', {
      userId,
      offer
    })
  })
}

const remotePeerConnection = new RTCPeerConnection(servers)
socket.on('local-description', (data) => {
  // if (userId == data.userId) return
  console.log('come local description', data.offer)
  remotePeerConnection.setRemoteDescription(data.offer)

  remotePeerConnection.addEventListener('icecandidate', (event) => {
    if (event.cancelable) {
      socket.emit('remote-candidate', event.candidate)
      console.log('remote candidate', JSON.stringify(event.cancelable))
    }
  })

  remotePeerConnection.addEventListener('track', (event) => {
    console.log('track')
    console.log(event)
    otherVideo.value.srcObject = event.streams[0]
  })

  socket.on('local-candidate', (candidate) => {
    remotePeerConnection.addIceCandidate(new RTCIceCandidate(candidate))
  })

  remotePeerConnection.createAnswer().then((answer) => {
    remotePeerConnection.setLocalDescription(answer)
    socket.emit('remote-description', {
      userId,
      answer
    })
  })
})
// const userId = Date.now()

// async function sendCall() {
//   const myStream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true })
//   ownerVideo.value.srcObject = myStream
//   const myTrack = myStream.getTracks()[0]

//   const peerConnection = new RTCPeerConnection(servers)
//   peerCondatanection.addEventListener('icecandidate', (e) => console.log(e))

//   socket.on('answer', async (message) => {
//     if (message.userId == userId) return
//     const remoteAnswerDescription = new RTCSessionDescription(message.answer)
//     peerConnection.setRemoteDescription(remoteAnswerDescription)
//   })

//   const offer = await peerConnection.createOffer()
//   await peerConnection.setLocalDescription(offer)
//   localDescription.value =
//   peerCondatanection.onicecandidate = function (e) {
//     console.log(e)
//   }

//   socket.emit('offer', { userId, offer })
// }

// const peerCondatanection = new RTCPeerConnection(servers)
// peerCondatanection.addEventListener('icecandidate', (e) => console.log(e))
// peerCondatanection.addEventListener('track', (e) => console.log(e))

// socket.on('offer', async (message) => {
//   if (message.userId === userId) return

//   console.log('on offer :', message)
//   const desc = new RTCSessionDescription(message.offer)
//   await peerCondatanection.setRemoteDescription(desc)
//   const answer = await peerCondatanection.createAnswer()
//   peerCondatanection.setLocalDescription(answer)

//   socket.emit('answer', {
//     userId,
//     answer
//   })
// })
// peerCondatanection.addEventListener('icecandidate', (e) => console.log(e))
// peerCondatanection.addEventListener('track', (e) => console.log(e))

// const localDescription = ref('')
// const remoteDescription = ref('')

// const localCandidate = ref('')
// const remoteCandidate = ref('')
</script>

<template>
  <div>
    <button @click="sendMessage">Send message</button>
    <div>
      <div v-for="message in messages" :key="message.text">{{ message.text }}</div>
    </div>
    <h2>Video call</h2>
    <button @click="sendCall">Send call</button>
    <div>
      <h2>Local Description</h2>
      <p>{{ localDescription }}</p>
    </div>

    <div>
      <h2>remote Description</h2>
      <p>{{ remoteDescription }}</p>
    </div>
    <div class="video-container" style="display: flex; justify-content: start; flex-wrap: wrap">
      <video ref="ownerVideo" width="200" height="200" autoplay muted></video>
      <video ref="otherVideo" width="200" height="200" autoplay muted></video>
      <!--        <video ref="videoBlocks" autoplay :id="String(consumer.id)" v-for="consumer in consumers" height="200" :key="consumer.id"></video>-->
    </div>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
