<script lang="ts">
  import { onMount } from 'svelte';
  let ws : WebSocket;
  let username : string;
  let password : string;
  let messages : string[] = [];
  let message = '';
  let token = '';
  let lastAction = '';
  let isDarkMode = false;
  const urlAddress = "ws://192.168.54.98:9001"

  function register() {
    if (!username.match(/^[a-zA-Z0-9-_]+$/)) {
      alert('Username can only contain alphanumeric characters, "-" and "_"');
      return;
    }
    if (!password.match(/^[a-zA-Z0-9-_]+$/)) {
      alert('Username can only contain alphanumeric characters, "-" and "_"');
      return;
    }
    ws.send(`register ${username} ${password}`);
    lastAction = 'register';
  }

  function join() {
    if (!username.match(/^[a-zA-Z0-9-_]+$/)) {
      alert('Username can only contain alphanumeric characters, "-" and "_"');
      return;
    }
    if (!password.match(/^[a-zA-Z0-9-_]+$/)) {
      alert('Username can only contain alphanumeric characters, "-" and "_"');
      return;
    }
    ws.send(`join ${username} ${password}`);
    lastAction = 'join';
  }

  function sendMessage() {
    if (message === '') {
      return;
    }
    if (message.length > 1024) {
      alert('Message is too long');
      return;
    }
    ws.send(`chat ${username} ${token} ${message}`);
    message = '';
    lastAction = 'chat';
  }

  function handleKeyDown(event: KeyboardEvent) {
    if (token === '') {
      return;
    }
    if (event.key === "Enter") {
      sendMessage();
    }
  }

  onMount(() => {
    const cookie = document.cookie.split(';').find((cookie) => cookie.trim().startsWith('isDarkMode='));
    if (cookie) {
      isDarkMode = cookie.split('=')[1] === 'true';
    }
    ws = new WebSocket(urlAddress);
    ws.onmessage = (event) => {
      if (lastAction === 'register') {
        if (event.data === '1') {
          alert('Registered successfully, please join');
        } else {
          alert(event.data);
        }
      }
      else if (lastAction === 'join') {
        if (event.data[0] === '1') {
          token = event.data.split(' ')[1];
          alert('Joined successfully');
        } else {
          alert(event.data);
        }
      }
      else{
        const chatMessages = document.querySelector('.chat-messages');
        const isScrolledToBottom = chatMessages!.scrollHeight - chatMessages!.clientHeight <= chatMessages!.scrollTop + 10;
        messages = [...messages, event.data];
        if (isScrolledToBottom) {
          chatMessages!.scrollTop = chatMessages!.scrollHeight - chatMessages!.clientHeight;
        }
      }
      lastAction = '';
    };
  });
  
  function toggleDarkMode() {
    isDarkMode = !isDarkMode;
    document.cookie = `isDarkMode=${isDarkMode}`;
  }
</script>

<div class="wrapper" class:dark-mode={isDarkMode}>
  <div class="chat-window">
    <button class="dark-mode-button" on:click={toggleDarkMode}>Dark Mode</button>
    {#if !token}
      <h2>Register</h2>
      <input type="text" bind:value={username} placeholder="Username">
      <input type="password" bind:value={password} placeholder="Password">
      <button on:click={register}>Register</button>
      <h2>Join</h2>
      <input type="text" bind:value={username} placeholder="Username">
      <input type="password" bind:value={password} placeholder="Password">
      <button on:click={join}>Join</button>
    {:else}
      <div class="chat-messages">
        <ul>
          {#each messages as message}
            <div class="message">{message}</div>
          {/each}
        </ul>
      </div>
      <div>
        <input type="text" class="big-input" bind:value={message} placeholder="Message" on:keydown={handleKeyDown}>
        <button on:click={sendMessage}>Send</button>
      </div>
    {/if}
  </div>
</div>

<style>
  .wrapper {
    font-family: sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 90vh;
    margin: 0;
    padding-bottom: 4rem;
  }

  .wrapper.dark-mode {
    background-color: #333;
    color: #fff;
  }

  .wrapper.dark-mode .message {
    background-color: #444;
  }

  .chat-window {
    display: flex;
    flex-direction: column;
    height: 100%;
    max-width: 800px;
    width: 100%;
  }

  .chat-messages {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    justify-content: flex-start;
    overflow-y: auto;
    padding: 1rem;
  }

  .message {
    margin-bottom: 0.5rem;
    padding: 0.5rem;
    background-color: #f5f5f5;
    border-radius: 0.5rem;
    text-align: left;
    max-width: 80%;
  }

  ul {
    list-style: none;
    padding: 0;
    margin: 0;
    width: 100%;
  }

  /* li {
    margin-bottom: 0.5rem;
    padding: 0.5rem;
    background-color: #f5f5f5;
    border-radius: 0.5rem;
    text-align: center;
  } */

  input[type="text"], input[type="password"] {
    padding: 0.5rem;
    border-radius: 0.5rem;
    border: 1px solid #ced4da;
    margin-right: 0.5rem;
  }

  button {
    padding: 0.5rem;
    border-radius: 0.5rem;
    border: none;
    background-color: #007bff;
    color: #fff;
    cursor: pointer;
  }

  .big-input {
    width: 100%;
  }

  .dark-mode-button {
    position: absolute;
    top: 1rem;
    right: 1rem;
    padding: 0.5rem;
    border-radius: 0.5rem;
    border: none;
    background-color: #007bff;
    color: #fff;
    cursor: pointer;
  }
  .dark-mode-button:hover {
    background-color: #0069d9;
  }
</style>