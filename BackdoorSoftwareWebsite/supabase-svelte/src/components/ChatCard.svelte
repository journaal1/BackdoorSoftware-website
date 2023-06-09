<script lang="ts">
    import { supabase } from "../supabaseClient.ts";
    import {AuthSession} from "@supabase/supabase-js";

    export let session: AuthSession;

    function getChatUserNames () {
        let userEmail = session.user.email;
        return userEmail.toString();
    }
    const chatUserNames = getChatUserNames();
</script>

<div class="p-4 bg-gray-100 shadow rounded">
    <h2 class="text-large font-semibold">Chat with: {chatUserNames}</h2>
</div>