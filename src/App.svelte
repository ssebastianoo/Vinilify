<script lang="ts">
  import { onMount } from "svelte";
  import { token } from "./lib/store";
  import Floor from "./lib/Floor.svelte";

  function makeID(length) {
    let result = "";
    let characters =
      "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
    let charactersLength = characters.length;
    for (let i = 0; i < length; i++) {
      result += characters.charAt(Math.floor(Math.random() * charactersLength));
    }
    return result;
  }

  let loginUrl = "https://accounts.spotify.com/authorize";
  let logged = false;

  onMount(() => {
    let urlParams = new URLSearchParams(window.location.hash.replace("#", "?"));
    $token = urlParams.get("access_token");

    if ($token) {
      localStorage.setItem("token", $token);
      location.href =
        location.protocol + "//" + location.host + location.pathname;
      logged = true;
    } else {
      if (localStorage.getItem("token")) {
        $token = localStorage.getItem("token");
        logged = true;
      } else {
        let client_id = "b11fe8fc0cb943c489d4e24202a3fb67";
        let redirect_uri =
          location.protocol + "//" + location.host + location.pathname;
        let scope = "user-library-read";
        let state = makeID(16);
        loginUrl += "?response_type=token";
        loginUrl += "&client_id=" + encodeURIComponent(client_id);
        loginUrl += "&scope=" + encodeURIComponent(scope);
        loginUrl += "&redirect_uri=" + encodeURIComponent(redirect_uri);
        loginUrl += "&state=" + encodeURIComponent(state);
      }
    }
  });

  function login() {
    localStorage.setItem("shouldBeLogged", "true");
    location.href = loginUrl;
  }
</script>

{#if !logged}
  <div class="login">
    <button class="button" on:click={login}>Login</button>
  </div>
{:else}
  <Floor />
{/if}

<style lang="scss">
  .login {
    width: 100%;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
  }
</style>
