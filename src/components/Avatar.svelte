<script lang="ts">
  import { createEventDispatcher } from 'svelte'
  import { supabase } from '../lib/supabase'

  export let url: string | null

  let avatarUrl: string | null = null
  let uploading = false
  let files: FileList
  let loading = false;

  const dispatch = createEventDispatcher()

  const downloadImage = async (path: string) => {
    try {
      loading = true
      
      const { data, error } = await supabase.storage.from('avatars').download(path)

      if (error) {
        throw error
      }

      const url = URL.createObjectURL(data)
      avatarUrl = url
    } catch (error) {
      if (error instanceof Error) {
        console.log('Error downloading image: ', error.message)
      }
    }
    finally{
      loading = false
    }
  }

  const uploadAvatar = async () => {
    try {
      uploading = true

      if (!files || files.length === 0) {
        throw new Error('You must select an image to upload.')
      }

      const file = files[0]
      const fileExt = file.name.split('.').pop()
      const filePath = `${Math.random()}.${fileExt}`

      const { error } = await supabase.storage.from('avatars').upload(filePath, file)

      if (error) {
        throw error
      }

      url = filePath
      dispatch('upload')
    } catch (error) {
      if (error instanceof Error) {
        alert(error.message)
      }
    } finally {
      uploading = false
    }
  }

  $: if (url) downloadImage(url)
</script>

<div aria-live="polite">
  {#if loading}
    <span class="loading loading-spinner loading-lg"></span>
  {:else}
    {#if avatarUrl} 
      <div class="mb-4">
        <img src={avatarUrl} alt={avatarUrl ? 'Avatar' : 'No image'} class="max-w-64" /> 
      </div>
    {/if}
  {/if}
  <div>
    <label class="btn btn-wide" for="single">
      {uploading ? 'Uploading ...' : 'Upload avatar'}
    </label>
    <span style="display:none">
      <input
        type="file"
        id="single"
        accept="image/*"
        bind:files
        on:change="{uploadAvatar}"
        disabled="{uploading}"
      />
    </span>
  </div>
</div>