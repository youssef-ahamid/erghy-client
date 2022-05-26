<script>
  import "../app.css";
  import { io } from "socket.io-client";
  import { flip } from "svelte/animate";
  import Message from "./lib/Message.svelte";
  import Modal from "./lib/Modal.svelte";

  const socket = io("http://localhost:8080");
  let user, room, messagesEnd, modalActive;
  let username = localStorage.getItem("username");
  let messages = [];
  let msg = "";

  const addMessage = (message, reset = false) => {
    messages = [message, ...messages];
    if (reset) msg = "";
    if (!!messagesEnd)
      messagesEnd.scrollIntoView({ behavior: "smooth", block: "end" });
    else console.log("messages end not mounted");
  };

  const addRoom = (r, go = false) => {
    user.rooms = [...user.rooms, r];
    if (go) room = r;
  };

  socket.on("disconnect", () => {
    console.log("disconnected");
  });

  socket.on("receive-message", (message) => {
    addMessage(message);
  });

  const createRoom = () => {
    if (!user) {
      login();
      return;
    }
    if (!modalActive) {
      modalActive = true;
      return;
    }
    socket.emit("create-room", newRoom, (r) => {
      addRoom(r);
      modalActive = false;
    });
  };

  const login = () => {
    if (!username) return;
    localStorage.setItem("username", username);
    socket.emit("get-user-chats", username, (u) => {
      if (u.status === 400) return;
      user = u;
    });
  };

  const selectRoom = (r) => {
    room = r;
    messages = room.last10messages;
  };

  const clearRoom = () => {
    room = null;
    messages = [];
  };

  const send = () => {
    if (!user) {
      login();
      return;
    }
    socket.emit("send-message", msg, room._id);
    addMessage({ text: msg, sender: user._id }, true);
  };
  let newRoom = {
    name: "",
    description: "",
    participants: [],
  };

  socket.on("connect", () => {
    console.log("connected");
    login();
  });

  socket.on("no-user", login);
</script>

<main>
  {#if room}
    <div class="w-full text-center py-4 bg-slate-300 fixed top-0">
      <button
        class="text-sm lowercase font-bold text-blue-600 absolute left-0 top-1/2 -translate-y-1/2"
        on:click={clearRoom}>back</button
      >
      <h1>{room.title}</h1>
    </div>
    <div class="w-full flex flex-col-reverse mt-20">
      {#each messages as message}
        {#key message._id || message.text.concat(message.sender)}
          <Message {message} mine={message.sender == user._id} />
        {/key}
      {/each}
    </div>
    <form on:submit|preventDefault={send}>
      <label for="message" class="fixed bottom-0 py-5 bg-slate-200">
        <textarea
          type="text"
          name="message"
          placeholder="Message"
          bind:value={msg}
        />
        <button type="submit">Send</button>
      </label>
    </form>
    <div bind:this={messagesEnd} class="h-40 w-full" />
  {:else if user}
    <div class="w-full text-center py-4 bg-slate-300 fixed top-0">
      <button
        class="text-sm lowercase font-bold text-blue-600 absolute right-0 top-1/2 -translate-y-1/2"
        on:click={createRoom}>new chat</button
      >
      <h1>chats</h1>
    </div>
    <Modal bind:active={modalActive}>
      <h2 class="font-bold text-xl text-center p-2">Create A New Room</h2>
      <form on:submit|preventDefault={createRoom}>
        <label for="title" class="p-2 flex-wrap group">
          <p class="group-focus-within:text-blue-60">room title</p>
          <input
            type="text"
            name="title"
            class="peer"
            bind:value={newRoom.title}
          />
        </label>
        <label for="description" class="p-2 flex-wrap group">
          <p class="group-focus-within:text-blue-60">room description</p>
          <textarea
            type="text"
            name="description"
            placeholder="What we're gathering around"
            bind:value={newRoom.description}
          />
        </label>
        <label for="invite" class="p-2 flex-wrap group">
          <p class="group-focus-within:text-blue-60">invite a friend</p>
          <input
            type="text"
            name="invite"
            placeholder="write their username"
            bind:value={newRoom.invite}
          />
        </label>
        <button type="submit" class="min-w-full mt-6"> Create </button>
      </form>
    </Modal>

    <div class="mt-12 flex flex-col-reverse">
      {#each user.rooms as r (r._id)}
        <div
          animate:flip
          on:click={() => selectRoom(r)}
          class="w-full p-5 odd:bg-slate-200 even:bg-white relative"
        >
          <h2 class="font-bold text-xl">{r.title}</h2>
          <p>{r.description}</p>
          <span
            class="absolute right-2 top-1/2 -translate-y-1/2 rounded-full w-4 h-4 text-center bg-slate-400 text-white font-bold text-xs"
            >></span
          >
        </div>
      {/each}
    </div>
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
    {/if}
  </form>
</main>
