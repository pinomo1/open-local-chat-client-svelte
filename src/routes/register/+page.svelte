<script lang="ts">
    import { onMount } from 'svelte';
  
    const localStorageKey = 'ipAddress';
    const wsUrl = `ws://${localStorage.getItem(localStorageKey)}:9001`;
    let username = '';
    let password = '';
    let confirmPassword = '';
  
    let ws: WebSocket;
  
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
        window.location.href = '/login';
      }
    }
  
    function handleRegister() {
      if (password !== confirmPassword) {
        console.error('Error: Passwords do not match');
        return;
      }
      const message = `register ${username} ${password}`;
      ws.send(message);
    }

    function handleLogin() {
      window.location.href = '/login';
    }
  </script>
  
  <main>
    <h1>Register</h1>
    <label>
      Username:
      <input type="text" bind:value={username} />
    </label>
    <label>
      Password:
      <input type="password" bind:value={password} />
    </label>
    <label>
      Confirm Password:
      <input type="password" bind:value={confirmPassword} />
    </label>
    <button on:click={handleRegister}>Register</button>
    <p>Already have an account? <a href="#top" on:click={handleLogin}>Login</a></p>
  </main>