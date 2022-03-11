<script>
    import { auth, db } from '../services/firebase';
    import router from 'page';
    import Chat from '../components/Chat.svelte';
    import { onDestroy } from 'svelte';

    let user;
    let messages = [];
    let cooldown = false;

    auth.onAuthStateChanged((u) => (user = u));

    $: {
        // user === null is explicitly called instead of !user because we need firebase to decide what the user is, and not us, so we dont initialise user up there.
        if (user === null) router.redirect('/auth?action=login&next=%2Fchat');
    }

    const unsubscribe = db.collection('messages').onSnapshot((snapshot) => {
        snapshot.docChanges().forEach((change) => {
            if (change.type === 'added') {
                messages = [...messages, change.doc.data()];
                setTimeout(() => {
                    if (document.getElementById('scroll-to'))
                        document.getElementById('scroll-to').scrollIntoView({ behavior: 'smooth' });
                }, 500);
            }
        });
    });

    function messageSubmit(e) {
        if (e.key.toLowerCase() !== 'enter') return;
        if (cooldown) return;
        const message = (document.getElementById('message-input').value || '').trim();
        if (!message) return;

        document.getElementById('message-input').value = '';
        cooldown = true;
        setTimeout(() => (cooldown = false), 3000);

        db.collection('messages').add({
            message,
            email: user.email,
            photoURL: user.photoURL,
            uid: user.uid,
            createdAt: Date.now(),
        });
    }

    onDestroy(unsubscribe);

    function logout() {
        if (auth.currentUser) {
            auth.signOut()
                .then(() => {})
                .catch((e) => {
                    throw new Error(e);
                });
        }
    }
</script>

{#if typeof user === 'undefined'}
    <p class="w3-center w3-section"><i class="fas fa-spinner w3-spin fa-3x" /> Loading</p>
{:else if user}
    <h1 class="w3-jumbo w3-center">F80 DATA</h1>
    <p class="w3-center">Live data</p>
    <p class="w3-center"><button class="w3-button w3-blue" on:click={logout}>Logout</button></p>

    <br />
    <div
        class="w3-container w3-border w3-border-gray"
        style="margin: 0 auto; width: 60%; height: 600px; overflow-y: auto;"
    >
        <br />
        {#if messages.length > 0}
            {#each messages as m}
                <Chat {...m} self={user.uid === m.uid} />
            {/each}
        {:else}
            <p class="w3-center w3-text-gray">Looks like nobody's sent a message. Be the first!</p>
        {/if}
        <!-- Dummy element used to scroll chat -->
        <br id="scroll-to" />
    </div>
    <input
        on:keydown={messageSubmit}
        type="text"
        style="margin: 0 auto; width: 60%; margin-top: -1px"
        placeholder={cooldown ? '3 second cooldown' : 'Enter message and press enter'}
        class="w3-input w3-border w3-border-gray {cooldown && 'w3-pale-red'}"
        id="message-input"
    />
    <br />
{:else}
    <p class="w3-center w3-section">Not logged in!</p>
{/if}