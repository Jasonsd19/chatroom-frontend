<script lang="ts">
  import { onDestroy, onMount } from "svelte";
  import Swal from "sweetalert2";

  interface UserMessage {
    username: string;
    message: string;
  }

  let username: string = "";
  let sendText: HTMLTextAreaElement;
  let messages: UserMessage[] = [];

  let connected = false;
  let ws: WebSocket | null = null;

  const onPressEnter = (ev: KeyboardEvent) => {
    if (ev.code === "Enter" && !ev.shiftKey && ws) {
      ev.preventDefault();
      const m: UserMessage = { username, message: sendText.value };
      if (m && m.message.length <= 256) {
        ws.send(JSON.stringify(m));
        sendText.value = "";
      }
    }
  };

  const onChangeUsername = (e: Event) => {
    const event = e.target as HTMLInputElement;
    username = event.value;
  };

  onMount(() => {
    window.addEventListener("keypress", onPressEnter);
    return () => window.removeEventListener("keypress", onPressEnter);
  });

  onDestroy(() => {
    if (ws && ws.readyState === ws.OPEN) ws.close();
  });

  const connect = () => {
    if (username.length <= 8) {
      Swal.fire({
        title: "Error",
        text: "Username must be at least 8 characters long.",
        icon: "error",
        confirmButtonText: "Close",
      });
      return;
    }

    if ("WebSocket" in window || "MozWebSocket" in window) {
      ws = new WebSocket("wss://chatroom-backend-production.up.railway.app/ws");
      ws.onopen = () => {
        connected = true;
      };

      ws.onclose = () => {
        connected = false;
      };

      ws.onerror = () => {
        connected = false;
        Swal.fire({
          title: "Error",
          text: "We ran into an unexpected error, please refresh the page and try again.",
          icon: "error",
          confirmButtonText: "Close",
        });
      };

      ws.onmessage = (event) => {
        const message = JSON.parse(event.data) as UserMessage;
        if (messages.length >= 50) messages = [message, ...messages.slice(0, -1)];
        else messages = [message, ...messages];
      };
    } else {
      Swal.fire({
        title: "Error",
        text: "Your browser does not support websockets!",
        icon: "error",
        confirmButtonText: "Close",
      });
    }
  };
</script>

<div class="mainContainer">
  {#if connected}
    <div class="chatroomContainer">
      <div class="messageContainer">
        {#if messages.length}
          {#each messages as message, i (i)}
            <div class="message">
              {`${message.username}: ${message.message}`}
            </div>
          {/each}
        {:else}
          <div class="message">
            {"No messages yet!"}
          </div>
        {/if}
      </div>
      <div class="textContainer">
        <textarea maxlength={200} bind:this={sendText} />
      </div>
    </div>
  {:else}
    <div class="usernameContainer">
      <label for="username">Username: </label>
      <input inputmode="text" on:input={onChangeUsername} />
    </div>
    <div class="joinButtonContainer">
      <button on:click={connect}>Join Chatroom!</button>
    </div>
  {/if}
</div>

<style>
  .mainContainer {
    width: 100vw;
    height: 100vh;
    overflow-x: hidden;
    overflow-y: hidden;
  }

  .header {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
    align-content: center;
    font-size: 8vw;
    height: 8vh;
  }

  .chatroomContainer {
    display: flex;
    height: 90vh;
    width: 100%;
    flex-wrap: wrap;
    justify-content: center;
  }

  .messageContainer {
    display: flex;
    flex-direction: column-reverse;
    height: 90%;
    width: 100%;
    overflow-x: hidden;
    overflow-y: visible;
    border: 5px solid black;
    border-radius: 15px;
  }

  .message {
    display: flex;
    flex-wrap: wrap;
    width: 100%;
    justify-content: left;
    padding: 1vh;
  }

  .textContainer {
    display: flex;
    flex-wrap: wrap;
    align-content: center;
    justify-content: space-evenly;
    height: 8%;
    width: 100%;
    padding-top: 1vh;
  }

  .leaveButton {
    width: 45px;
    cursor: pointer;
  }

  .textContainer textarea {
    height: 100%;
    width: 60%;
    resize: none;
  }

  .textContainer button {
    width: 20%;
    background-color: white;
    color: black;
    padding: 0;
  }

  .joinContainer {
    display: flex;
    flex-direction: column;
    flex-wrap: wrap;
    height: 80vh;
    width: 100vw;
    align-content: center;
    justify-content: center;
  }

  .usernameContainer {
    display: flex;
    flex-direction: column;
    width: 50%;
    height: 10%;
    font-size: 7.5vw;
    padding: 10vw;
    align-items: center;
  }

  .usernameContainer label {
    margin-bottom: 2vw;
  }

  .usernameContainer input {
    height: 100%;
    width: 100%;
    font-size: 5vw;
    text-align: center;
  }

  .joinButtonContainer {
    display: flex;
    flex-wrap: wrap;
    align-content: center;
    justify-content: center;
    padding: 10vw;
  }

  .joinButtonContainer button {
    width: 33vw;
    font-size: 3vw;
  }

  .githubLink {
    width: 7vw;
  }

  .websiteLink {
    width: 7vw;
  }

  @media only screen and (min-width: 600px) {
    .header {
      height: auto;
      font-size: 40px;
    }

    .chatroomContainer {
      height: 85vh;
    }

    .messageContainer {
      width: 80vw;
    }

    .textContainer {
      padding: 10px;
      width: 80vw;
    }

    .leaveButton {
      width: 50px;
      cursor: pointer;
    }

    .textContainer textarea {
      height: 100%;
      width: 60%;
      resize: none;
      font-size: 1.25vw;
    }

    .textContainer button {
      width: 20%;
      background-color: white;
      color: black;
      padding: 0;
      font-size: 2vw;
    }

    .usernameContainer {
      width: 200px;
      font-size: 30px;
      padding: 85px;
    }

    .usernameContainer label {
      margin-bottom: 25px;
    }

    .usernameContainer input {
      font-size: 25px;
    }

    .joinButtonContainer {
      padding: 85px;
    }

    .joinButtonContainer button {
      width: auto;
      font-size: inherit;
    }

    .githubLink {
      width: 42.5px;
    }

    .websiteLink {
      width: 42.5px;
    }
  }

  @media only screen and (min-width: 800px) {
    .messageContainer {
      width: 70vw;
    }

    .textContainer {
      width: 70vw;
    }
  }

  @media only screen and (min-width: 1000px) {
    .messageContainer {
      width: 60vw;
    }

    .textContainer {
      width: 60vw;
    }
  }

  @media only screen and (min-width: 1200px) {
    .chatroomContainer {
      flex-direction: column;
      align-content: center;
      height: 90vh;
    }

    .messageContainer {
      height: 85%;
      width: 40vw;
    }

    .textContainer {
      width: 40vw;
    }
  }
</style>
