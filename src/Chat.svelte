<script>
  import Login from './Login.svelte';
  import ChatMessage from './ChatMessage.svelte';
  import { onMount } from 'svelte';
  import { username, user } from './user';
  import debounce from 'lodash.debounce';

  import GUN from 'gun';
  import axios from 'axios';

  const db = GUN();
  // const ipfsAPIUrl = 'https://ipfs.infura.io:5001/api/v0'; // Your Infura IPFS API URL
  //infura
  // const apiKey = '5ceff1f78aea472abf6f3363033640d9';
  // const apiSecret = '8vRWcKqKP+gUOJNuZdsM1sZX5icEg8/1s4BIbdjTRPeKVu93x2rPQA';

  //basic pinata
  // const JWT = eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySW5mb3JtYXRpb24iOnsiaWQiOiI2ZWRiMDUyYS02MzlmLTRkN2ItODhjMC0zMmQzZDI4NjE0OWQiLCJlbWFpbCI6ImRldmFuc2gwMDdrYXVzaGlrQGdtYWlsLmNvbSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJwaW5fcG9saWN5Ijp7InJlZ2lvbnMiOlt7ImlkIjoiRlJBMSIsImRlc2lyZWRSZXBsaWNhdGlvbkNvdW50IjoxfSx7ImlkIjoiTllDMSIsImRlc2lyZWRSZXBsaWNhdGlvbkNvdW50IjoxfV0sInZlcnNpb24iOjF9LCJtZmFfZW5hYmxlZCI6ZmFsc2UsInN0YXR1cyI6IkFDVElWRSJ9LCJhdXRoZW50aWNhdGlvblR5cGUiOiJzY29wZWRLZXkiLCJzY29wZWRLZXlLZXkiOiJmNDRkODBmN2U2ZDdiOWQxNjFkNSIsInNjb3BlZEtleVNlY3JldCI6ImIzYTM1ZjIwZTNkZWM3ZWQ5YTdlOTY3NGJjYTUyZTNhYzVjYjNlMGQ2ZTIzZjk4ZjA4NDRlNWM2MWVmYWFiMGYiLCJpYXQiOjE3MTM2ODM1NTh9.-28h3px6kU3vOmdkO3qJ0D8t9Rd7U01Ug1HQtTpkqw4
  // const gateway_key = qySif5m7xFG6EmY7w8QkRMrP7htTG_rpLpwUIxxrU6Y3qiYfixfggPdGJPbPE1wG
  // https://white-elegant-weasel-617.mypinata.cloud/{CID}?pinataGatewayToken={Gateway API Key}


  let newFile;
  let messages = [];

  let scrollBottom;
  let lastScrollTop;
  let canAutoScroll = true;
  let unreadMessages = false;

  function autoScroll() {
    setTimeout(() => scrollBottom?.scrollIntoView({ behavior: 'auto' }), 50);
    unreadMessages = false;
  }

  function watchScroll(e) {
    canAutoScroll = (e.target.scrollTop || Infinity) > lastScrollTop;
    lastScrollTop = e.target.scrollTop;
  }

  $: debouncedWatchScroll = debounce(watchScroll, 1000);

  onMount(() => {
    var match = {
      '.': {
        '>': new Date(+new Date() - 1 * 1000 * 60 * 60 * 10).toISOString(),
      },
      '-': 1,
    };

    db.get('distributed_backup')
      .map(match)
      .once(async (data, id) => {
        if (data) {
          const key = '#foo';

          var message = {
            who: await db.user(data).get('alias'),
            what: (await SEA.decrypt(data.what, key)) + '',
            when: GUN.state.is(data, 'what'),
            fileName: await data.fileName
          };

          if (message.what) {
            messages = [...messages.slice(-100), message].sort((a, b) => a.when - b.when);
            if (canAutoScroll) {
              autoScroll();
            } else {
              unreadMessages = true;
            }
          }
        }
      });
  });

  async function sendMessage() {
    if (!newFile) return;

    let file = null;
    const formData = new FormData();

    // Check if a file is selected
    const fileInput = document.getElementById('fileInput');
    if (fileInput && fileInput.files.length > 0) {
      file = fileInput.files[0];
      formData.append('file', file);
    }

    try {
        const fileData = new FormData();
        fileData.append("file", file);

        const PINATA_API_KEY = 'be9a5a81d89176e88559';
        const PINATA_SECRET_KEY = '596e909ee13bf5a5001298116c388b86b16ca34a546f5c492a16a808239a964a';

        const responseData = await axios({
        method: "post",
        url: "https://api.pinata.cloud/pinning/pinFileToIPFS",
        data: fileData,
        headers: {
          pinata_api_key: PINATA_API_KEY,
          pinata_secret_api_key: PINATA_SECRET_KEY,
          "Content-Type": "multipart/form-data",
          },
          onUploadProgress: function(progressEvent) {
            const percentCompleted = Math.round((progressEvent.loaded * 100) / progressEvent.total);
            const progress = document.getElementById('uploadProgress');
            progress.value = percentCompleted;
            progress.style.display = 'block'; // Hide the progress bar
          }
        })

        const fileUrl = "https://gateway.pinata.cloud/ipfs/" + responseData.data.IpfsHash;
      const cid = responseData.data.IpfsHash;
      // console.log(cid);
      const secret = await SEA.encrypt(cid, '#foo');
      // console.log(secret, file.name); 
      const message = user.get('all').set({ what: secret, fileName: file.name });
      
      const naam = await user.get('all');
      // console.log(message);
      const index = new Date().toISOString();
      db.get('distributed_backup').get(index).put(message);

      // Reset progress bar and possibly hide it
      const progress = document.getElementById('uploadProgress');
      progress.value = 0; // Reset progress bar to zero
      progress.style.display = 'none'; // Hide the progress bar

      newFile = '';
      canAutoScroll = true;
      autoScroll();
    } catch (err) {
      console.error('Error uploading file to IPFS:', err);
    }
  }

  async function downloadFile(message) {
    try {
      // console.log("DOWBNLOAD TIMEW", message);
      // const cid = await SEA.decrypt(message.what, '#foo');
      // console.log(message.what);
      const fileUrl = `https://gateway.pinata.cloud/ipfs/${message.what}`; // Correctly formatted URL to access the file
      // console.log(fileUrl);
      
      const response = await axios.get(fileUrl, { responseType: 'blob' });        
      const fileBlob = response.data;

      const fileURL = URL.createObjectURL(fileBlob);

      // Create a link element and trigger the download
      const link = document.createElement('a');
      link.href = fileURL;
      link.download = `downloaded_file_${message.what}.zip`; // Set filename dynamically based on CID
      document.body.appendChild(link);  // Append link to the body temporarily
      link.click();  // Simulate a click on the link to start download
      document.body.removeChild(link);  // Remove the link after triggering download
    
    } catch (error) {
        alert("Failed to download the file. Please check if your ad blocker or any browser extensions are blocking the request.");
        console.error('Error downloading file from IPFS via Pinata:', error);
    }

  }
</script>

<div class="container">
  {#if $username}
    <main on:scroll={debouncedWatchScroll}>
      {#each messages as message (message.when)}
        <ChatMessage {message} sender={$username} {downloadFile}/>
      {/each}
      <div class="dummy" bind:this={scrollBottom} />
    </main>

    <form on:submit|preventDefault={sendMessage}>
      <input type="file" id="fileInput" accept=".zip" bind:value={newFile}/> <!-- File input field -->
      <button type="button" on:click={sendMessage} disabled={!newFile}>💥</button>
      <progress id="uploadProgress" value="0" max="100" style="display: none;"></progress> <!-- Progress bar -->
      </form>

    {#if !canAutoScroll}
    <div class="scroll-button">
      <button on:click={autoScroll} class:red={unreadMessages}>
        {#if unreadMessages}
          💬
        {/if}
        👇
      </button>
    </div>
    {/if}
  {:else}
    <main>
      <Login />
    </main>
  {/if}
</div>
