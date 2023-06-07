<script lang="ts">
    import { onMount } from "svelte";
    import type { AuthSession } from "@supabase/supabase-js";
    import { supabase } from "../supabaseClient.ts";

    export let session: AuthSession;

    let loading = false
    let username: string | null = null
    let website: string | null = null
    let avatarUrl: string | null = null

    onMount(() => {
        getProfile()
    })

    const getProfile = async () => {
        try {
            loading = true
            const { user } = session

            const { data, error, status } = await supabase
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
            const { user } = session

            const updates = {
                id: user.id,
                username,
                website,
                avatar_url: avatarUrl,
                updated_at: new Date().toISOString(),
            }

            let { error } = await supabase.from('profiles').upsert(updates)

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
</script>
<div>
    <h1>Backdoor software</h1>
    <h2>Welkom</h2>
</div>
<form on:submit|preventDefault={updateProfile} class="form-widget">
    <div>Profile picture: {session.user.profileurl}</div>
    <div>Email: {session.user.email}</div>
    <button type="button" class="button block" on:click={() => supabase.auth.signOut()}>
        Sign Out
    </button>
</form>

