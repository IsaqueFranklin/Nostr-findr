<script lang="ts">
    // Import the package
    import NDK from "@nostr-dev-kit/ndk";
    //I just wanna connect while in the browser and not broser and node server as svelte does
    import { browser } from '$app/environment';
    import {NDKNip07Signer} from "@nostr-dev-kit/ndk";
    import type {NDKUserProfile} from "@nostr-dev-kit/ndk"

    // Create a new NDK instance with explicit relays
    const ndk = new NDK({
        explicitRelayUrls: ["wss://relay.nostr.band", "wss://relay.damus.io", "wss://purplepag.es"],
    });

    let userProfile: NDKUserProfile;
    let hexpubkey = null;

    //make suro to not run the code on the server
    if(browser){
        ndk.connect().then(() => {
            console.log("Connected")
        });
    }

    //const user = ndk.getUser({npub: 'npub18qjdjm9qlf2dl4rm2k8fye8y82ve33e6kdehnlnamnmvk69cl8wqtxyng3'});
    
    const eventsPromise = ndk.fetchEvents({ kinds: [1], authors: [hexpubkey] });

    async function login(){
        const signer = new NDKNip07Signer();

        ndk.signer = signer;
        signer.user().then((user) => {
            user.ndk = ndk;
            user.fetchProfile().then((eventSet) => {
                console.log(user);
                hexpubkey = ndk.fetchEvents({ kinds: [1], authors: [user.hexpubkey] });;
                userProfile = user.profile as NDKUserProfile;
            })
        });
    }

</script>

<div class="max-w-4xl mx-auto px-8 mt-12">
    <h1 class="text-6xl font-semibold text-white">Nostr Start</h1>

    <button on:click={login} class="bg-white text-gray-800 py-2 px-3 rounded-md mt-8">Login with Nostr</button>
</div>

<div class="max-w-4xl mx-auto px-8 text-white mt-8">
    {#if userProfile}
    <div class="border border-gray-800 rounded-2xl py-6 px-8 mb-8">
        <p>
            <img src={userProfile.image} class="rounded-full w-48 h-48 p-4" alt="fdsa" />    
        </p>
        <h2 class="text-5xl mb-4">{userProfile.name}</h2>
        <p>{userProfile.about}</p>
    </div>
    {/if}

    {#if hexpubkey}
        {#await hexpubkey then events}
            {#each Array.from(events) as event}
                <div class="border border-gray-800 rounded-2xl py-6 px-8 my-6">
                    <p>{event.content}</p>
                </div>
            {/each}
        {/await}
    {/if}
</div>

<style lang="postcss">
    .eventBlock {
        padding: 10px;
        border: 1px black;
        border-radius: 10px; 
        margin-bottom: 4px;
    }
    :global(html) {
        background-color: black;
    }
</style>