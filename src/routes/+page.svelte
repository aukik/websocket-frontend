<script>
    let userName="";
    let password="";
    let buttonDisabled = false;
    let login = async() => {
        buttonDisabled = true;
        const response = await fetch(import.meta.env.VITE_URL+'/login', {
            
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    username: userName,
                    password: password,
                }),
            },
            
            )
            
            if (response.ok) {
                // sessionStorage.setItem('password', response.json().access_token);
                const data = await response.json();
                
                sessionStorage.setItem('password', data.access_token);
                console.log('New posts checker started successfully.');
                window.location.href = "/scrapper";
                
            } else {
                console.error('Failed to start new posts checker.');
                alert('Invalid Credentials');
            }
            buttonDisabled = false;
    }
</script>

<div class="flex flex-col justify-center h-[100vh] py-8">
    <div class="flex flex-col items-center self-center bg-slate-100 rounded-3xl p-4">
        <h1 class="text-black text-2xl font-bold m-4">Login</h1>
        <div class="flex flex-col m-4">
            <p class="label text-black font-bold">Username</p>
            <input bind:value={userName} type="text" class="input input-alert border-2 border-slate-500 rounded-md text-white" />
            <p class="label text-black font-bold">Password</p>
            <input bind:value={password} type="password" class="input input-alert border-2 border-slate-500 rounded-md text-white" />
        </div>
        <button disabled={buttonDisabled} class="btn btn-primary m-4" on:click={login}>Login</button>
    </div>
</div>