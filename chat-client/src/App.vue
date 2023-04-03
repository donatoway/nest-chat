<script setup>
import {io} from 'socket.io-client';
import { onBeforeMount, ref } from 'vue';

const socket = io('http://localhost:3000', {credentials:'include'});
const messages = ref([]);
const name = ref('');
const messageText = ref('');
const joined = ref(false);
const privateChat = ref(false);
const ChatRoom = ref('');
const useToSilent = ref(undefined);
const userToBan = ref(undefined);
const userToUnBan = ref(undefined);
const Password = ref(undefined);
const PasswordToJoin = ref(undefined)
const AdminsVal = ref(undefined);
const destUser = ref(undefined);
const timer  = ref(0);
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

const muteOn = async () => {
  socket.emit('muteOn', {channel: ChatRoom.value,
                            useToSilent: useToSilent.value,
                            name: name.value,
                            timer: timer.value}, () => {
  }) 
}

const muteOff = async () => {
  socket.emit('muteOff', {channel: ChatRoom.value,
                            useToSilent: useToSilent.value,
                            name: name.value}, () => {
  }) 
}

const banOff = async () => {
  socket.emit('banOff', {channel: ChatRoom.value,
                            userToUnBan: userToUnBan.value,
                            name: name.value}, () => {
  }) 
}


const banUser = async () => {
    socket.emit('banUser', { channel: ChatRoom.value,
                userToBan: userToBan.value,
                name: name.value,
                timer: timer.value,
    })
  }

const disconnectFun = async () => {
  socket.emit('leaveChannel', {channel: ChatRoom.value, name: name.value}, () => {
  }) 
}

const prvChatOn = () => {
  privateChat.value = true;
}

const directMessage = async () => {
 // const nome = await getUsers();
  socket.emit('directMessage', {text: messageText.value, name: name.value, dest: destUser.value}, () => {
    messageText.value = '';
  })
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
          create if u want create one room or direct to send a direct mex
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
  <div v-if="joinOrcreate == 'direct'">
      <form @submit.prevent="prvChatOn">
        <label>your Username</label>
        <input v-model="name">
        <label>destination</label>
        <input v-model="destUser">
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
  <div class="disconnect" v-if="joined">
    <form @submit.prevent="disconnectFun">
      <label>disconnect</label>
      <button type="submit">Send</button>
    </form>
  </div>
  <div class="silentUser" v-if="joined">
    <form @submit.prevent="muteOn">
      <label>silent user</label>
      <input v-model="useToSilent">
      <label>minutes</label>
      <input v-model="timer">
      <button type="submit">Send</button>
    </form>
  </div>

  <div class="banUser" v-if="joined">
    <form @submit.prevent="banUser">
      <label>ban user</label>
      <input v-model="userToBan">
      <label>minutes</label>
      <input v-model="timer">
      <button type="submit">Send</button>
    </form>
  </div>

  <div class="unBan" v-if="joined">
    <form @submit.prevent="banOff">
      <label>ban off</label>
      <input v-model="userToUnBan">
      <button type="submit">Send</button>
    </form>
  </div>

  <div class="unMute" v-if="joined">
    <form @submit.prevent="muteOff">
      <label>mute off</label>
      <input v-model="useToSilent">
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
      <div class="messageInput" v-if="joined">
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messageText"/>
            <button type="submit">Send</button>
        </form>
      </div>
    </div>

    <div class="directChat" v-if="privateChat">
      <div>direct message: </div>
        <div class="message-container">
          <div v-for="message in messages">
            [{{message.name }}]: {{ message.text }}
          </div>
        </div>
      <div class="messageInputDirect" v-if="privateChat">
          <form @submit.prevent="directMessage">
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
