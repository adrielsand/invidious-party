<script>
    import { onMount } from 'svelte'

	import { chooseInstance } from '$lib/_helper'
	import { chosen } from '$lib/_memoryStore'
	import { instances } from '$lib/_localStore'

    import AsyncError from '$lib/AsyncError.svelte'
    import AsyncLoading from '$lib/AsyncLoading.svelte'
    import Chat from '$lib/Chat.svelte'
    import Video from '$lib/Video.svelte'
    import VideoInfo from '$lib/VideoInfo.svelte'

    let counter = 0
    let retry = false
    let videoID

    // let videoTMP = JSON.parse()

    onMount(async () => videoID = window.location.search.split('=')[1])
    
	const fetchVideo = async(instance, videoID) => {
        try {
            ///api/v1/videos/aqz-KE-bpKQ?fields=videoId,title,description
            const req = await fetch(`https://${instance}/api/v1/videos/${videoID}`)
            const res = await req.json()
            counter ++
            return res
		} catch(err) {
            retry = true
		}
	}

	$: if(retry) {
        $chosen = chooseInstance($instances)
        retry = false
    }
</script>

{#await fetchVideo($chosen, videoID)}
    <AsyncLoading chosen={$chosen} on:rotate={() => $chosen = chooseInstance($instances)} />
    <div class="top">
        <div class="video">
            <!-- {#if $chosen} -->
                <!-- <Video chosen={$chosen} {videoID} on:error={() => $chosen = chooseInstance($store.instances)} /> -->
            <!-- {/if} -->
        </div>
        <div class="chat">
            <Chat roomID={videoID} />
        </div>
    </div>
{:then videoAPI}
    <div class="top">
        <div class="video">
            {#if $chosen}
                <Video chosen={$chosen} {videoAPI} on:error={$chosen = chooseInstance($instances)} />
            {/if}
        </div>
        <div class="chat">
            <Chat roomID={videoID} />
        </div>
    </div>
    <!-- <Videoo chosen={$chosen} {video} /> -->
    <div class="bottom">
        <div class="info">
            <VideoInfo chosen={$chosen} {videoAPI} on:rotate={$chosen = chooseInstance($instances)} />
        </div>
        <div class="related">
            <!--<div class="header">YT RECOMMENDS <img src="data:image/png;base64,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" alt="KEKW"></div>-->
            <!-- <Videos {related} /> -->
        </div>
    </div>
{:catch error}
    <AsyncError {error} on:rotate={$chosen = chooseInstance($instances)} />
{/await}

<style>
.top {
    display: flex;
}
.video {
    width: 100%;
    margin-right: 11px;
    border: 1px solid var(--light-border);
    border-radius: 5px;
}
.chat {
    width: 31.8%;
}
.bottom {
    display: flex;
}
.bottom .info {
    width: 100%;
}
.related {
    width: 31.8%;
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-end;
    border-bottom-right-radius: 5px;
    margin-top: 11px;
    margin-left: 11px;
}
.related .header {
    width: 100%;
    height: calc(50% - 4px);
    padding: 0;
    font-size: 1.618em;
    background: var(--bg-dark-second);
    display: flex;
    align-items: center;
    justify-content: space-evenly;
    border: 1px solid var(--light-border);
    border-radius: 5px;
}
</style>