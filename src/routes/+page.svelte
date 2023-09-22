<script lang="ts">
    import { onMount } from 'svelte';
    
    const localStorageKey = 'ipAddress';
    let ipAddress = '';
  
    onMount(() => {
      const savedIpAddress = localStorage.getItem(localStorageKey);
      if (savedIpAddress) {
        const useSavedIpAddress = confirm(`Would you like to use the saved IP address: ${savedIpAddress}?`);
        if (useSavedIpAddress) {
          ipAddress = savedIpAddress;
        }
      }
    });
  
    function saveIpAddress() {
      localStorage.setItem(localStorageKey, ipAddress);
      window.location.href = '/login';
    }
  </script>
  
  <main>
    <h1>Enter IP Address</h1>
    <input type="text" bind:value={ipAddress} />
    <button on:click={saveIpAddress}>Save</button>
  </main>