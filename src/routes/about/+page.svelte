<script lang="ts">
    // Import the package
    import NDK from "@nostr-dev-kit/ndk";
    //I just wanna connect while in the browser and not broser and node server as svelte does
    import { browser } from '$app/environment';
    import {NDKNip07Signer} from "@nostr-dev-kit/ndk"

    // Create a new NDK instance with explicit relays
    const ndk = new NDK({
        explicitRelayUrls: ["wss://relay.nostr.band", "wss://relay.damus.io", "wss://purplepag.es"],
    });

    //make suro to not run the code on the server
    if(browser){
        ndk.connect().then(() => {
            console.log("Connected")
        });
    }

    //const user = ndk.getUser({npub: 'npub18qjdjm9qlf2dl4rm2k8fye8y82ve33e6kdehnlnamnmvk69cl8wqtxyng3'});
    
    //const eventsPromise = ndk.fetchEvents({ kinds: [1], authors: [user.hexpubkey] });

    async function login(){
        const signer = new NDKNip07Signer();

        ndk.signer = signer;
        signer.user().then((user) => {
            console.log(user);
        });
    }
</script>

<h1>Nostr Start</h1>

<button on:click={login}>Login with Nostr</button>
<!--{#await user.fetchProfile() then events}
    <h2>{user.profile?.name}</h2>
    <p>
        <img src={user.profile?.image} style="width:100px; height:100px" alt="fdsa" />    
    </p>
    <p>{user.profile?.about}</p>
{/await}

{#await eventsPromise then events}
    {#each Array.from(events) as event}
        <div class="eventBlock">
            <p>{event.content}</p>
        </div>
    {/each}
{/await}

<style>
    .eventBlock {
        padding: 10px;
        border: 1px black;
        border-radius: 10px; 
        margin-bottom: 4px;
    }
</style>-->