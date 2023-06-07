<script lang="ts">
  import { supabase } from '../supabaseClient.ts'

  let loading = false

  const handleDiscordLogin = async () => {
    try {
      loading = true
      const { data, error } = await supabase.auth.signInWithOAuth({
        provider: 'discord',
      })
      console.log(data + "testje");
      if (error) throw error
    } catch (error) {
      if (error instanceof Error) {
        alert(error.message)
      }
    } finally {
      loading = false
    }
  }
</script>

<div class="row flex-center flex">
  <div class="col-6 form-widget" aria-live="polite">
    <h1 class="header">Backdoor software login.</h1>
    <h2>Welcome!</h2>
    <form class="form-widget" on:submit|preventDefault="{handleDiscordLogin}">
      <div>
        <label>Login using discord</label>
      </div>
      <div>
        <button type="submit" class="button block" aria-live="polite" disabled="{loading}">
          <span>{loading ? 'Loading' : 'DISCORD LOGIN'}</span>
        </button>
      </div>
    </form>
  </div>
</div>