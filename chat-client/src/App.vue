<script setup>
import {io} from 'socket.io-client';
import { onBeforeMount, ref } from 'vue';

const socket = io('http://localhost:3000', {credentials:'include'});
const messages = ref([]);
const name = ref('');
const messageText = ref('');
const joined = ref(false);

async function getUsers() {
    let url = 'http://localhost:3000/getUserIdentity';
    try {
        let res = await fetch(url, {credentials: 'include'});
        return await res.json();
    } catch (error) {
        console.log(error);
    }
}

onBeforeMount(() => {
  socket.emit('findAllMessages', {}, (response) => {
    messages.value = response;
  })

  socket.on('message', (message) =>
  {
    messages.value.push(message);
  });
});

 const join = async () => {
  const nome = await getUsers();
  console.log(nome)
  socket.emit('join', {name: nome.username}, () =>{
    joined.value = true
  })
}

const sendMessage = () => {
  socket.emit('createMessage', {text: messageText.value}, () => {
    messageText.value = '';
  })
}
</script>

<template>
 <div class="chat"> 
  <div v-if="!joined">
    <form @submit.prevent="join">
      <label>what's your name?</label>
      <input v-model="name">
      <button type="submit">Send</button>
    </form>
  </div>

    <div class="chat-container" v-else>
        <div class="message-container">
          <div v-for="message in messages">
            [{{message.name }}]: {{ message.text }}
          </div>
        </div>
      <div class="messageInput">
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messageText"/>
            <button type="submit">Send</button>
        </form>
      </div>
    </div>
 </div>
</template>
<style>
@import './assets/base.css';
  .chat{
    padding: 10px;
    height: 80vh;
  }

  .chat-container{
    display: flex;
    flex-direction: column;
    height:100%;
  }

  .messages-container{
    flex:3;
  }
</style>
