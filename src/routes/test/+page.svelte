<script lang='ts'>
    import RegistrationForm from '../../components/RegistrationForm.svelte';
    import LoginForm from '../../components/LoginForm.svelte';

    let mode = 'register';


    function toggleMode() {
        mode = mode === 'login' ? 'register' : 'login';
    }

</script>

{#if mode === 'register'}
    <RegistrationForm username="test" onClick={toggleMode} />
{:else}
    <LoginForm username="test" onClick={toggleMode}/>
{/if}