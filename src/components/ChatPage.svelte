<script lang="ts">

  
  document.title = 'Chat ';
  import { onMount, afterUpdate } from 'svelte';
  import gravatar from 'gravatar';
  import { io } from 'socket.io-client';


  function generateProfilePicture(username: string) {
    // Generate a Gravatar URL based on the user's email (you can customize this logic)
    const emailHash = gravatar.url(username, { s: '200', d: 'identicon', r: 'pg' });
    return emailHash;
  }
  const localStorageKey = 'ipAddress';
  const url = localStorage.getItem(localStorageKey);
  const wsUrl = `ws://${url}:9001`;
  const token = localStorage.getItem('token');
  let messages: { username: string; message: string }[] = [];
  let message = '';
  let chatContainer: HTMLElement | null = null;
  let users: string[] = [];

  let ws = io(wsUrl, { transports : ['websocket'] });
  ws.on("connect", handleOpen);
  ws.on("chat", handleMessage);
  ws.on("joined", handleJoin)
  ws.on("left", handleLeft)
  ws.on("error", handleError)
  ws.on("users", handleUsers)

  function handleUsers(newUsers: string[]) {
    users = newUsers;
  }

  
  function handleOpen() {
    ws.emit("join", token);
  }

  function handleError(error: string) {
    if (error === "Invalid token") {
      window.location.href = '/login';
    }
  }

  function handleJoin(username: string) {
    handleMessage("System", `${username} joined the chat`);
    users = [...users, username];
  }

  function handleLeft(username: string) {
    handleMessage("System", `${username} left the chat`);
    users = users.filter(user => user !== username);
  }

  function handleMessage(username: string, message: string) {
    let chatMessage = { username, message };
    if (messages.length > 1024) {
      messages = messages.slice(256);
    }
    messages = [...messages, chatMessage];
  }

  function handleSend() {
    if (isReplying){
      isReplying = false;
      ws.emit("chat","@"+replyBody.get("sender")+"  "+ replyBody.get("message")+"\n\n");
    }
    ws.emit("chat", message);
    message = '';
  }

  function handleLogout() {
    ws.emit("logout");
    localStorage.removeItem('token');
    handleDisconnect();
  }

  function handleDisconnect() {
    window.location.href = '/login';
  }

  function handleKeyPress(event: KeyboardEvent) {
    if (event.key === 'Enter' && !event.shiftKey) {
      event.preventDefault(); // Prevent the Enter key from adding a new line
      scrollToBottom();
      handleSend();
    }
  }

  function scrollToBottom(force: boolean = false) {
    if (chatContainer) {
      if (chatContainer.scrollHeight - chatContainer.scrollTop < 1000 || force) {
        chatContainer.scrollTop = chatContainer.scrollHeight;
      }
    }
  }

  let isHovered = false;

function showHoverPanel() {
  isHovered = true;
}

function hideHoverPanel() {
  isHovered = false;
}

function handleReact(sender: string) {
  // Handle react action
  alert(`React to message from ${sender}`);
}

let isReplying = false;
let replyBody : Map<string,string> = new Map<string,string>();
function handleReply(sender: string , message: string) {
  isReplying = true;
  //get <p> by id='reply'
  if(message.length >10){
    replyBody.set("message",message.slice(0,10)+'...')
  }else {
    replyBody.set("message",message)

  }
  replyBody.set("sender", sender)
  const rtext = document.getElementById("reply")
  if(rtext){
    rtext.innerHTML="->@"+replyBody.get("sender")+" "+replyBody.get("message")

  }
  
  // alert(`Reply to message from ${sender}`);
}

  afterUpdate(scrollToBottom);
</script>
  
  <div class="chat-container">
    <div class="chat-header">
      <div class="chat-header-text">Global Chat - {url}</div>
      <div class="logout-btn">
        <a href="#top" on:click={handleLogout}>Logout
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-box-arrow-right" viewBox="0 0 16 16">
            <path fill-rule="evenodd" d="M10 12.5a.5.5 0 0 1-.5.5h-8a.5.5 0 0 1-.5-.5v-9a.5.5 0 0 1 .5-.5h8a.5.5 0 0 1 .5.5v2a.5.5 0 0 0 1 0v-2A1.5 1.5 0 0 0 9.5 2h-8A1.5 1.5 0 0 0 0 3.5v9A1.5 1.5 0 0 0 1.5 14h8a1.5 1.5 0 0 0 1.5-1.5v-2a.5.5 0 0 0-1 0v2z"/>
            <path fill-rule="evenodd" d="M15.854 8.354a.5.5 0 0 0 0-.708l-3-3a.5.5 0 0 0-.708.708L14.293 7.5H5.5a.5.5 0 0 0 0 1h8.793l-2.147 2.146a.5.5 0 0 0 .708.708l3-3z"/>
          </svg>
        </a>
        
      </div>
      <div class="logout-btn">
        <a href="#top" on:click={handleDisconnect}>Disconnect</a>
      </div>
    </div>
    <div class="chat-messages" bind:this={chatContainer}>
      {#each messages as chatMessage, index (index)}
        {#if index === 0 || chatMessage.username !== messages[index - 1].username}
          <div class="message" key={index} role="button" tabindex="0" aria-label="Message Options" on:mouseenter={showHoverPanel} > 
            <div class="sender-avatar">
              {#if chatMessage.username === 'System'}
              <img src={'https://cdn-icons-png.flaticon.com/512/305/305098.png'} alt="Sender Avatar" />
              {:else}
              <img src={generateProfilePicture(chatMessage.username)} alt="Sender Avatar" />
              {/if}
            </div>
              <div class="message-content">
                <div class="sender-name">{chatMessage.username}</div>
              <div class="message-text" style="white-space: pre-line;">{@html chatMessage.message.replace(/\n/g, "<br>")}</div>
            </div>
            
          {#if isHovered}
              <div style= "margin:0; padding:0" class="hover-panel">
                <!-- <button on:click={() => handleReact(chatMessage.sender)}>React</button> -->
                <button on:click={() => handleReply(chatMessage.username,chatMessage.message)}>
                  <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" fill="currentColor" class="bi bi-reply-all-fill" viewBox="0 0 16 16">
                    <path d="M8.021 11.9 3.453 8.62a.719.719 0 0 1 0-1.238L8.021 4.1a.716.716 0 0 1 1.079.619V6c1.5 0 6 0 7 8-2.5-4.5-7-4-7-4v1.281c0 .56-.606.898-1.079.62z"/>
                    <path d="M5.232 4.293a.5.5 0 0 1-.106.7L1.114 7.945a.5.5 0 0 1-.042.028.147.147 0 0 0 0 .252.503.503 0 0 1 .042.028l4.012 2.954a.5.5 0 1 1-.593.805L.539 9.073a1.147 1.147 0 0 1 0-1.946l3.994-2.94a.5.5 0 0 1 .699.106z"/>
                  </svg>
                </button>
              </div>
          {/if}
        </div>
            
        {:else}
        
        <div style="padding: 3px; margin: 3px; padding-left: 8vh;" class="message" key={index} role="button" tabindex="0" aria-label="Message Options" on:mouseenter={showHoverPanel} >
          <div class="message-text" style="white-space: pre-line;">{@html chatMessage.message.replace(/\n/g, "<br>")}</div>
          {#if isHovered}
              <div style= "margin:0; padding:0" class="hover-panel">
                <!-- <button on:click={() => handleReact(chatMessage.sender)}>React</button> -->
                <button on:click={() => handleReply(chatMessage.username,chatMessage.message)}>
                  <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" fill="currentColor" class="bi bi-reply-all-fill" viewBox="0 0 16 16">
                    <path d="M8.021 11.9 3.453 8.62a.719.719 0 0 1 0-1.238L8.021 4.1a.716.716 0 0 1 1.079.619V6c1.5 0 6 0 7 8-2.5-4.5-7-4-7-4v1.281c0 .56-.606.898-1.079.62z"/>
                    <path d="M5.232 4.293a.5.5 0 0 1-.106.7L1.114 7.945a.5.5 0 0 1-.042.028.147.147 0 0 0 0 .252.503.503 0 0 1 .042.028l4.012 2.954a.5.5 0 1 1-.593.805L.539 9.073a1.147 1.147 0 0 1 0-1.946l3.994-2.94a.5.5 0 0 1 .699.106z"/>
                  </svg>
                </button>
              </div>
          {/if}
        </div>
        {/if}
      {/each}
    </div>
    <div class="input-container-container">
      


      <div style="padding-left: 2vh; display:flex" class="">
      {#if isReplying}
        <button style="margin-right: 5px; background-color: #40444b; border: none; color: white; padding: 2px 6px; border-radius: 3px; cursor: pointer; overflow:visible; z-index: 10000 !important;" on:click={() => isReplying = false}>X</button>
        <p id="reply">->@{replyBody.get("sender")+" "+replyBody.get("message")} </p>
      {/if}
      </div>
      <div class="input-container">
        <textarea rows="2" placeholder="Type a message..." bind:value={message} on:keydown={handleKeyPress}></textarea>
        <a class="send-btn" href="#top" on:click={handleSend}>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="16"
            height="16"
            fill="currentColor"
            class="bi bi-send-fill"
            viewBox="0 0 16 16"
          >
            <path
              d="M15.964.686a.5.5 0 0 0-.65-.65L.767 5.855H.766l-.452.18a.5.5 0 0 0-.082.887l.41.26.001.002 4.995 3.178 3.178 4.995.002.002.26.41a.5.5 0 0 0 .886-.083l6-15Zm-1.833 1.89L6.637 10.07l-.215-.338a.5.5 0 0 0-.154-.154l-.338-.215 7.494-7.494 1.178-.471-.47 1.178Z"
            ></path>
          </svg>
        </a>
      </div>
    </div>
  </div>
  
  <style>
    /* Discord-like theme */
    .chat-container {
      max-width: 100%;
      margin: 0 auto;
      display: flex;
      flex-direction: column;
      background-color: #36393f;
      color: white;
      height: 100vh;
      overflow: hidden; /* Disable any scrollbars outside the chat container */
    }
  
    .chat-messages {
      flex: 1;
      border: 0px solid #ccc;
      /* padding: 10px; */
      overflow-y: auto;
      overflow-x: hidden; /* Disable horizontal scroll */
      background-color: transparent;
      scrollbar-width: thin;
      scrollbar-color: #202225 #202225; /* Scrollbar color and track color */
      padding-bottom: 5vh;
    }
  
    .chat-messages::-webkit-scrollbar {
      width: 8px;
      border-radius: 8px; /* Rounded corners */
    }
  
    .chat-messages::-webkit-scrollbar-thumb {
      background-color: #585d63; /* Thumb color */
      border-radius: 8px; /* Rounded corners */
    }
  
    .message {
      margin: 5px 0;
      padding: 5px;
      padding-left: 2vh;
      margin-left: 0 !important;
      display: flex;
      align-items: center;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }
  
    .message:hover {
      background: rgb(2, 0, 36);
      background: linear-gradient(90deg, rgba(2, 0, 36, 1) 0%, rgba(42, 42, 42, 1) 0%, rgba(0, 212, 255, 0) 50%);
    }
    .message-text {
        white-space: pre-wrap;
        overflow-wrap: break-word;
        word-break: break-all;
    }
  
    .message-content {
      text-align: left;
      word-wrap: break-word; /* Wrap long words when resizing */
    }
  
    .sender-name {
      font-weight: bold;
    }
  
    .input-container-container {
      padding: 10px;
      padding-top: 0;
      /* display: flex; */
      justify-content: center;
      align-items: center;
      background-color: #36393f;
      box-shadow: #36393f 0px 0px 100px 0.5px;
      /* max-width: 100vh; */
    }
  
    .input-container {
      padding: 10px;
      padding-top: 0;
      display: flex;
      justify-content: space-between;
      align-items: center;
      background-color: #36393f;
      box-shadow: #36393f 0px 0px 100px 0.5px;
    }
  
    textarea {
      flex: 1;
      padding: 5px;
      padding-top: 6px;
      resize: none;
      background-color: #40444b;
      color: white;
      border: none;
      outline: none;
      border-radius: 5px 0 0 5px;
    }
  
    textarea:focus {
      outline: none;
    }
  
    .send-btn {
      cursor: pointer;
      padding-top: 10px;
      padding-bottom: 9px;
      padding-right: 15px;
      background-color: #40444b;
      border: none;
      color: white;
      text-decoration: none;
      border-radius: 0 5px 5px 0;
      outline: none;
    }
  
    .send-btn svg {
      width: 20px;
      height: 20px;
      display: block;
      rotate: 25deg;
      padding-bottom: 3px;
    }
  
    .send-btn svg:hover {
      rotate: 40deg;
      transition: all 0.2s ease-in-out;
    }
  
    .send-btn:hover {
      text-decoration: none;
      color: white;
      background-color: #202225;
      padding-left: 10px;
      transition: all 0.2s ease-in-out;
    }
    .send-btn:active {
      background-color: #202225;
      transform: scale(0.9);
      transition: all 0.2s ease-in-out;
    }
    .chat-header {
      background-color: #313236;
      display: flex;
      justify-content: space-between;
      padding: 10px;
      box-shadow: #36393f 0px 0px 100px 0.5px;
      /* background: transparent; */
      /* backdrop-filter: blur(20px); */
    }
    
  
    .logout-btn {
      float: right;
      padding: 5px;
      padding-left: 10px;
      padding-right: 10px;
      border-radius: 5px;
      background-color: #40444b;
      color: white;
      text-decoration: none;
      transition: all 0.2s ease-in-out;
      cursor: pointer;
    }
    .logout-btn:hover {
      background-color: #202225;
      transition: all 0.2s ease-in-out;
    }
    .logout-btn:active {
      background-color: #202225;
      transform: scale(0.9);
      transition: all 0.2s ease-in-out;
    }
      
    
    .logout-btn a{
      color: white;
      text-decoration: none;
    }
    .logout-btn a svg{
      color: white;
      text-decoration: none;
      position: relative;
      top: 3px;
      left: 4px;
    }

    .sender-avatar {
      margin-right: 10px;
      border-radius: 50%; 
      overflow: hidden;
      width: 32px;
      height: 32px;
    }
  
    .sender-avatar img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
    
    /* Hover panel styles */
.hover-panel {
  position: relative;
  top: -20px; 
  right: -50px; 
  /* background-color: rgba(0, 0, 0, 0.3); */
  border-radius: 5px;
  display: flex;
  gap: 5px;
  padding: 5px;
  margin: 0;
  z-index: 100 !important;
  overflow: visible;
}

.hover-panel button {
  background-color: #40444b;
  border: none;
  color: white;
  padding: 2px 6px;
  border-radius: 3px;
  cursor: pointer;
  overflow:visible;
  z-index: 10000 !important;
}

.hover-panel button:hover {
  background-color: #202225;
}

/* Show/hide hover panel on hover */
.message:hover .hover-panel {
  display: flex
}

/* Hide the hover panel by default */
.hover-panel {
  display: none;
}
  </style>
  
  
  
  
  