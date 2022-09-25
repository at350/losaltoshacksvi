<script>
    import { goto } from "$app/navigation";
    import VanillaTilt from "vanilla-tilt";
    import { onMount } from "svelte";
    import { getAuth, onAuthStateChanged } from "firebase/auth";

    onMount(() => {
        typeof document !== 'undefined'; 
        const paragraph = document.querySelector(".description");
        VanillaTilt.init(paragraph)
    });


    function login() {
        checkLoggedIn('/login');
    }

    function signup() {
        checkLoggedIn('/signup');
    }

    function checkLoggedIn(inputPath) {
        let uid = localStorage.getItem("uid");
        if (uid)
            window.location.href = '/home'
        else
            window.location.href = inputPath;
    }
</script>

<nav class="navbar navbar-light" style="background-color: rgba(255, 141, 141, 0.9);">
    <div class="container-fluid">
      <a class="navbar-brand" href="/"> </a>
      <div class="d-flex" >
        <button class="nav-link m-3 textButtons" on:click={login}>Login</button>
        <button class="nav-link m-3 textButtons" on:click={signup}>Signup</button>
      </div>
    </div>
  </nav>


  
  <div class="header">
      <h1>Communicar</h1>
      <div class="description">
            <p> 
                Find people in your area to carpool with. Save fuel and the environment!<br>
                Help your community out by giving a lift to your coworkers without a ride! <br>
                Please log in or sign up for an account to access our services.
            </p>
    </div>
</div>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;700&display=swap');

    :global(body) {
        background: url("https://c4.wallpaperflare.com/wallpaper/542/270/492/abstract-blue-simple-wallpaper-preview.jpg") no-repeat center center fixed; 
        -webkit-background-size: cover;
        -moz-background-size: cover;
        -o-background-size: cover;
    }

    .header h1 {
        margin-top: 100px;
        margin: 6% auto;
        font-family: "Open Sans", sans-serif;
        font-weight:bold;
        background: -webkit-linear-gradient(bottom right,rgb(255, 23, 23), rgb(255, 199, 199));
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        font-size: 5rem;
        text-align: center;
    }

    .textButtons {
        border: none;
        background-color: inherit;
        font-size: 16px;
        cursor: pointer;
        display: inline-block;
    }
    
    .header p {
        font-family: "Open Sans", sans-serif;
        font-size: 24px;
        color: black;
    }

    .header {
        margin-top: 100px; 
        display: flex;
        flex-direction: column;
        width: 100%;
    }
    
    .header .description { 
        margin-top: 100px;
        background-color: rgba(256,256,256,0.5);
        margin: 0 auto;
        padding: 15px;
        border-radius: 15px;
        justify-content: center;
    }
    
</style>