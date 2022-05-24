<script>
  export let mine = false;
  export let message = {};
</script>

<div class="w-full p-1 flex" class:justify-end={mine}>
    <div
      class="py-4 px-6 rounded-full {mine ? 'self-end bg-green-200' : 'self-start bg-slate-200 '}"
    >
      <p>{message.text}</p>
    </div>
</div>
