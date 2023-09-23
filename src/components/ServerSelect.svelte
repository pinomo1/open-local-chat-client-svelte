<script lang="ts">
    import { onMount } from 'svelte';
    import AnimatedButton from './AnimatedButton.svelte';
    
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

<div class="container">
    <div class="wrapper">
        <form action="">
            <h1>Enter IP Address</h1>
            <div class="input-box">
                <input type="text" placeholder="IP Address" bind:value={ipAddress} required/>
                <i class='bx bxs-lock-alt'></i>
            </div>
            
            <AnimatedButton onClick={saveIpAddress} text="Save" />
        </form>
    </div>
</div>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap');
    
    *{
        margin: 0;
        padding: 0;
        box-sizing: border-box;
        font-family: "Poppins", sans-serif;
    }
    
    .container{
        min-height: 100vh;
        background: #fff;
        display: flex;
        justify-content: center;
        align-items: center;
        background: url(https://e1.pxfuel.com/desktop-wallpaper/645/664/desktop-wallpaper-dark-gray-minimalist-minimalist-grey.jpg) no-repeat;
        background-size: cover;
        background-position: center;
    }
    
    .wrapper{
        color: white;
        background-color: transparent;
        backdrop-filter: blur(20px);
        box-shadow: 0 0 10px rgba(0,0,0, 0.2);
        border: 2px solid rgba(255, 255, 255, 0.2);
        width: 420px;
        border-radius: 10px;
        padding: 30px 40px;
    }
    
    .wrapper h1{
        font-size: 36px;
        font-weight: 600;
        text-align: center;
        padding: 30px 40px;
    }
    
    .wrapper .input-box{
        position: relative;
        width: 100%;
        height: 50px;
        margin: 30px 0;
    }
    
    .input-box input{
        width: 100%;
        height: 100%;
        border: none;
        background: transparent;
        border: 2px solid rgba(255, 255, 255, 2);
        border-radius: 40px;
        font-size: 16px;
        color: #fff;
        padding: 20px 45px 20px 20px;
    }
    
    .input-box input::placeholder{
        color: #fff;
    }
    
    .input-box i{
        position: absolute;
        right: 20px;
        top: 50%;
        transform: translateY(-50%);
        font-size: 20px;
    }
    
    .wrapper .btn{
        width: 100%;
        height: 45px;
        border: none;
        outline: none;
        border-radius: 40px;
        font-size: 16px;
        font-weight: 600;
        cursor: pointer;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        background-color: #fff;
        color: #333;
    }
</style>