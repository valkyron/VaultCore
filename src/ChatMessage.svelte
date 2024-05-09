<script>
  export let message;
  export let sender;
  export let downloadFile; // Function to trigger file download

  const messageClass = message.who === sender ? 'sent' : 'received';
  const ts = new Date(message.when);
</script>

<div class={`message ${messageClass}`}>
  <div class="message-text">
    <p>{message.fileName}</p> <!-- Display the file name -->
    <button on:click={() => downloadFile(message)}>⬇️</button> <!-- Download button -->
    <time>{ts.toLocaleTimeString()}</time>
  </div>
</div>
