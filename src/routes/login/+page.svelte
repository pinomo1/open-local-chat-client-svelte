<script lang="ts">
    import { onMount } from 'svelte';
  
    const localStorageKey = 'ipAddress';
    const wsUrl = `ws://${localStorage.getItem(localStorageKey)}:9001`;
    let username = '';
    let password = '';
  
    let ws: WebSocket;
    let token: string;
  
    onMount(() => {
      ws = new WebSocket(wsUrl);
      ws.addEventListener('message', handleMessage);
    });
  
    function handleMessage(event: MessageEvent) {
      const message = event.data;
      if (message.startsWith('0')) {
        const errorCode = message.slice(2);
        alert(`Error: ${errorCode}`);
      } else if (message.startsWith('1')) {
        token = message.slice(2);
        localStorage.setItem('token', token);
        window.location.href = '/chat';
      }
    }
  
    function handleLogin() {
      const message = `login ${username} ${password}`;
      ws.send(message);
    }
  
    function handleRegister() {
      window.location.href = '/register';
    }
  </script>
  
  <main>
    <h1>Login</h1>
    <label>
      Username:
      <input type="text" bind:value={username} />
    </label>
    <label>
      Password:
      <input type="password" bind:value={password} />
    </label>
    <button on:click={handleLogin}>Login</button>
    <p>Don't have an account? <a href="#top" on:click={handleRegister}>Register</a></p>
  </main>