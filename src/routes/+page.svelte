<script>
    import { Alert } from 'flowbite-svelte';
    import { onMount, onDestroy } from 'svelte';
    // import io from 'socket.io-client';

    const userId = Math.floor(Math.random() * 1000000000);
    // @ts-ignore
    let socket;
    // @ts-ignore
    let messages = [];
    // @ts-ignore
    let currentState = "Waiting...";
    let numberOfInstances = 0;
    let isInstancesRunning = false;
    let isNewUserAdding = false;
    let isPostCheckingOn = false;

    // Fields for the POST request
    let orderId = '';
    let profileLink = '';
    let orderAmount = '';
    let maxPerPost = '';

    let scrapperButtonDisabled = false;

    // onMount(() => {
    //     socket = io('localhost:5000');

    //     socket.on('connect', () => {
    //         console.log('Connected to the server!');
    //         // @ts-ignore
    //         socket.emit('userId', { message: "I am connected now", userId: userId });
    //     });

    //     socket.on('user event', (data) => {
    //         // @ts-ignore
    //         messages = [...messages, data];
    //         console.log(data);
    //     });

    //     socket.on('disconnect', () => {
    //         console.log('Disconnected from the server!');
    //     });
    // });

    // onDestroy(() => {
    //     // @ts-ignore
    //     if (socket) {
    //         socket.disconnect();
    //     }
    // });
    onMount(() => {
        startPolling();
    });

    const sendMessage = () => {
        // @ts-ignore
        // socket.emit('user event', { "message": "I pressed a button", userId: userId });
        console.log("socket")
    }

    const startNewPostsChecker = async () => {
        try {
            const response = await fetch(import.meta.env.VITE_URL+'/start_new_posts_checker', {
                method: 'GET',
                headers: {
                    'Authorization': `Bearer ${sessionStorage.getItem('password')}` 
                },
            });

            if (response.ok) {
                console.log('New posts checker started successfully.');
                
                currentState = 'New posts checker started successfully!';
            } else {
                console.error('Failed to start new posts checker.');
            }
            showMessage();
        } catch (error) {
            console.error('Error:', error);
        }
    }


    const checkForState = async () => {
        try {
            const response = await fetch(import.meta.env.VITE_URL+'/checkForState', {
                method: 'GET',
                headers: {
                    'Authorization': `Bearer ${sessionStorage.getItem('password')}` 
                },
            });
            if (response.ok) {
                const data = await response.json();
                // Update variables based on the response
                numberOfInstances = data.number_of_instances;
                isInstancesRunning = data.is_instances_on;
                isNewUserAdding = data.is_new_user_adding_on;
                isPostCheckingOn = data.is_post_checking_on;
            } else {
                console.error('Failed to fetch state.');
            }
        } catch (error) {
            console.error('Error:', error);
        }
    };

    const startPolling = () => {
        checkForState(); // Initial call
        setInterval(checkForState, 10000); // Poll every 5 seconds
    };







    const endCurrentProcess = async () => {
        if(scrapperButtonDisabled) {
            currentState="Please try again after a while!"
            showMessage();
            return;
        }
        try {
            const response = await fetch(import.meta.env.VITE_URL+'/end', {
                method: 'GET',
                headers: {
                    'Authorization': `Bearer ${sessionStorage.getItem('password')}` 
                },
            });

            if (response.ok) {
                console.log('process Ended.');
                
                currentState = 'Current Process ended successfully!';
            } else {
                console.error('Failed to End the process.');
            }
            showMessage();
        } catch (error) {
            console.error('Error:', error);
        }
    }

    const submitPostRequest = async () => {
        try {
            const response = await fetch(import.meta.env.VITE_URL+'/userid_insert', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${sessionStorage.getItem('password')}` 
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
                currentState = "User added successfully!"
                currentState = `User added ${profileLink} successfully!`
            } else {
                console.error('Failed to make POST request.');
                currentState = "Failed to add user!"
            }
            showMessage();
        } catch (error) {
            console.error('Error:', error);
        }
    }

    // New form fields
    let instances = 1;

    const startInstances = async () => {
        try {
            if(instances==0) {
                currentState="Number of instances must be greater than zero!"
                showMessage();
                return;
            }
            if(scrapperButtonDisabled || isInstancesRunning) {
                currentState="Scrapper is already running!"
                showMessage();
                return;
            }
            scrapperButtonDisabled = true;
            currentState = `Starting ${instances} instances...`
            const response = await fetch(import.meta.env.VITE_URL+`/start?instances=${instances}`, {
                method: 'GET',
                headers: {
                    'Authorization': `Bearer ${sessionStorage.getItem('password')}` 
                },
            });

            if (response.ok) {
                console.log(`Started ${instances} instances successfully.`);
                currentState = `Started ${instances} instances successfully!`
               
            } else {
                console.error(`Failed to start ${instances} instances.`);
                currentState = `Failed to start ${instances} instances!`
            }
            showMessage();
            scrapperButtonDisabled = false;
        } catch (error) {
            console.error('Error:', error);
        }
    }

    

    let showAlert = false;
    let password = null
    function showMessage() {
        showAlert = true;
        setTimeout(() => {
            showAlert = false;
        }, 10000);

    }
</script>

<style>
    @import './css/styles.css';
</style>
{#if showAlert}
<div role="alert" class="alert alert-info w-fit alert-info shadow-lg fixed right-4 top-4 right px-8">
    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" class="stroke-current shrink-0 w-6 h-6"><path  strokeLinejoin="round" strokeWidth="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
    <div>
      <h3 class="font-bold text-white">{currentState}</h3>
   
    </div>

  </div>
  {/if}


    


  
<div  style="max-height: 90vh; , overflowY: auto">
<div class="w-full  flex flex-row justify-center my-8">
    <div class="flex flex-col  justify-center">
        <p class="text-2xl text-white font-bold self-center">Welcome to Instagram Scrapper</p>
        
    </div>
</div>

<!-- <div class="w-full h-full flex flex-row justify-center my-8">
    <div class="flex flex-col justify-center">
        <p class="text-2xl font-bold self-center">Current State</p>
        <p class="text-2xl font-bold self-center" style="color: red;">{currentState}</p>
    </div>
</div> -->




<div class="flex flex-row justify-around">
    <div class="w-1/2 px-16">
        <div>
            <div class="flex flex-col items-center my-8">
            
                <div class="w-full h-full flex flex-row justify-center my-8">
                    {#if isInstancesRunning}
                        <div class="flex flex-col justify-center ">
                            <p class="text-2xl font-bold self-center">Current State: {numberOfInstances} instances Running!</p>
                        </div>
                    {:else if isNewUserAdding}
                        <div class="flex flex-col justify-center ">
                            <p class="text-2xl font-bold self-center">Current State: New User is being added!</p>
                        </div>
                    {:else if isPostCheckingOn}
                        <div class="flex flex-col justify-center ">
                            <p class="text-2xl font-bold self-center">Current State: Post checker is on!</p>
                        </div>
                    {:else}
                        <div class="flex flex-col justify-center ">
                            <p class="text-2xl font-bold self-center">Current State: No process is Running!</p>
                        </div>
                    {/if}
                </div>
    
            </div>
            <div class="flex flex-col items-center my-8 bg-slate-100 m-8 p-4 rounded-lg">
                <p class="text-lg text-black font-bold  my-2">Are multi-instances running: <span class="font-normal">{isInstancesRunning?"Yes":"No"}</span></p>
                <p class="text-lg text-black font-bold  my-2">Number of instances running: <span class="font-normal">{numberOfInstances}</span></p>
                <p class="text-lg text-black font-bold  my-2">Is New Post Checker Running: <span class="font-normal">{isPostCheckingOn?"Yes":"No"}</span></p>
                <p class="text-lg text-black font-bold  my-2">Is Premium User Insertion Running: <span class="font-normal">{isNewUserAdding?"Yes":"No"}</span></p>
            </div>
            
            
        </div>

        <div role="alert" class="alert alert-success flex flex-col items-center alert-info shadow-lg right ">
           
            <div>
              <h3 class="font-bold text-white">Enter Your Password:</h3>
                <input type="password" id="password" bind:value={password}  class="bg-slate-50" />
                <div class="btn btn-accent my-2" on:click={()=>{sessionStorage.setItem('password',password)}}>Submit</div>
           
            </div>
        
          </div>

        
        
        
        
    </div>

    <div class="w-1/2 px-16">

        <div class=" bg-slate-100 rounded-lg flex flex-col items-center m-8 p-4">
            <h1 class="text-xl text-black font-bold self-center my-2">Run Scrapper</h1>
            <div >
                <label class="text-black " for="instances">Number of Instances:</label>
                <input class=" bg-slate-50" type="number" id="instances" bind:value={instances}  />
                <div class="flex flex-row justify-around my-4">
                    {#if scrapperButtonDisabled==false}
                        <div class="btn btn-success bg-white border border-gray-800 mx-2" on:click={startInstances}>Start Instances</div>
                    {:else}
                        <div class="loading loading-spinner loading-sm"></div>
                    {/if}
                    <div class="btn btn-error bg-black border border-gray-300 text-white mx-2" on:click={endCurrentProcess}>End Scrapping</div>
                    
                    <!-- <button on:click={showMessage}>gg</button> -->
                </div>
            </div>
            
            

        </div>
        
        
        <div class=" flex flex-col items-center my-8 bg-slate-100 m-8 p-4 rounded-lg">
            <h1 class="text-xl text-black font-bold self-center my-2">Add New Premium User</h1>
            <div class="my-2">
                <label class="text-black" for="orderId">Order ID:</label>
                <input  type="text" id="orderId" bind:value={orderId} class="bg-slate-50" />
            </div>
            
            <div class="my-2">
                <label class="text-black" for="profileLink">Profile Link:</label>
                <input type="text" id="profileLink" bind:value={profileLink} class="bg-slate-50" />
            </div>
            
            <div class="my-2">
                <label class="text-black" for="orderAmount">Order Amount:</label>
                <input type="text" id="orderAmount" bind:value={orderAmount} class="bg-slate-50" />
            </div>
            
            
            <div class="my-2">
                <label class="text-black" for="maxPerPost">Max Per Post:</label>
                <input type="text" id="maxPerPost" bind:value={maxPerPost} class="bg-slate-50" />
            </div>
            
        
            <div class="btn btn-primary my-2" on:click={submitPostRequest}>Add New User</div>
        </div>
        
        
        <!-- New form components -->
        
        <!-- <div class="flex flex-row justify-center my-4">
            <div class="btn btn-primary bg-blue-500 text-white border border-black p-2 rounded-lg my-4" on:click={startNewPostsChecker}>Check for new post</div>
        </div> -->
        
    </div>


</div>


</div>
