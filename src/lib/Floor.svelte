<script lang="ts">
  import { onMount } from "svelte";
  import { token } from "./store";

  const apiUrl = "https://api.spotify.com/v1";

  let loaded = false;
  let items = [];
  let nextCall: string;
  let checking = false;

  const headers = {
    "Content-Type": "application/json",
    Authorization: "Bearer " + $token,
  };

  async function checkScroll() {
    if (!checking) {
      // @ts-ignore
      if (window.scrollY >= window.scrollMaxY - 200) {
        checking = true;
        const res = await fetch(nextCall, {
          headers,
        });
        const json = await res.json();
        items = [...items, ...json.items];
        nextCall = json.next;
        checking = false;
      }
    }
  }

  onMount(async () => {
    const res = await fetch(apiUrl + "/me/tracks", {
      headers,
    });

    if (res.status === 401) {
      localStorage.removeItem("token");
      location.reload();
    }

    const json = await res.json();
    items = json.items;
    nextCall = json.next;
    loaded = true;

    checkScroll();
    window.addEventListener("scroll", checkScroll);
  });

  let audios = {};

  function playAudio(url) {
    if (url) {
      if (audios[url]) {
        audios[url].play();
      } else {
        audios[url] = new Audio(url);
        audios[url].play();
      }
    }
  }
</script>

{#if !loaded}
  <p>loading...</p>
{:else}
  <div class="wrapper">
    <div class="tracks">
      {#each items as item}
        <a
          on:mouseover={() => {
            playAudio(item.track.preview_url);
          }}
          on:focus={() => {
            playAudio(item.track.preview_url);
          }}
          on:mouseout={() => {
            if (audios[item.track.preview_url]) {
              audios[item.track.preview_url].pause();
            }
          }}
          on:blur={() => {
            if (audios[item.track.preview_url]) {
              audios[item.track.preview_url].pause();
            }
          }}
          href={item.track.external_urls.spotify}
          target="_blank"
        >
          <div class="track">
            <div class="info">
              <p class="title">
                {item.track.name}
              </p>
              <p class="artists">
                {item.track.artists.map((a) => a.name).join(", ")}
              </p>
            </div>
            <img
              src={item.track.album.images[0].url}
              width="150px"
              alt={item.track.name}
            />
          </div>
        </a>
      {/each}
    </div>
  </div>
{/if}

<style lang="scss">
  .wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: calc(100vh);
  }

  .tracks {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-wrap: wrap;
    max-width: 1000px;
    gap: 10px;
    padding: 20px;
  }

  .track {
    transition: all 0.2s cubic-bezier(0.86, 0, 0.07, 1);
    transform: rotate3d(1, -1, 0, 30deg);
    position: relative;
    cursor: pointer;

    img {
      box-shadow: -20px 20px 5px black;
      border-radius: 5px;
    }

    .info {
      opacity: 0;
      position: absolute;
      width: calc(100% - 20px);
      height: calc(100% - 20px);
      padding: 10px;
      background-color: rgba(0, 0, 0, 0.7);
      transition: opacity 0.2s ease-in-out;
      pointer-events: none;

      .artists {
        position: absolute;
        bottom: 10px;
        right: 10px;
        text-align: right;
      }
    }

    &:hover {
      z-index: 999;
      scale: 1.2;
      transform: rotate3d(0, 0, 0, 0deg);

      .info {
        opacity: 1;
      }
    }
  }
</style>
