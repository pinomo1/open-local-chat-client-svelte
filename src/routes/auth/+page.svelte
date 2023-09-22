<script lang='ts'>
    import RegistrationForm from '../../components/RegistrationForm.svelte';
    import LoginForm from '../../components/LoginForm.svelte';
    import ChatPage from '../../components/ChatPage.svelte';

    let mode = 'login';


    function toggleMode() {
        mode = mode === 'login' ? 'register' : 'login';
    }

</script>

{#if mode === 'register'}

    <RegistrationForm username="test" onClick={toggleMode} />
<!-- {:else if mode === 'chat'}
    <ChatPage />
    -->
{:else}
    <LoginForm username="test" onClick={toggleMode}/>
{/if}