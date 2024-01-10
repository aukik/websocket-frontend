<script>
    import { onMount, onDestroy } from 'svelte';
    import io from 'socket.io-client';

    //generate random user id
    const userId = Math.floor(Math.random() * 1000000000);
    let socket;
    let messages=[]

    onMount(() => {
        console.log("on mount")
        // Initialize the socket connection
        socket = io('localhost:5000'); // replace with your server's URL and port

        socket.on('connect', () => {
            console.log('Connected to the server!');
            socket.emit('userId', { message:"I am connected now",userId: userId });
        });

        socket.on('user event', (data) => {
            // messages.push(data)
            messages = [...messages, data]
            console.log(data);
        });

        socket.on('disconnect', () => {
            console.log('Disconnected from the server!');
        });
    });

    onDestroy(() => {
        // Disconnect the socket when the component is destroyed
        if (socket) {
            socket.disconnect();
        }
    });
    const sendMessage = () => {
        console.log("send message")
        socket.emit('user event', { "message":"I pressed a button", userId:userId });
    }

</script>



<div class="w-full flex flex-row justify-center my-8">
    <div class="flex flex-col justify-center ">
        <p class="text-2xl font-bold self-center">Welcome to Instagram Scrapper</p>
        <p>User Id {userId}</p>
    </div>
        
</div>
<div class="w-full h-full flex flex-row justify-center my-8">
    <div class="flex flex-col justify-center ">
        <p class="text-2xl font-bold self-center">Messages</p>
        {#each messages as message}
            <p>{message.message}</p>
        {/each}
    </div>

</div>
<div class="w-full h-full flex flex-row justify-center my-8">
    <button class="bg-slate-100 border border-black p-2 rounded-lg my-4" on:click={sendMessage}>Send Message</button>
</div>
