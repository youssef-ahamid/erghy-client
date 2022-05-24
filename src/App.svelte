<script>
  import "../app.css";
  import { io } from "socket.io-client";
  import { flip } from "svelte/animate";
  import Message from "./lib/Message.svelte";

  const socket = io("http://localhost:8080");
  let user, username, room, messagesEnd;
  let update = 0;
  let messages = [];
  let msg = "";

  const addMessage = (message, reset = false) => {
    messages = [message, ...messages];
    update++;
    if (reset) msg = "";
    if (!!messagesEnd)
      messagesEnd.scrollIntoView({ behavior: "smooth", block: "end" });
    else console.log("messages end not mounted");
  };

  socket.on("connect", () => {
    console.log("connected");
  });

  socket.on("disconnect", () => {
    console.log("disconnected");
  });

  socket.on("receive-message", (message) => {
    addMessage(message);
  });

  const login = () => {
    if (!username) return;
    socket.emit("get-user-chats", username, (u) => {
      if (u.status === 400) return;
      user = u;
    });
  };

  const selectRoom = (r) => {
    room = r;
    messages = room.last10messages;
  }

  const send = () => {
    if (!user) {
      login();
      return;
    }
    socket.emit("send-message", msg, room._id);
    addMessage({ text: msg, sender: user._id }, true);
  };

  socket.on("no-user", login);
</script>

<main>
  {#if room }
  <div class="w-full flex items-center bg-slate-300">
    <h1>{room.title}</h1>
    <p>{room.description}</p>
  </div>
  <div class="w-full flex flex-col-reverse">
    {#each messages as message}
      {#key message._id || message.text.concat(message.sender)}
        <Message {message} mine={message.sender == user._id} />
      {/key}
    {/each}
  </div>
  <div bind:this={messagesEnd} class="h-40 w-full" />
  {:else if user }
    {#each user.rooms as r }
      <div on:click={() => selectRoom(r)} class="w-full p-5 odd:bg-slate-200 even:bg-white">
        <h2>{room.title}</h2>
        <p>{room.description}</p>
      </div>
    {/each}
  {/if}
  <form on:submit|preventDefault={send}>
    {#if !user}
      <label for="username">
        <input
          type="text"
          name="username"
          placeholder="username"
          bind:value={username}
        />
        <button type="button" on:click={login}>login</button>
      </label>
    {:else}
      <label for="message" class="fixed bottom-0 py-5 bg-slate-200">
        <input
          type="text"
          name="message"
          placeholder="Message"
          bind:value={msg}
        />
        <button type="submit">Send</button>
      </label>
    {/if}
  </form>
</main>
