<script lang="ts">
  export let users: string[];
</script>

<div class="descUserContainer">
  <div class="descContainer">
    <p style="margin: 10px;">
      Conversin is a simple chatroom application I wrote utilizing Go for the backend and TypeScript/Svelte for the
      front-end. The main focus was on working with and learning about websockets, and gaining a deeper understanding on
      developing APIs using the net/http package for Go. While I did spend some time working on the front-end it is
      still very rudimentary and was not the main focus, however I still took the time to practice mobile-first
      development and make the application responsive to both mobile and desktop.<br /><br />
      If I had more time to spend on this project I would look to add two more features. One would be the ability to have
      multiple chatrooms available, and a simple selector to jump between them as needed. The second feature would be having
      the back-end cache recent messages, so when a user joins they can see what was previously being talked about (I would
      probably implement this with a linked list).<br /><br /> If you are interested in the back end or front end code you
      can click the GitHub icon in the top left to check it out. Also, if you are interested in learning more about me and
      other projects I have worked please feel free to click the icon in the top right to check out my portflio website.
    </p>
  </div>
  <div class="userContainer">
    {#each users as user, i (i)}
      <div class="user">
        <img style="width: 10px;" src="online.svg" alt="Online" />
        {user}
      </div>
    {/each}
  </div>
</div>

<style>
  .descUserContainer {
    display: flex;
    height: 100%;
    width: 100%;
    background-color: #242424;
    border: 2px solid grey;
    border-radius: 5px;
  }

  .descContainer {
    width: 60%;
    font-size: 12.5px;
    overflow: auto;
  }

  .userContainer {
    display: flex;
    flex-direction: column;
    flex-wrap: wrap;
    width: 40%;
    border-left: 2px solid grey;
  }

  .user {
    padding: 5px;
    font-size: 3.25vw;
  }

  @media only screen and (min-width: 600px) {
    .descUserContainer {
      display: flex;
      flex-direction: column;
      height: 100%;
      width: 40%;
      background-color: inherit;
      border: 0px solid black;
      border-radius: 0px;
    }

    .descContainer {
      padding-top: 20px;
      width: 100%;
      height: 60%;
      border-bottom: 5px solid black;
    }

    .userContainer {
      flex-direction: row;
      flex-wrap: wrap;
      width: 100%;
      height: 40%;
      border: 0px;
      font-size: 15px;
      overflow: auto;
    }

    .user {
      padding: 0;
      font-size: 8px;
      width: 30%;
      padding-left: 10px;
    }
  }

  @media only screen and (min-width: 800px) {
    .user {
      font-size: 9px;
    }
  }

  @media only screen and (min-width: 900px) {
    .user {
      font-size: 10px;
    }
  }

  @media only screen and (min-width: 1000px) {
    .user {
      font-size: 11px;
    }

    .descContainer {
      font-size: 15px;
    }
  }

  @media only screen and (min-width: 1200px) {
    .user {
      font-size: 13px;
    }
  }

  @media only screen and (min-width: 1400px) {
    .user {
      font-size: 15px;
    }
  }
</style>
