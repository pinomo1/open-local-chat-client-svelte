<script lang="ts">
  import { afterUpdate } from 'svelte';
  import { io } from 'socket.io-client';

  const localStorageKey = 'ipAddress';
  const url = localStorage.getItem(localStorageKey);
  const wsUrl = `ws://${url}:9001`;
  const token = localStorage.getItem('token');
  let messages: { username: string; message: string }[] = [];
  let message = '';
  let chatContainer: HTMLElement | null = null;

  let ws = io(wsUrl, { transports : ['websocket'] });
  ws.on("connect", handleOpen);
  ws.on("chat", handleMessage);
  ws.on("joined", handleJoin)
  ws.on("left", handleLeft)
  ws.on("error", handleError)

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
  }

  function handleLeft(username: string) {
    handleMessage("System", `${username} left the chat`);
  }

  function handleMessage(username: string, message: string) {
    let chatMessage = { username, message };
    if (messages.length > 1024) {
      messages = messages.slice(256);
    }
    messages = [...messages, chatMessage];
  }

  function handleSend() {
    ws.emit("chat", message);
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

  afterUpdate(scrollToBottom);
</script>
  
  <div class="chat-container">
    <div class="chat-header">
      <div class="chat-header-text">Global Chat - {url}</div>
      <div class="logout-btn">
        <a href="#top" on:click={handleLogout}>Logout</a>
      </div>
    </div>
    <div class="chat-messages" bind:this={chatContainer}>
      {#each messages as chatMessage, index (index)}
          <div class="message" key={index}>
            <div class="message-content">
              {#if index === 0 || chatMessage.username !== messages[index - 1].username}
                <div class="sender-name">{chatMessage.username}</div>
              {/if}
              <div class="message-text" style="white-space: pre-line;">{@html chatMessage.message.replace(/\n/g, "<br>")}</div>
            </div>
          </div>
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
      background-color: #383c40;
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
      margin-left: 0 !important;
      display: flex;
      align-items: center;
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
      padding-top: 12px;
      padding-bottom: 12px;
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
    }
  </style>
  
  
  
  
  