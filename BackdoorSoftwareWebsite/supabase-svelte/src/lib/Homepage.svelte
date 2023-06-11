<script lang="ts">
    import {onDestroy, onMount} from "svelte";
    import Router, {push} from "svelte-spa-router";
    import type {AuthSession} from "@supabase/supabase-js";
    import MessageCard from "../components/MessageCard.svelte";
    import ChatCard from "../components/ChatCard.svelte";
    import type {message} from "../types/custom.ts";
    import {supabase} from "../supabaseClient.ts";


    export let session: AuthSession;

    let allMessages: message[] = [];
    let newMessage: string = "";
    let messagesWatcher;

    let loading = false
    let username: string | null = null
    let website: string | null = null
    let avatarUrl: string | null = null

    onMount(async () => {
        getProfile();

        ({data: allMessages} = await getAllMessages());

        messagesWatcher = supabase
            .channel("custom-all-channel")
            .on("postgres_changes", {event: "*", schema: "public", table: "messages"}, async () => {
                ({data: allMessages} = await getAllMessages());
            })
            .subscribe();
    })

    const getProfile = async () => {
        try {
            loading = true
            const {user} = session

            const {data, error, status} = await supabase
                .from('profiles')
                .select('username, website, avatar_url')
                .eq('id', user.id)
                .single()

            if (error && status !== 406) throw error

            if (data) {
                username = data.username
                website = data.website
                avatarUrl = data.avatar_url
            }
        } catch (error) {
            if (error instanceof Error) {
                alert(error.message)
            }
        } finally {
            loading = false
        }
    }

    const updateProfile = async () => {
        try {
            loading = true
            const {user} = session

            const updates = {
                id: user.id,
                username,
                website,
                avatar_url: avatarUrl,
                updated_at: new Date().toISOString(),
            }

            let {error} = await supabase.from('profiles').upsert(updates)

            if (error) {
                throw error
            }
        } catch (error) {
            if (error instanceof Error) {
                alert(error.message)
            }
        } finally {
            loading = false
        }
    }

    async function getAllMessages() {
        return await supabase.from("messages").select("*").eq("chat_id", 1);
    }

    onDestroy(() => {
        messagesWatcher?.unsubscribe();
    });

    async function sendMessage() {
        const {data, error} = await supabase.from("messages").insert([
            {
                chat_id: 1,
                author_id: session.user.email,
                content: newMessage,
            },
        ]);
    }
</script>
<div>
    <h1>Backdoor software</h1>
    <h2>Welkom</h2>
</div>
<form on:submit|preventDefault={updateProfile} class="form-widget">
    <div>Email: {session.user.email}</div>
    <ChatCard>
        <div class="flex flex-col space-y-1 p-2">
            {#each allMessages as message}
                <MessageCard own={message.author_id === session.user.email}>
                    {message.content}
                </MessageCard>
            {/each}
        </div>
    </ChatCard>
    <div class="absolute bottom-0 flex w-full space-x-4">
        <input bind:value={newMessage} type="text" placeholder="Message" class="flex-1"/>
        <button on:click={sendMessage} class="bg-gray-200 rounded p-2 hover:bg-gray-300">Send</button>
    </div>


    <button type="button" class="button block" on:click={() => supabase.auth.signOut()}>
        Sign Out
    </button>
</form>
