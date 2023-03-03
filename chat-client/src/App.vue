<script setup>
import {io} from 'socket.io-client';
import { onBeforeMount, ref } from 'vue';

const socket = io('http://localhost:3000', {credentials:'include'});
const messages = ref([]);
const name = ref('');
const messageText = ref('');
const joined = ref(false);
const ChatRoom = ref('');
const Password = ref(undefined)
const PasswordToJoin = ref(undefined)
const AdminsVal = ref(undefined);
let joinOrcreate = ref('');

async function addAdmins() {
    let url = 'http://localhost:3000/messages/admin';
    try {
        let res = await fetch(url, {credentials: 'include',headers: {
                        channel : ChatRoom.value, 
                        Admin: AdminsVal.value 
        }});
        return await res.json();
    } catch (error) {
        console.log(error);
    }
}


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
 /*socket.emit('findAllMessages', {}, (response) => {
    messages.value = response;
  })
*/
  socket.on('message', (message) =>
  {
    messages.value.push(message);
  });
});
/*
 const join = async () => {
  const nome = await getUsers();
  console.log(nome)
  socket.emit('join', {name: nome.username}, () =>{
    joined.value = true
  })
}
*/

const join = async () => {
 // const nome = await getUsers();
  socket.emit('join', {name: name.value, channel:ChatRoom.value, Password: PasswordToJoin.value}, () =>{
    joined.value = true
  })
}
  
const sendMessage = async () => {
 // const nome = await getUsers();
  socket.emit('createMessage', {text: messageText.value, channel:ChatRoom.value, name: name.value}, () => {
    messageText.value = '';
  })
}

const createRoom = async () => {
 // const nome = await getUsers();
  socket.emit('createRoom', {channel: ChatRoom.value, name: name.value, Password: Password.value}, () => {
    joined.value = true;
  }) 
}

const AddAdmins = async () => {
  const Admin = await addAdmins();
}

</script>

<template>

 <!-- <div class="creteRoom">
    <form @submit.prevent="createRoom">
      <label>crea un canale</label>
      <input v-model="name">
      <button type="submit">Send</button>
    </form>
  </div>
-->
  <div class="chat"> 

    <div>
      <form>
        <label>type join if u want join a room or
          create if u want create one room
        </label>
      <input v-model="joinOrcreate">
      <button type="submit">send</button> 
    </form>
    </div>

  <div v-if="joinOrcreate == 'create'">
    <form @submit.prevent="createRoom">
      <label>ChatRoom:</label>
      <input v-model="ChatRoom">
      <label> Username </label>
      <input v-model="name">
      <label> Password </label>
      <input v-model="Password">
      <button type="submit">Send</button>
    </form>
  </div>

  <div class="join-room" v-if="joinOrcreate == 'join'">
    <form @submit.prevent="join">
      <label>which chatRoom do you want join?</label>
      <input v-model="ChatRoom">
      <label>username</label>
      <input v-model="name">
      <label>Password</label>
      <input v-model="PasswordToJoin">
      <button type="submit">Send</button>
    </form>
  </div>

  <div class="Admins" v-if="joined">
    <form @submit.prevent="AddAdmins">
      <label>admin to add</label>
      <input v-model="AdminsVal">
      <button type="submit">Send</button>
    </form>
  </div>


    <div class="chat-container" v-if="joined">
      <div>ChatRoom: {{ChatRoom}}</div>
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
