<script lang="ts">
    // Import the package
    import NDK from "@nostr-dev-kit/ndk";
    //I just wanna connect while in the browser and not broser and node server as svelte does
    import { browser } from '$app/environment';
    import {NDKNip07Signer} from "@nostr-dev-kit/ndk";
    import type {NDKUserProfile} from "@nostr-dev-kit/ndk"
    import { nip19 } from "nostr-tools";

    // Create a new NDK instance with explicit relays
    const ndk = new NDK({
        explicitRelayUrls: ["wss://relay.nostr.band", "wss://relay.damus.io", "wss://purplepag.es"],
    });

    let userProfile: NDKUserProfile
    let nome = null;
    let hexpubkey = null;
    let carlos = null;

    //make suro to not run the code on the server
    if(browser){
        ndk.connect().then(() => {
            console.log("Connected")
        });
    }

    function fetchEventFromSub(){
        const sub = ndk.subscribe({kinds: [1]});
        return carlos = sub;

        sub.on('event', (event) => {
            //console.log(event);
        });

        sub.on('eose', () => {
            console.log('EOSE')
        });

        sub.on('notice', (notice) => {
            console.log(notice);
        });

    }

    const kind0Filter = (pubkey) => ({
        kinds: [0],
        authors: [pubkey],
    });

    async function findProfile(query){

        let coisa = "npub" + query;
        console.log(coisa)
        const decodedNpub = nip19.decode(coisa);
        const pubkey = decodedNpub.data;

        let filter0 = {};
        filter0 = kind0Filter(pubkey);

        nome = ndk.fetchEvent(filter0);
        console.log(nome)
        if (!nome) {
          console.log(false)
        } else {
          console.log(true)
        }
        //return nome.name
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
        })

        //const sub = await ndk.subscribe({kinds: [1]});
        //carlos = sub

        carlos = await ndk.fetchEvents({ kinds: [1] });
        console.log(carlos)

        }

</script>

<div class="mt-12">
    <button on:click={login} class="bg-white text-gray-800 py-2 px-3 rounded-md">Login with Nostr</button>
</div>

<div class="max-w-4xl mx-auto text-white mt-8">
    {#if userProfile}
    <div class="border border-gray-800 rounded-2xl py-6 px-8 mb-8">
        <div class="flex">
            <img src={userProfile.image} class="rounded-full w-32 h-32 p-4" alt="fdsa" />    
        
            <div class="mx-4 my-auto">
                <h2 class="text-3xl mb-2">{userProfile.name}</h2>
                <p>{userProfile.about}</p>
            </div>
        </div>
    </div>
    {/if}

    {#if carlos}
        {#await carlos then events}
        {#each Array.from(events) as event}
            <div class="border border-gray-800 rounded-xl py-6 px-8 my-6">
                <p>{findProfile(event.pubkey)}</p>
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
