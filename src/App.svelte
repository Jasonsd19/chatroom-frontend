<script lang="ts">
  import { onDestroy } from "svelte";
  import Swal from "sweetalert2";
  import type { ClientList, UserMessage } from "./lib/types";
  import DescriptionUserContainer from "./lib/DescriptionUserContainer.svelte";

  let sendText: HTMLTextAreaElement;
  let joinButton: HTMLButtonElement;

  let username = "";
  let messages: UserMessage[] = [];
  let users: string[] = [];
  let areButtonsDisabled = false;
  let isSidebarOpen = false;

  let connected = false;
  let ws: WebSocket | null = null;

  const websocketBaseUrl =
    import.meta.env.VITE_WEBSOCKET_URL ?? "wss://chatroom-backend-production.up.railway.app/ws";

  const resetConnection = () => {
    connected = false;
    username = "";
    messages = [];
    users = [];
    isSidebarOpen = false;
    areButtonsDisabled = false;
  };

  const onPressEnter = (event: KeyboardEvent) => {
    if ((event.code !== "Enter" && event.code !== "NumpadEnter") || event.shiftKey) return;

    if (!connected && joinButton && username) {
      event.preventDefault();
      joinButton.click();
      return;
    }

    if (connected && sendText && ws && event.target === sendText) {
      event.preventDefault();
      onPressSend();
    }
  };

  const onPressLeave = () => {
    if (ws && ws.readyState === WebSocket.OPEN && !areButtonsDisabled) {
      areButtonsDisabled = true;
      ws.close();
    }
  };

  const onPressSend = () => {
    if (!sendText?.value || !ws || ws.readyState !== WebSocket.OPEN) return;

    const message: UserMessage = { username, message: sendText.value };
    if (message.message.length <= 256) {
      ws.send(JSON.stringify(message));
      sendText.value = "";
    }
  };

  onDestroy(() => {
    if (ws && ws.readyState === WebSocket.OPEN) ws.close();
  });

  const connect = () => {
    username = username.trim();
    if (username.length < 3 || username.length > 15) {
      Swal.fire({
        title: "Error",
        text: "Username must be between 3 and 15 characters.",
        icon: "error",
        confirmButtonText: "Close",
        allowEnterKey: true,
        allowEscapeKey: true,
      });
      return;
    }

    if (!("WebSocket" in window || "MozWebSocket" in window)) {
      Swal.fire({
        title: "Error",
        text: "Your browser does not support WebSockets.",
        icon: "error",
        confirmButtonText: "Close",
        allowEnterKey: true,
        allowEscapeKey: true,
      });
      return;
    }

    ws = new WebSocket(`${websocketBaseUrl}?username=${encodeURIComponent(username)}`);
    ws.onopen = () => {
      connected = true;
    };

    ws.onclose = resetConnection;

    ws.onerror = () => {
      resetConnection();
      Swal.fire({
        title: "Connection failed",
        text: "We could not connect to the chatroom. Please try again.",
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
        messages = messages.length >= 50 ? [message, ...messages.slice(0, -1)] : [message, ...messages];
      } else {
        const data = parsedData as ClientList;
        users = [...data.user_list];
      }
    };
  };
</script>

<svelte:window on:keydown={onPressEnter} />

<div class="appShell">
  <header class="header">
    <a
      class="headerLink"
      href="https://github.com/Jasonsd19/chatroom-backend"
      target="_blank"
      rel="noreferrer noopener"
      aria-label="View the backend source on GitHub"
    >
      <img src="github.svg" alt="" />
    </a>
    <div class="brand">Conversin</div>
    <a
      class="headerLink"
      href="https://jasondeol.com/"
      target="_blank"
      rel="noreferrer noopener"
      aria-label="Visit Jason Deol's portfolio"
    >
      <img src="website.svg" alt="" />
    </a>
  </header>

  {#if connected}
    <main class="roomLayout">
      <button
        class="menuButton"
        type="button"
        aria-label="Toggle chat details"
        aria-expanded={isSidebarOpen}
        on:click={() => (isSidebarOpen = !isSidebarOpen)}
      >
        <img src="expand.svg" alt="" />
      </button>

      <aside class="sidebar" class:sidebarOpen={isSidebarOpen}>
        <DescriptionUserContainer {users} />
      </aside>

      <section class="chatPanel" aria-label="Chatroom">
        <div class="chatHeader">
          <div>
            <p class="eyebrow">Live chat</p>
            <h1>The conversation</h1>
          </div>
          <span class="memberCount">{users.length} online</span>
        </div>

        <div class="messageContainer" aria-live="polite">
          {#if messages.length}
            {#each messages as message, i (i)}
              <article class="message" class:ownMessage={message.username === username}>
                <strong>{message.username}</strong>
                <p>{message.message}</p>
              </article>
            {/each}
          {:else}
            <div class="emptyState">
              <span>●</span>
              <p>No messages yet. Start the conversation.</p>
            </div>
          {/if}
        </div>

        <div class="composer">
          <button
            class="leaveButton"
            type="button"
            aria-label="Leave chatroom"
            title="Leave chatroom"
            on:click={onPressLeave}
            disabled={areButtonsDisabled}
          >
            <img src="leave.svg" alt="" />
          </button>
          <label class="srOnly" for="message">Message</label>
          <textarea id="message" maxlength={256} bind:this={sendText} placeholder="Write a message…" />
          <button class="sendButton" type="button" on:click={onPressSend} disabled={areButtonsDisabled}>Send</button>
        </div>
      </section>
    </main>
  {:else}
    <main class="joinScreen">
      <section class="joinCard" aria-labelledby="join-title">
        <p class="eyebrow">A small place to chat</p>
        <h1 id="join-title">Enter the room</h1>
        <p class="joinCopy">Choose a name and join the live conversation.</p>

        <label for="username">Username</label>
        <input id="username" inputmode="text" maxlength={15} bind:value={username} autocomplete="nickname" />
        <p class="inputHint">3–15 characters · names are case-insensitive</p>

        <button class="joinButton" type="button" on:click={connect} bind:this={joinButton}>Join chatroom</button>
      </section>
    </main>
  {/if}
</div>

<style>
  .appShell {
    min-height: 100dvh;
    background:
      radial-gradient(circle at top left, rgba(255, 255, 255, 0.08), transparent 28rem),
      #151515;
  }

  .header {
    box-sizing: border-box;
    display: grid;
    grid-template-columns: 3rem minmax(0, 1fr) 3rem;
    align-items: center;
    gap: 1rem;
    width: min(100%, 1500px);
    min-height: 5rem;
    margin: 0 auto;
    padding: 0.85rem clamp(1rem, 3vw, 2.5rem);
    border-bottom: 1px solid #4a4a4a;
  }

  .brand {
    color: #fafafa;
    font-size: clamp(1.55rem, 3vw, 2.25rem);
    font-weight: 700;
    letter-spacing: -0.09em;
    text-align: center;
  }

  .headerLink,
  .menuButton,
  .leaveButton {
    display: grid;
    place-items: center;
    border: 1px solid #4a4a4a;
    background: #202020;
    color: #f5f5f5;
    transition: background-color 160ms ease, border-color 160ms ease, transform 160ms ease;
  }

  .headerLink {
    width: 2.5rem;
    height: 2.5rem;
    border-radius: 50%;
  }

  .headerLink img {
    width: 1.15rem;
    height: 1.15rem;
  }

  .headerLink:hover,
  .menuButton:hover,
  .leaveButton:hover:not(:disabled) {
    border-color: #d7d7d7;
    background: #303030;
    transform: translateY(-1px);
  }

  .roomLayout {
    display: grid;
    grid-template-columns: minmax(15rem, 0.68fr) minmax(0, 1.65fr);
    gap: clamp(1rem, 2vw, 1.75rem);
    width: min(100%, 1500px);
    min-height: calc(100dvh - 5rem);
    margin: 0 auto;
    padding: clamp(1rem, 2.5vw, 2rem);
    box-sizing: border-box;
  }

  .sidebar,
  .chatPanel {
    min-height: 0;
  }

  .chatPanel {
    display: grid;
    grid-template-rows: auto minmax(0, 1fr) auto;
    min-height: min(44rem, calc(100dvh - 9rem));
    overflow: hidden;
    border: 1px solid #5b5b5b;
    border-radius: 0.85rem;
    background: #1d1d1d;
    box-shadow: 0 1.25rem 3rem rgba(0, 0, 0, 0.22);
  }

  .chatHeader {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    padding: 1.15rem clamp(1rem, 2.5vw, 1.75rem);
    border-bottom: 1px solid #444;
  }

  .eyebrow {
    margin: 0 0 0.35rem;
    color: #a6a6a6;
    font-size: 0.68rem;
    font-weight: 700;
    letter-spacing: 0.12em;
    text-transform: uppercase;
  }

  h1 {
    margin: 0;
    color: #f7f7f7;
    font-size: clamp(1.15rem, 2vw, 1.55rem);
    line-height: 1.1;
  }

  .memberCount {
    flex: 0 0 auto;
    padding: 0.35rem 0.65rem;
    border: 1px solid #4e4e4e;
    border-radius: 999px;
    color: #d6d6d6;
    font-size: 0.72rem;
  }

  .messageContainer {
    display: flex;
    flex-direction: column-reverse;
    gap: 0.7rem;
    min-height: 0;
    overflow-y: auto;
    padding: 1.25rem clamp(1rem, 2.5vw, 1.75rem);
  }

  .message {
    max-width: min(42rem, 88%);
    padding: 0.75rem 0.9rem;
    border: 1px solid #4b4b4b;
    border-radius: 0.35rem 0.85rem 0.85rem;
    background: #242424;
  }

  .message strong {
    display: block;
    margin-bottom: 0.25rem;
    color: #fff;
    font-size: 0.78rem;
  }

  .message p {
    margin: 0;
    color: #d5d5d5;
    font-size: 0.88rem;
    line-height: 1.45;
    overflow-wrap: anywhere;
    white-space: pre-wrap;
  }

  .ownMessage {
    align-self: flex-end;
    border-color: #868686;
    background: #2b2b2b;
    border-radius: 0.85rem 0.35rem 0.85rem 0.85rem;
  }

  .emptyState {
    display: grid;
    place-content: center;
    justify-items: center;
    flex: 1;
    min-height: 100%;
    color: #9c9c9c;
    text-align: center;
  }

  .emptyState span {
    margin-bottom: 0.75rem;
    color: #ececec;
    font-size: 1.4rem;
  }

  .emptyState p {
    margin: 0;
    font-size: 0.85rem;
  }

  .composer {
    display: grid;
    grid-template-columns: auto minmax(0, 1fr) auto;
    gap: 0.7rem;
    align-items: center;
    padding: 0.9rem clamp(1rem, 2.5vw, 1.5rem);
    border-top: 1px solid #444;
    background: #202020;
  }

  .leaveButton {
    width: 2.45rem;
    height: 2.45rem;
    padding: 0.55rem;
    border-radius: 0.55rem;
  }

  .leaveButton img {
    width: 100%;
    height: 100%;
  }

  textarea,
  input {
    box-sizing: border-box;
    width: 100%;
    border: 1px solid #555;
    border-radius: 0.5rem;
    outline: none;
    background: #151515;
    color: #f8f8f8;
    font: inherit;
    transition: border-color 160ms ease, box-shadow 160ms ease;
  }

  textarea:focus,
  input:focus {
    border-color: #d4d4d4;
    box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.1);
  }

  textarea {
    min-height: 2.55rem;
    max-height: 6rem;
    padding: 0.65rem 0.75rem;
    resize: vertical;
    font-size: 0.86rem;
  }

  .sendButton,
  .joinButton {
    border: 1px solid #e9e9e9;
    border-radius: 0.5rem;
    background: #ededed;
    color: #151515;
    font: inherit;
    font-size: 0.8rem;
    font-weight: 700;
    letter-spacing: 0.02em;
    transition: background-color 160ms ease, transform 160ms ease, box-shadow 160ms ease;
  }

  .sendButton {
    min-height: 2.55rem;
    padding: 0 1rem;
  }

  .sendButton:hover:not(:disabled),
  .joinButton:hover {
    background: #fff;
    box-shadow: 0 0.35rem 0.9rem rgba(255, 255, 255, 0.1);
    transform: translateY(-1px);
  }

  button:disabled {
    cursor: not-allowed;
    opacity: 0.55;
  }

  .joinScreen {
    display: grid;
    place-items: center;
    min-height: calc(100dvh - 5rem);
    padding: clamp(1.5rem, 5vw, 4rem);
    box-sizing: border-box;
  }

  .joinCard {
    width: min(100%, 29rem);
    padding: clamp(1.5rem, 4vw, 2.5rem);
    border: 1px solid #555;
    border-radius: 0.9rem;
    background: #1d1d1d;
    box-shadow: 0 1.5rem 3.5rem rgba(0, 0, 0, 0.25);
  }

  .joinCard h1 {
    font-size: clamp(1.55rem, 4vw, 2.1rem);
  }

  .joinCopy {
    margin: 0.7rem 0 2rem;
    color: #b5b5b5;
    font-size: 0.92rem;
    line-height: 1.55;
  }

  .joinCard label {
    display: block;
    margin-bottom: 0.55rem;
    color: #ededed;
    font-size: 0.83rem;
    font-weight: 700;
  }

  .joinCard input {
    padding: 0.8rem 0.85rem;
    font-size: 1rem;
  }

  .inputHint {
    margin: 0.55rem 0 1.5rem;
    color: #909090;
    font-size: 0.72rem;
  }

  .joinButton {
    width: 100%;
    min-height: 2.8rem;
  }

  .menuButton {
    display: none;
  }

  .srOnly {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
  }

  @media (max-width: 700px) {
    .header {
      min-height: 4.35rem;
      grid-template-columns: 2.5rem minmax(0, 1fr) 2.5rem;
      padding-inline: 1rem;
    }

    .headerLink {
      width: 2.2rem;
      height: 2.2rem;
    }

    .roomLayout {
      display: block;
      min-height: calc(100dvh - 4.35rem);
      padding: 0.75rem;
    }

    .chatPanel {
      min-height: calc(100dvh - 5.85rem);
    }

    .menuButton {
      position: fixed;
      z-index: 3;
      top: 5rem;
      right: 1.4rem;
      display: grid;
      width: 2.75rem;
      height: 2.75rem;
      padding: 0.65rem;
      border-radius: 50%;
      box-shadow: 0 0.6rem 1.3rem rgba(0, 0, 0, 0.3);
    }

    .menuButton img {
      width: 100%;
      height: 100%;
    }

    .chatHeader {
      padding-right: 4.75rem;
    }

    .sidebar {
      position: fixed;
      z-index: 2;
      top: 4.35rem;
      right: 0;
      bottom: 0;
      width: min(88vw, 24rem);
      padding: 0.75rem;
      box-sizing: border-box;
      transform: translateX(100%);
      transition: transform 220ms ease;
    }

    .sidebarOpen {
      transform: translateX(0);
    }

    .messageContainer {
      padding: 1rem;
    }

    .composer {
      gap: 0.5rem;
      padding: 0.75rem;
    }

    .sendButton {
      padding: 0 0.7rem;
    }
  }

  @media (max-width: 420px) {
    .memberCount {
      display: none;
    }

    .chatHeader {
      padding: 1rem;
    }

    .leaveButton {
      width: 2.25rem;
      height: 2.25rem;
    }

    .sendButton {
      font-size: 0.72rem;
    }
  }
</style>
