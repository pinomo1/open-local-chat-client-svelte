<script lang="ts">
    import { onMount } from 'svelte';
    import AnimatedButton from './AnimatedButton.svelte';
    import axios from 'axios';
    
    const localStorageKey = 'ipAddress';
    let ipAddress = '';
    
  
    function showPopup() {
      const popup: HTMLElement = document.querySelector('.popup')!;
      console.log("show");
      popup.classList.remove('hidden');
    }

    function hidePopup() {
      const popup: HTMLElement = document.querySelector('.popup')!;
      console.log("hide");
      popup.classList.add('hidden');
    }

    function getSiteUrl() {
      return window.location.hostname;
    }

    function getCheckUrl(url?: string) {
        if (url) {
            return 'http://' + url + ':9001/api/canaccess';
        }
      return 'http://' + getSiteUrl() + ':9001/api/canaccess';
    }

    onMount(() => {
      ipAddress = localStorage.getItem(localStorageKey) || '';
      let siteUrl = getCheckUrl();
      axios.post(siteUrl, {})
        .then((response) => {
            if (response.status === 200) {
                showPopup();
            }
        })
        .catch((error) => {
            console.log(error);
        });
    });
  
    function useLocal(){
        ipAddress = getSiteUrl();
        saveIpAddress();
    }

    function saveIpAddress() {
      let siteUrl = getCheckUrl(ipAddress);
        axios.post(siteUrl, {})
            .then((response) => {
                console.log(response);
                if (response.status === 200) {
                    localStorage.setItem(localStorageKey, ipAddress);
                    window.location.href = '/login';
                }
            })
            .catch((error) => {
                console.log(error);
            });
    }
</script>

<div class="container">
    <div class="popup hidden">
        <div class="wrapper">
            <form action="">
                <h1>There is a server on this IP address, connect?</h1>
                <AnimatedButton onClick={useLocal} text="Yes" />
                <AnimatedButton onClick={hidePopup} text="No" />
            </form>
        </div>
    </div>
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
    
    .popup{
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100vh;
        background: rgba(0,0,0,0.5);
        display: flex;
        justify-content: center;
        align-items: center;
        z-index: 1000;
    }

    .popup.hidden{
        display: none;
    }

    .container{
        min-height: 100vh;
        background: #fff;
        display: flex;
        justify-content: center;
        align-items: center;
        background: url(images/bg.jpg) no-repeat;
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