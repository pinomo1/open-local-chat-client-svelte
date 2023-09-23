<script lang="ts">
  
  document.title = 'Chat ';
  import { onMount, afterUpdate } from 'svelte';
  import gravatar from 'gravatar';

  function generateProfilePicture(username: string) {
    // Generate a Gravatar URL based on the user's email (you can customize this logic)
    const emailHash = gravatar.url('user@example.com', { s: '200', d: 'identicon', r: 'pg' });
    return emailHash;
  }
  const localStorageKey = 'ipAddress';
  const wsUrl = `ws://${localStorage.getItem(localStorageKey)}:9001`;
  const token = localStorage.getItem('token');
  let messages: { sender: string; text: string }[] = [];
  let message = '';
  let chatContainer: HTMLElement | null = null;

  let ws: WebSocket;

  onMount(() => {
    ws = new WebSocket(wsUrl);
    ws.addEventListener('open', handleOpen);
    ws.addEventListener('message', handleMessage);
  });

  
  function handleOpen() {
    const joinMessage = `join ${token}`;
    ws.send(joinMessage);
  }

  function handleMessage(event: MessageEvent) {
    const message = event.data;
    if (message.startsWith('0')) {
      const errorMessage = message.slice(2);
      alert(`Error: ${errorMessage}`);
      window.location.href = '/login';
    } else if (message.startsWith('1')) {
      // Do nothing
    } else {
      let sender = message.slice(0, message.indexOf(':'));
      let text = message.slice(message.indexOf(':') + 2);
      let chatMessage = { sender, text };
      if (messages.length > 1024) {
        messages = messages.slice(512);
      }
      messages = [...messages, chatMessage];
    }
  }

  function handleSend() {
    if (message === '') {
      return;
    }
    if (message.length > 1024) {
      alert('Message is too long');
      return;
    }
    const chatMessage = `chat ${token} ${message}`;
    ws.send(chatMessage);
    message = '';
  }

  function handleLogout() {
    const logoutMessage = `logout ${token}`;
    localStorage.removeItem('token');
    ws.send(logoutMessage);
    window.location.href = '/login';
  }

  function handleKeyPress(event: KeyboardEvent) {
    if (event.key === 'Enter' && !event.shiftKey) {
      event.preventDefault(); // Prevent the Enter key from adding a new line
      scrollToBottom();
      handleSend();
    }
  }

  function scrollToBottom() {
    if (chatContainer) {
      if (chatContainer.scrollHeight - chatContainer.scrollTop < 1000) {
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

function handleReply(sender: string) {
  // Handle reply action
  alert(`Reply to message from ${sender}`);
}

  afterUpdate(scrollToBottom);
</script>
  
  <div class="chat-container">
    <div class="chat-header">
      <div class="chat-header-text">Global Chat</div>
      <div class="logout-btn">
        <a href="#top" on:click={handleLogout}>Logout
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-box-arrow-right" viewBox="0 0 16 16">
            <path fill-rule="evenodd" d="M10 12.5a.5.5 0 0 1-.5.5h-8a.5.5 0 0 1-.5-.5v-9a.5.5 0 0 1 .5-.5h8a.5.5 0 0 1 .5.5v2a.5.5 0 0 0 1 0v-2A1.5 1.5 0 0 0 9.5 2h-8A1.5 1.5 0 0 0 0 3.5v9A1.5 1.5 0 0 0 1.5 14h8a1.5 1.5 0 0 0 1.5-1.5v-2a.5.5 0 0 0-1 0v2z"/>
            <path fill-rule="evenodd" d="M15.854 8.354a.5.5 0 0 0 0-.708l-3-3a.5.5 0 0 0-.708.708L14.293 7.5H5.5a.5.5 0 0 0 0 1h8.793l-2.147 2.146a.5.5 0 0 0 .708.708l3-3z"/>
          </svg>
        </a>
        
      </div>
    </div>
    <div class="chat-messages" bind:this={chatContainer}>
      {#each messages as chatMessage, index (index)}
        {#if index === 0 || chatMessage.sender !== messages[index - 1].sender}
          <div class="message" key={index} role="button" tabindex="0" aria-label="Message Options" on:mouseenter={showHoverPanel} > 
            <div class="sender-avatar">
              {#if chatMessage.sender === 'System'}
              <img src={'https://cdn-icons-png.flaticon.com/512/305/305098.png'} alt="Sender Avatar" />
              {:else}
              <img src={generateProfilePicture(chatMessage.sender)} alt="Sender Avatar" />
              {/if}
            </div>
            <div class="message-content">
              <div class="sender-name">{chatMessage.sender}</div>
              <div class="message-text" style="white-space: pre-line; padding-top: 2px;position:relative; top:3px">{@html chatMessage.text.replace(/\n/g, "<br>")}</div>
            </div>
            {#if isHovered}
              <div class="hover-panel">
                <button on:click={() => handleReact(chatMessage.sender)}>React</button>
                <button on:click={() => handleReply(chatMessage.sender)}>Reply</button>
              </div>
            {/if}
          </div>
          
        {:else}
        
        <div style="padding: 3px; margin: 3px; padding-left: 6.55vh;" class="message" key={index}>
          <div class="message-text" style="white-space: pre-line;">{@html chatMessage.text.replace(/\n/g, "<br>")}</div>
          {#if isHovered}
              <div class="hover-panel">
                <button on:click={() => handleReact(chatMessage.sender)}>React</button>
                <button on:click={() => handleReply(chatMessage.sender)}>Reply</button>
              </div>
            {/if}
        </div>
        {/if}
      {/each}
    </div>
    <div class="input-container-container">
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
      padding-top: 9px;
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
      /* background-color: #313236; */
      display: flex;
      justify-content: space-between;
      padding: 10px;
      box-shadow: #36393f 0px 0px 100px 0.5px;
      background: transparent;
      backdrop-filter: blur(20px);
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
  background-color: rgba(0, 0, 0, 0.3);
  border-radius: 5px;
  display: flex;
  gap: 5px;
  padding: 5px;
  margin: 0;
  z-index: 1;
}

.hover-panel button {
  background-color: #40444b;
  border: none;
  color: white;
  padding: 2px 6px;
  border-radius: 3px;
  cursor: pointer;
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
  
  
  
  
  