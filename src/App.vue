<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'
import { io } from 'socket.io-client'
let token = localStorage.getItem('token')
console.log(token)
const socket = io('ws://test-deploy-backend-rouge.vercel.app/', {
  transports: ['websocket']
})

socket.on('connect', () => {
  console.log('connect')
})

socket.on('new-message', (data) => {
  showMessageFromServer(data)
})

function showMessageFromServer(data:any) {
  message.value = data
  setTimeout(() => {
    message.value = ''
  }, 1000)
}


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
  socket.emit('message', {message: 'hello'})
}
</script>

<template>
  <div>
    <button @click="handleSendMessage">Send message</button>
    <button @click="testClick">Get todos!</button>
    <button @click="clearTodos">Clear Todos!</button>
    <div>тут будет сообщение: {{message}}</div>
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
