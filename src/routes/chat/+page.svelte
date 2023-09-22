<script lang="ts">
    import { onMount } from 'svelte';
  
    const localStorageKey = 'ipAddress';
    const wsUrl = `ws://${localStorage.getItem(localStorageKey)}:9001`;
    const token = localStorage.getItem('token');
    let messages: string[] = [];
    let message = '';
  
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
        messages = [...messages, message];
      }
    }
  
    function handleSend() {
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
  </script>
  
  <main>
    <h1>Chat</h1>
    <ul>
      {#each messages as message}
        <li>{message}</li>
      {/each}
    </ul>
    <input type="text" bind:value={message} on:keydown={event => event.key === 'Enter' && handleSend()} />
    <button on:click={handleLogout}>Logout</button>
  </main>