<script lang="ts">
  export let users: string[];
</script>

<div class="sidebarContent">
  <section class="aboutSection" aria-labelledby="about-heading">
    <p class="eyebrow">About Conversin</p>
    <h2 id="about-heading">A deliberately simple chatroom.</h2>
    <p>
      Conversin is a small real-time chat application built with Go, WebSockets, TypeScript, and Svelte.
    </p>
    <p>
      It is a place to start a conversation without accounts, feeds, or distractions—just choose a name and join in.
    </p>
  </section>

  <section class="peopleSection" aria-labelledby="online-heading">
    <div class="sectionHeader">
      <div>
        <p class="eyebrow">In this room</p>
        <h2 id="online-heading">Online now</h2>
      </div>
      <span>{users.length}</span>
    </div>

    <div class="userList">
      {#if users.length}
        {#each users as user (user)}
          <div class="user">
            <img src="online.svg" alt="Online" />
            <span>{user}</span>
          </div>
        {/each}
      {:else}
        <p class="emptyUsers">Waiting for someone to join.</p>
      {/if}
    </div>
  </section>
</div>

<style>
  .sidebarContent {
    display: grid;
    grid-template-rows: minmax(0, 1fr) auto;
    height: 100%;
    min-height: min(32rem, calc(100dvh - 9rem));
    overflow: hidden;
    border: 1px solid #5b5b5b;
    border-radius: 0.85rem;
    background: #1d1d1d;
    box-shadow: 0 1.25rem 3rem rgba(0, 0, 0, 0.22);
  }

  .aboutSection,
  .peopleSection {
    padding: clamp(1rem, 2.4vw, 1.5rem);
  }

  .aboutSection {
    overflow-y: auto;
  }

  .eyebrow {
    margin: 0 0 0.4rem;
    color: #a6a6a6;
    font-size: 0.65rem;
    font-weight: 700;
    letter-spacing: 0.12em;
    text-transform: uppercase;
  }

  h2 {
    margin: 0;
    color: #f7f7f7;
    font-size: 1rem;
    line-height: 1.35;
  }

  .aboutSection > p:not(.eyebrow) {
    margin: 1rem 0 0;
    color: #b8b8b8;
    font-size: 0.82rem;
    line-height: 1.6;
  }

  .peopleSection {
    min-height: 10rem;
    border-top: 1px solid #444;
    background: #202020;
  }

  .sectionHeader {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 0.75rem;
  }

  .sectionHeader span {
    display: grid;
    width: 1.75rem;
    height: 1.75rem;
    place-items: center;
    border: 1px solid #555;
    border-radius: 50%;
    color: #ececec;
    font-size: 0.72rem;
  }

  .userList {
    display: grid;
    gap: 0.45rem;
    max-height: 12rem;
    margin-top: 1rem;
    overflow-y: auto;
  }

  .user {
    display: flex;
    align-items: center;
    gap: 0.55rem;
    min-width: 0;
    padding: 0.45rem 0.55rem;
    border: 1px solid transparent;
    border-radius: 0.4rem;
    color: #dedede;
    font-size: 0.8rem;
  }

  .user:hover {
    border-color: #484848;
    background: #292929;
  }

  .user img {
    width: 0.6rem;
    height: 0.6rem;
    flex: 0 0 auto;
  }

  .user span {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .emptyUsers {
    margin: 0;
    color: #929292;
    font-size: 0.75rem;
    font-style: italic;
  }

  @media (max-width: 700px) {
    .sidebarContent {
      min-height: 100%;
      background: #1b1b1b;
    }

    .aboutSection {
      padding-top: 1.5rem;
    }
  }
</style>
