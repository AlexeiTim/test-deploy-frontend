<script setup lang="ts">
import { createClient } from '@supabase/supabase-js'
import { onMounted, reactive, ref } from 'vue'
import axios from 'axios'

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
  console.log('send message')
  if (!newMessage.value.name || !newMessage.value.description) return
  try {
    state.loading = true
    await supabaseClient
      .from('posts')
      .insert(newMessage.value)
  } catch (e) {
    console.log('Error: ', e)
  } finally {
    state.loading = false
  }
}


const fetchMessages = async () => {
  try {
    state.loading = true
    const { data } = await supabaseClient
      .from('posts')
      .select('*')
    if (data) messages.value = [...data]
    console.log(data)
  } catch (e) {
    console.log(e)
  }
}

function observerPosts(payload: any) {
  messages.value.push(payload.new)
}
function listenInsertPost() {
  supabaseClient
    .channel('posts')
    .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'posts' }, observerPosts)
    .subscribe()
}

onMounted(async () => {
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

function handleSendMessage() {
  // socket.emit('message', {message: 'hello'})
}

onMounted(() => {

})
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
    <button @click="handleSendMessage">Send message</button>
    <button @click="testClick">Get todos!</button>
    <button @click="clearTodos">Clear Todos!</button>
    <div>тут будет сообщение: {{ message }}</div>
    <div v-for="todo in todos" :key="todo.id" style="border:1px solid black; margin-bottom: 10px;">
      <div>ID: {{ todo.id }}</div>
      <div>TITLE: {{ todo.title }}</div>
    </div>
    <RouterView />
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
