<script lang="ts">
    import { readablestreamStore } from '$lib/readablestreamStore';
    import { fly } from 'svelte/transition';
    import Typingindecator from '$lib/typingindecator.svelte';
    import Markdown from '@magidoc/plugin-svelte-marked';

    const response = readablestreamStore();
    let chat_history: { role: 'user' | 'assistant'; content: string }[] = [];

    // response.subscribe((val) => console.log(`subscribed: ${val.text}`));

    $: test_text = $response.text;

    async function handleSubmit(this: HTMLFormElement) {
        if ($response.loading) return;

        const formData: FormData = new FormData(this);
        const message = formData.get('message') as string;

        if (message == '') return;

        chat_history = [...chat_history, { role: 'user', content: message }];

        try {
            const answer = response.request(
                new Request('https://aoji.mooo.com/ollama/api/chat', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ model: 'llama3.2:latest', messages: chat_history })
                })
            );

            this.reset();

            chat_history = [
                ...chat_history,
                { role: 'assistant', content: (await answer) as string }
            ];
        } catch (err) {
            chat_history = [
                ...chat_history,
                { role: 'assistant', content: `Error in front: ${err}` }
            ];
        }
    }
</script>

<svelte:head>
    <title>Chat with Llama 3.2</title>
    <meta name="description" content="Chat with Llama3.2" />
</svelte:head>

<div class="flex flex-col items-center space-y-4">
    <form class="chat-wrapper" on:submit|preventDefault={handleSubmit}>
        <div class="flex aspect-square w-full flex-col space-y-2 overflow-y-auto text-sm">
            {#await new Promise((res) => setTimeout(res, 400)) then _}
                <div class="flex">
                    <div in:fly={{ y: 50, duration: 400 }} class="assistant-chat">
                        Hello! How can I help you today?
                    </div>
                </div>
            {/await}

            {#each chat_history as chat}
                {#if chat.role == 'user'}
                    <div class="flex justify-end">
                        <div in:fly={{ y: 50, duration: 600 }} class="user-chat">
                            {chat.content}
                        </div>
                    </div>
                {:else}
                    <div class="flex">
                        <div in:fly={{ y: 50, duration: 600 }} class="assistant-chat">
                            <Markdown source={chat.content} />
                            <!-- {chat.content} -->
                        </div>
                    </div>
                {/if}
            {/each}

            {#if $response.loading}
                {#await new Promise((res) => setTimeout(res, 400)) then _}
                    <div class="flex">
                        {#if $response.text == ''}
                            <Typingindecator />
                        {/if}
                        {#if $response.text != ''}
                            <div class="flex flex-col">
                                <div in:fly={{ y: 50, duration: 600 }} class="assistant-chat">
                                    <Markdown source={$response.text} />
                                </div>

                                <div class="m-1 w-4">
                                    <svg
                                        class="h-4 w-4 animate-spin text-neutral-600 dark:text-neutral-400"
                                        xmlns="http://www.w3.org/2000/svg"
                                        fill="none"
                                        viewBox="0 0 24 24"
                                    >
                                        <circle
                                            class="opacity-25"
                                            cx="12"
                                            cy="12"
                                            r="10"
                                            stroke="currentColor"
                                            stroke-width="4"
                                        />
                                        <path
                                            class="opacity-75"
                                            fill="currentColor"
                                            d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
                                        />
                                    </svg>
                                </div>
                            </div>
                        {/if}
                    </div>
                {/await}
            {/if}

            <span class="flex flex-row space-x-4">
                <input
                    type="text"
                    placeholder="Type your message..."
                    name="message"
                    class="chat-message"
                />
                <button type="submit" class="chat-send"> Send </button>
            </span>
        </div>
    </form>
</div>

<style lang="postcss">
    .chat-wrapper {
        @apply flex max-w-6xl flex-col space-y-4 md:min-w-[28rem] lg:min-w-[32rem] xl:min-w-[36rem];
    }

    .assistant-chat {
        @apply prose prose-sm prose-pre:font-mono prose-pre:border prose-pre:bg-white prose-code:border-gray-300 my-0 max-w-xs rounded-lg bg-gray-200 px-4 py-2 text-gray-800;
    }

    .user-chat {
        @apply prose prose-sm prose-pre:font-mono prose-pre:border prose-pre:bg-white prose-code:border-gray-300 my-0 max-w-xs rounded-lg bg-[#FF3E00] px-4 py-2 text-white;
    }

    .chat-message {
        @apply block w-full rounded-md border-0 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-black sm:text-sm sm:leading-6;
    }

    .chat-send {
        @apply block items-center rounded-md border border-transparent bg-neutral-800 px-4 py-2 text-sm font-medium text-white shadow-sm hover:bg-black focus:outline-none focus:ring-2 focus:ring-black focus:ring-offset-2;
    }
</style>
