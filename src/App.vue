<script setup lang="ts">
import { createClient } from '@supabase/supabase-js'
import { onMounted, reactive, ref } from 'vue'
import axios from 'axios'

interface MyEvent {
  [key: string]: Function[]
}

class MyWebSocket {
  events: MyEvent

  constructor() {
    this.events = {}
  }

  on(event: string, cb: Function) {
    console.log('subscribe on :', event, cb)
    this.supabaseListen(event, (data: any) => {
      cb((data.new[event]))
    })
    // this.supabaseListen(event, cb)
    // if (!this.events[event]) this.events[event] = []
    // this.events[event].push(() => () => {
    //   console.log('event from cb :', event)
    // })
  }

  async send(event: string, payload: any) {
    await this.supabaseSend(event, payload)
    // for (const type of this.events[event])
    //   console.log('send :', event, type)
      //
  }

  async supabaseSend(event: string, data: any) {
    console.log('supabase send :', event, data)
    await supabaseClient
      .from(event)
      .insert(data)
  }

  supabaseListen(event: string, cb: Function) {
    console.log('supabase listen :', JSON.stringify({ event, cb }))
    return supabaseClient
      .channel(event)
      .on('postgres_changes', { event: 'INSERT', schema: 'public', table: event }, (data) => {
        console.log('supabaseListen callback :', event)
        cb(data)
      })
      .subscribe()
  }
}


function listenInsertLocalDescription() {
  console.log('listen insert local description')
  // supabaseClient
  //   .channel('local_description')
  //   .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'local_description' }, observerLocalDescription)
  //   .subscribe()
}

function observerLocalDescription(data: any) {
  console.log(JSON.parse(data.new.description), 'observerLocalDescription')
}

async function insertLocalDescription(localDescription: any) {
  console.log(localDescription, 'insertLocalDescription')
  // await supabaseClient
  //   .from('local_description')
  //   .insert({
  //     description: JSON.stringify(localDescription)
  //   })
}

const websocket: MyWebSocket = new MyWebSocket()

interface Post {
  id: number
  created_at: string
  name: string
  description: string
}

const supabaseClient = createClient('https://hnfhoxgvhdxicrmypedy.supabase.co', 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImhuZmhveGd2aGR4aWNybXlwZWR5Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3MDYzNzQ0MzksImV4cCI6MjAyMTk1MDQzOX0.Qmi3G2ynrIDCB9SKqHHIVmiEO3BQKtPqi_wP864sTLA')


const state = reactive({
  loading: false,
  disabled: false,
  error: undefined
})
const messages = ref<Post[]>([])
type NewMessage = Pick<Post, 'name' | 'description'>
const newMessage = ref<NewMessage>({
  name: '',
  description: ''
})

const sendMessage = async () => {
  console.log('send message to db')
  // if (!newMessage.value.name || !newMessage.value.description) return
  // try {
  //   state.loading = true
  //   await supabaseClient
  //     .from('posts')
  //     .insert(newMessage.value)
  // } catch (e) {
  //   console.log('Error: ', e)
  // } finally {
  //   state.loading = false
  // }
}

const fetchMessages = async () => {
  console.log('fetch messages from db')
  // try {
  //   state.loading = true
  //   const { data } = await supabaseClient
  //     .from('posts')
  //     .select('*')
  //   if (data) messages.value = [...data]
  // } catch (e) {
  //   console.log(e)
  // }
}

function observerPosts(payload: any) {
  messages.value.push(payload.new)
}

function listenInsertPost() {
  console.log('listen insert Post')
  // supabaseClient
  //   .channel('posts')
  //   .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'posts' }, observerPosts)
  //   .subscribe()
}


onMounted(async () => {
  listenInsertLocalDescription()
  listenInsertPost()
  await fetchMessages()
})

interface Todo {
  id: number
  title: string
}

const todos = ref<Todo[]>([])
const message = ref('')

async function testClick() {
  const { data } = await axios.get<Todo[]>('https://test-deploy-backend-62zx.vercel.app/api/todos')
  todos.value = data
}

function clearTodos() {
  todos.value = []
}

// interface Consumer {
//   id: number
// }
// const consumers = ref<Consumer[]>([])
// const videoBlocks = ref([])
// const myVideoId = ref<Number | null>(null)
const servers: RTCConfiguration = {
  iceServers: [{
    "urls": [
      "turn:13.250.13.83:3478?transport=udp"
    ],
    "username": "YzYNCouZM1mhqhmseWk6",
    "credential": "YzYNCouZM1mhqhmseWk6"
  }]
}
let localPeerConnection: any, remotePeerConnection: any = null
const ownerVideo = ref()
const otherVideo = ref()

async function startMyStream() {
  console.log('init start my stream')
  console.log('get user media devices')
  const mediaStream = await navigator.mediaDevices.getUserMedia({
    video: true,
    audio: false
  })
  console.log('set my media sttream to video tag')
  ownerVideo.value.srcObject = mediaStream
  console.log('get my video track')
  const ownerVideoTrack = mediaStream.getVideoTracks()[0]
  console.log('create my LocalPeerConnection')
  localPeerConnection = new RTCPeerConnection(servers)
  console.log('add eventlistener on localpeer: icecandidate')
  localPeerConnection.addEventListener('icecandidate', (event: RTCIceCandidate) => {
    console.log('call listener localpeer icacandidate')
    if (event.candidate) {
      console.log('send by websocket local_candidate')
      websocket.send('local_candidate', {
        local_candidate: JSON.stringify(event.candidate)
      })
    }
  })
  websocket.on('remote_candidate', (candidate) => {
    localPeerConnection.addIceCandidate(new RTCIceCandidate(candidate))
  })

  websocket.on('remote_description', (description) => {
    localPeerConnection.setRemoteDescription(description)
  })
  console.log('add track')
  localPeerConnection.addTrack(ownerVideoTrack, mediaStream)
  console.log('create offer')
  const offer = await localPeerConnection.createOffer()
  console.log('set local description/offer')
  localPeerConnection.setLocalDescription(offer)
  // await websocket.send('local_description', offer)
  console.log('init websocket send local_description')
  await websocket.send('local_description', { local_description: JSON.stringify(offer) })
}

remotePeerConnection = new RTCPeerConnection(servers)

websocket.on('local_description', async (description: any) => {
  console.log('on local description :', description)
  remotePeerConnection.setRemoteDescription(description)

  remotePeerConnection.addEventListener('icecandidate', (event: RTCIceCandidate) => {
    if (event.candidate) {
      websocket.send('remote_candidate', {remote_candidate: JSON.stringify(event.candidate)})
    }
  })

  websocket.on('local_description', (candidate: RTCIceCandidate) => {
    remotePeerConnection.addIceCandidate(new RTCIceCandidate(candidate))
  })

  remotePeerConnection.addEventListener('track', (event: any) => {
    otherVideo.value.srcObject = event.streams[0]
  })

  const answerDescription = await remotePeerConnection.createAnswer()
  remotePeerConnection.setLocalDescription(answerDescription)
  console.log('remote peer send description')
  await websocket.send('remote_description', { remote_description: JSON.stringify(answerDescription)})
})


function deleteMyVideo() {
  ownerVideo.value.srcObject = null
  otherVideo.value.srcObject = null
}

function handleSendMessage() {
  // socket.emit('message', {message: 'hello'})
}
</script>

<template>
  <div>
    <div>
      <label>Create message</label>
      <input v-model="newMessage.name" />
      <input v-model="newMessage.description" />
      <button @click="sendMessage">Send message</button>
    </div>
    <div>
      <h2>Messages</h2>
      <div style="border: 1px solid black; padding: 4px" v-for="message in messages" :key="message.id">
        <p>Name: {{ message.name }}</p>
        <p>Description: {{ message.description }}</p>
      </div>
    </div>
    <button @click="testClick">Get todos!</button>
    <button @click="clearTodos">Clear Todos!</button>
    <div v-for="todo in todos" :key="todo.id" style="border:1px solid black; margin-bottom: 10px;">
      <div>ID: {{ todo.id }}</div>
      <div>TITLE: {{ todo.title }}</div>
    </div>
    <div>
      <h2>Video call</h2>
      <button @click="startMyStream">Start video chat</button>
      <button @click="deleteMyVideo">Out video</button>
      <div class="video-container" style="display: flex; justify-content: start; flex-wrap: wrap">
        <video ref="ownerVideo" width="200" height="200" autoplay></video>
        <video ref="otherVideo" width="200" height="200" autoplay></video>
        <!--        <video ref="videoBlocks" autoplay :id="String(consumer.id)" v-for="consumer in consumers" height="200" :key="consumer.id"></video>-->
      </div>
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

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

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
