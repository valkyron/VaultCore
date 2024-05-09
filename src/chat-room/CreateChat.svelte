
<!-- CreateChat.svelte -->
<script>
    // import GUN from 'gun';
    import { user } from './user';
    const db = GUN();

    let chatname;
    let password;
    let username;

    // The substring length is 8 characters.
    // Since each character can be one of 36 possible values (26 letters + 10 digits for base36), 
    // the total number of possible substrings is 36^8 (over 2 trillion).
    function generateChatID() {
        return Math.random().toString(36).substring(2, 10); // Generate a random string
    }

    async function noin() {
        const alias = await user.get('alias');
        console.log(alias);
    }

    async function join() {
        // const chat = await db.get('chats').get(chatname);
        // console.log(chatID);
        // chat.get(chatID);
        const ps = await db.get('chats').get(chatname).then();
        console.log(chatname);
        // const actual_pass = await SEA.decrypt(await db.get('chats').get(chatID).get('password'), '#foo'); 
        console.log(ps.password, password);
        // console.log(await SEA.decrypt(actual_pass, '#foo'));
        // actual_pass = await SEA.decrypt(actual_pass, '#foo');
    // const secret = await SEA.encrypt(chatID, '#foo');
        // chat.get('participants').set({ [username]: true }); // Add the creator as the participant

    // console.log(chatname, db.user().recall({sessionStorage: true}).get('alias'));
    // user.get('chatID').set()
    // chat.auth(chatname, password, ({ err }) => err && alert(err));
    // console.log(ps);
        if(password == await SEA.decrypt(ps.password, '#foo')) {
            const alias =  await user.get('alias');

            user.get('currChat').put(chatname);
            console.log(`Welcome ${alias} to the vault ${chatname}`);
        } else {
            alert("Chatname or password are incorrect!");
        }
    }

    async function createChat() {
        if (!chatname || !password || chatname.length < 3 || chatname.length > 30) {
            let errorMessage = 'Chat name and password must be between 3 and 30 characters.';
            alert(errorMessage);
            return; // Stop execution if chat name length is invalid
        }

        // console.log(password);

        // generate a chatID for the new chat, and put it in the chat node
        // const chatID = generateChatID();
        // console.log(password);
        const encrypted_password = await SEA.encrypt(password, '#foo');
        db.get('chats').get(chatname).put({'password' : encrypted_password});
        console.log(await db.get('chats').get(chatname).then())
        join(chatname);
        // console.log(chatname)

        // db.get('chats').get("abc").put({'password': "aop"});
        // var password = await db.get('chats').get("abc").then();
        // console.log(password.password); // Output: "aop"


        // console.log(db.get('chats').get("abc").then());
        // const decrypted_password = await SEA.decrypt(encrypted_password, '#foo');
        // console.log(password, decrypted_password);
        // // chat.get('joinID').put(secret);
        
        // db.get('chats').put(null);
        // console.log(encrypted_password)
        // const ps = await db.get('chats').get(chatID).then();
        // console.log(ps.password);
        // console.log(await SEA.decrypt(ps.password, '#foo'));
        // db.get('chats').put(null);


        // chat.create(chatname, password, ({ err }) => {
        //     if (err) {
        //         alert(err);
        //     } else {
        //         join(chatID);
        //     }
        // });
        
        

        // const chatRef = db.get(`${chatID}`);
        // chatRef.put({ name: chatname }); // You can set a default name for the new chat
    }
</script>

<label for="chatname">Chat Name:</label>
<input name="chatname" bind:value={chatname} minlength="3" maxlength="30" />

<label for="password">Create Password:</label>
<input name="password" bind:value={password} type="password" />

<button class="login" on:click={createChat}>Create New Chat</button>
<button class="login" on:click={join}>Join Chat</button>

<button class="login" on:click={noin}>Print username</button>
