<script>
    import { onMount, onDestroy } from 'svelte';
    import io from 'socket.io-client';

    const userId = Math.floor(Math.random() * 1000000000);
    let socket;
    let messages = [];

    // Fields for the POST request
    let orderId = '';
    let profileLink = '';
    let orderAmount = '';
    let maxPerPost = '';

    onMount(() => {
        socket = io('localhost:5000');

        socket.on('connect', () => {
            console.log('Connected to the server!');
            socket.emit('userId', { message: "I am connected now", userId: userId });
        });

        socket.on('user event', (data) => {
            messages = [...messages, data];
            console.log(data);
        });

        socket.on('disconnect', () => {
            console.log('Disconnected from the server!');
        });
    });

    onDestroy(() => {
        if (socket) {
            socket.disconnect();
        }
    });

    const sendMessage = () => {
        socket.emit('user event', { "message": "I pressed a button", userId: userId });
    }

    const startNewPostsChecker = async () => {
        try {
            const response = await fetch('http://65.108.39.161:80/start_new_posts_checker', {
                method: 'GET',
                headers: {
                    'Authorization': 'Bearer drdre'
                },
            });

            if (response.ok) {
                console.log('New posts checker started successfully.');
            } else {
                console.error('Failed to start new posts checker.');
            }
        } catch (error) {
            console.error('Error:', error);
        }
    }

    const submitPostRequest = async () => {
        try {
            const response = await fetch('http://65.108.39.161:80/userid_insert', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': 'Bearer drdre'
                },
                body: JSON.stringify({
                    order_id: orderId,
                    profile_link: profileLink,
                    order_amount: parseInt(orderAmount, 10),
                    max_per_post: parseInt(maxPerPost, 10),
                    comment: true,
                }),
            });

            if (response.ok) {
                console.log('POST request successful.');
            } else {
                console.error('Failed to make POST request.');
            }
        } catch (error) {
            console.error('Error:', error);
        }
    }

    // New form fields
    let instances = '';

    const startInstances = async () => {
        try {
            const response = await fetch(`http://65.108.39.161:80/start?instances=${instances}`, {
                method: 'GET',
                headers: {
                    'Authorization': 'Bearer drdre'
                },
            });

            if (response.ok) {
                console.log(`Started ${instances} instances successfully.`);
            } else {
                console.error(`Failed to start ${instances} instances.`);
            }
        } catch (error) {
            console.error('Error:', error);
        }
    }
</script>

<style>
    @import './css/styles.css';
</style>

<div class="w-full flex flex-row justify-center my-8">
    <div class="flex flex-col justify-center">
        <p class="text-2xl font-bold self-center">Welcome to Instagram Scrapper</p>
        <p>User Id {userId}</p>
    </div>
</div>

<div class="w-full h-full flex flex-row justify-center my-8">
    <div class="flex flex-col justify-center">
        <p class="text-2xl font-bold self-center">Current State</p>
        {#each messages as message}
            <p>{message.message}</p>
        {/each}
    </div>
</div>

<div class="w-full h-full flex flex-row justify-center my-8">
    <button class="bg-slate-100 border border-black p-2 rounded-lg my-4" on:click={sendMessage}>Send Message</button>
</div>

<div class="w-full h-full flex flex-row justify-center my-8">
    <button class="bg-blue-500 text-white border border-black p-2 rounded-lg my-4" on:click={startNewPostsChecker}>New post checker</button>
</div>

<div class="w-full h-full flex flex-col justify-center my-8 form-container">
    <label for="orderId">Order ID:</label>
    <input type="text" id="orderId" bind:value={orderId} class="form-group" />

    <label for="profileLink">Profile Link:</label>
    <input type="text" id="profileLink" bind:value={profileLink} class="form-group" />

    <label for="orderAmount">Order Amount:</label>
    <input type="text" id="orderAmount" bind:value={orderAmount} class="form-group" />

    <label for="maxPerPost">Max Per Post:</label>
    <input type="text" id="maxPerPost" bind:value={maxPerPost} class="form-group" />

    <button on:click={submitPostRequest}>Submit</button>
</div>

<!-- New form components -->
<div class="w-full h-full flex flex-col justify-center my-8 new-form-container">
    <label for="instances">Instances:</label>
    <input type="text" id="instances" bind:value={instances} class="form-group" />

    <button on:click={startInstances}>Start Instances</button>
</div>
