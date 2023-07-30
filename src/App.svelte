<script lang="ts">
  import { onDestroy, onMount } from "svelte";
  import Swal from "sweetalert2";
  import type { ClientList, UserMessage } from "./lib/types";
  import DescriptionUserContainer from "./lib/DescriptionUserContainer.svelte";

  let sendText: HTMLTextAreaElement;
  let joinButton: HTMLButtonElement;
  let sideMenu: HTMLDivElement;
  let expandIcon: HTMLImageElement;

  let animateSlide = false;

  let username: string = "";
  let messages: UserMessage[] = [];
  let users: string[] = [];
  let width: number;
  let areButtonsDisabled = false;

  let connected = false;
  let ws: WebSocket | null = null;

  const onPressEnter = (ev: KeyboardEvent) => {
    if ((ev.code === "Enter" || ev.code === "NumpadEnter") && !ev.shiftKey) {
      if (joinButton && username) {
        joinButton.click();
        return;
      }

      if (sendText && ws) {
        ev.preventDefault();
        onPressSend();
        return;
      }
    }
  };

  const toggleMenu = () => {
    if (sideMenu) {
      animateSlide = !animateSlide;
    }

    if (expandIcon) {
      expandIcon.classList.add("animateFade");
      setTimeout(() => {
        expandIcon.classList.remove("animateFade");
      }, 1250);
    }
  };

  const onPressLeave = () => {
    if (ws && !areButtonsDisabled) {
      areButtonsDisabled = true;
      ws.close();
    }
  };

  const onPressSend = () => {
    if (!sendText?.value) return;

    const m: UserMessage = { username, message: sendText.value };
    if (m && m.message.length <= 256) {
      ws.send(JSON.stringify(m));
      sendText.value = "";
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
    if (username.length < 3 || username.length > 15) {
      Swal.fire({
        title: "Error",
        text: "Username must be between 3 and 15 characters.",
        icon: "error",
        confirmButtonText: "Close",
        allowEnterKey: true,
        allowEscapeKey: true,
      });
      username = "";
      return;
    }

    if ("WebSocket" in window || "MozWebSocket" in window) {
      ws = new WebSocket(`wss://chatroom-backend-production.up.railway.app/ws?username=${username}`);
      ws.onopen = () => {
        connected = true;
      };

      ws.onclose = () => {
        connected = false;
        username = "";
        messages = [];
        users = [];
        areButtonsDisabled = false;
      };

      ws.onerror = () => {
        connected = false;
        username = "";
        messages = [];
        users = [];
        areButtonsDisabled = false;
        Swal.fire({
          title: "Error",
          text: "Username is already in use, please try another name.",
          icon: "error",
          confirmButtonText: "Close",
          allowEnterKey: true,
          allowEscapeKey: true,
        });
      };

      ws.onmessage = (event) => {
        const parsedData = JSON.parse(event.data);

        if (parsedData?.username) {
          const message = parsedData as UserMessage;
          if (messages.length >= 50) messages = [message, ...messages.slice(0, -1)];
          else messages = [message, ...messages];
        } else {
          const data = parsedData as ClientList;
          users = [...data.user_list];
        }
      };
    } else {
      Swal.fire({
        title: "Error",
        text: "Your browser does not support websockets!",
        icon: "error",
        confirmButtonText: "Close",
        allowEnterKey: true,
        allowEscapeKey: true,
      });
    }
  };
</script>

<svelte:window bind:innerWidth={width} />

<div class="mainContainer">
  <div class="header">
    <a href="https://github.com/Jasonsd19/chatroom-backend" target="_blank" rel="noreferrer noopener">
      <img class="githubLink" src="github.svg" alt="Check out the code on GitHub" />
    </a>
    <div class="title">Conversin</div>
    <a href="https://jasondeol.com/" target="_blank" rel="noreferrer noopener">
      <img class="websiteLink" src="website.svg" alt="Check out my portfolio website" />
    </a>
  </div>
  {#if connected}
    {#if width <= 600}
      <img
        class="expandIcon"
        src="expand.svg"
        alt="Expand/Close user menu"
        on:click={toggleMenu}
        bind:this={expandIcon}
      />
      <div
        class="activityContainer"
        class:slideMenu={animateSlide}
        class:closeMenu={!animateSlide}
        bind:this={sideMenu}
      >
        <DescriptionUserContainer {users} />
      </div>
      <div class="chatroomContainer">
        <div class="messageContainer">
          {#if messages.length}
            {#each messages as message, i (i)}
              <div class="message">
                {`${message.username}:\n${message.message}`}
              </div>
            {/each}
          {:else}
            <div class="message">
              {"No messages yet!"}
            </div>
          {/if}
        </div>
        <div class="textContainer">
          <img class="leaveButton" src="leave.svg" alt="Leave chatroom" on:click={onPressLeave} />
          <textarea maxlength={200} bind:this={sendText} />
          <button on:click={onPressSend} disabled={areButtonsDisabled}>Send</button>
        </div>
      </div>
    {:else}
      <div class="activityContainer">
        <DescriptionUserContainer {users} />
        <div class="chatroomContainer">
          <div class="messageContainer">
            {#if messages.length}
              {#each messages as message, i (i)}
                <div class="message">
                  {`${message.username}:\n${message.message}`}
                </div>
              {/each}
            {:else}
              <div class="message">
                {"No messages yet!"}
              </div>
            {/if}
          </div>
          <div class="textContainer">
            <img class="leaveButton" src="leave.svg" alt="Leave chatroom" on:click={onPressLeave} />
            <textarea maxlength={200} bind:this={sendText} />
            <button on:click={onPressSend} disabled={areButtonsDisabled}>Send</button>
          </div>
        </div>
      </div>
    {/if}
  {:else}
    <div class="joinContainer">
      <div class="usernameContainer">
        <label for="username">Username </label>
        <input inputmode="text" maxlength={15} value={username} on:input={onChangeUsername} />
      </div>
      <div class="joinButtonContainer">
        <button on:click={connect} bind:this={joinButton}>Join Chatroom!</button>
      </div>
    </div>
  {/if}
</div>

<style>
  .mainContainer {
    position: relative;
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

  .activityContainer {
    position: absolute;
    display: flex;
    top: 8vh;
    height: 91.5vh;
    width: 100vw;
    transition: transform 1s;
  }

  .expandIcon {
    position: absolute;
    top: 9vh;
    right: 2vw;
    width: 30px;
    cursor: pointer;
    z-index: 1;
  }

  .chatroomContainer {
    display: flex;
    height: 90vh;
    width: 100%;
    flex-wrap: wrap;
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
    white-space: pre-wrap;
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
    height: auto;
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
    color: white;
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

    .activityContainer {
      position: static;
      display: flex;
      height: 90.5vh;
      width: 100vw;
      transform: none;
    }

    .chatroomContainer {
      height: 85vh;
      width: 60%;
      padding-right: 5px;
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
      width: 65%;
      resize: none;
      font-size: 15px;
    }

    .textContainer button {
      width: 15%;
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

  @media only screen and (min-width: 900px) {
    .leaveButton {
      width: 50px;
    }

    .textContainer textarea {
      height: 100%;
      width: 70%;
      resize: none;
    }

    .textContainer button {
      width: 12.5%;
      background-color: white;
      color: black;
      padding: 0;
    }
  }

  @media only screen and (min-width: 1100px) {
    .leaveButton {
      width: 52.5px;
    }

    .textContainer textarea {
      height: 100%;
      width: 75%;
      resize: none;
    }

    .textContainer button {
      width: 12.5%;
      background-color: white;
      color: black;
      padding: 0;
    }
  }

  @media only screen and (min-width: 1500px) {
    .leaveButton {
      width: 52.5px;
    }

    .textContainer textarea {
      height: 100%;
      width: 75%;
      resize: none;
    }

    .textContainer button {
      width: 12.5%;
      background-color: white;
      color: black;
      padding: 0;
    }
  }

  .slideMenu {
    transform: translateX(0%);
  }

  .closeMenu {
    transform: translateX(100%);
  }

  :global(.animateFade) {
    animation: fadeInOut 1.25s;
  }

  @keyframes fadeInOut {
    0% {
      opacity: 1;
    }

    10% {
      opacity: 0;
    }

    90% {
      opacity: 0;
    }

    100% {
      opacity: 1;
    }
  }
</style>
