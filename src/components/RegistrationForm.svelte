<script lang=ts>
    import { onMount } from 'svelte';
	import AnimatedButton from "./AnimatedButton.svelte";
    import axios from 'axios';
  
    const localStorageKey = 'ipAddress';
    const url = localStorage.getItem(localStorageKey);
    const wsUrl = `http://${url}:9001/api/register`;
    let username = '';
    let password = '';
    let confirmPassword = '';

    function getCheckUrl() {
        return 'http://' + url + ':9001/api/canaccess';
    }

    onMount(() => {
        let siteUrl = getCheckUrl();
        axios.post(siteUrl, {})
        .then((response) => {
            console.log(response);
            if (response.status !== 200) {
                window.location.href = '/';
                return;
            }
        })
        .catch((error) => {
            window.location.href = '/';
        });
    });

    function handleRegister() {
        if (password !== confirmPassword) {
        console.error('Error: Passwords do not match');
        return;
        }
        axios.post(wsUrl, {
            username: username,
            password: password
        })
        .then((response) => {
            console.log(response);
            if (response.status !== 200) {
                console.log(response.data.error)
                return;
            }
            window.location.href = '/login';
        })
        .catch((error) => {
            console.log(error);
        });
    }

    function handleLogin() {
        window.location.href = '/login';
    }

    function handleServerChange() {
        window.location.href = '/';
    }
</script>
<div class="container">
    <div class="wrapper">
        <form action="">
            <h1>Sign up</h1>
            <div class="input-box">
                <input type="text" placeholder="Username" bind:value={username} required/>
                <i class='bx bxs-user'></i>
            </div>
    
            <div class="input-box">
                <input type="password" placeholder="Password" bind:value={password} required/>
                <i class='bx bxs-lock-alt' ></i>
            </div>

            <div class="input-box">
                <input type="password" placeholder="Confirm password" bind:value={confirmPassword} required/>
                <i class='bx bxs-lock-alt' ></i>
            </div>
            
            <AnimatedButton onClick={handleRegister} text="Register" />
                
            <div class="register-link">
                <p>Already have an account? <a href="#top" on:click={handleLogin}>Login</a></p>
                <br>
                <p>Server: {url}</p>
                <p><a href="#top" on:click={handleServerChange}>Change server</a></p>
            </div>
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

    .wrapper .register-link{
        text-align: center;
        margin: 20px 0 15px;    
        font-size: 14.5px;
    }

    .register-link p a{
        color: #fff;
        text-decoration: none;
        font-weight: 600;
    }

    .register-link p a:hover{
        text-decoration: underline;
    }
</style>