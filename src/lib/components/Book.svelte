<script>
    import { Ratings } from '@skeletonlabs/skeleton';
    import { formatDate } from '$lib/js/utils.js';
    export let book;
</script>
<div class="flex flex-col bg-transparent overflow-hidden border justify-between rounded-container border-surface-300-600-token">
    <div class="flex flex-col place-items-start">
        <!--Header-->
        <div class="flex justify-between p-6 py-4 sm:py-6 sm:p-8 border-b border-surface-300-600-token flex-wrap gap-y-2 place-items-center w-full">
            <p class="date text-surface-500-400-token">
            {book['Date Finished'] ? `${formatDate(book['Date Finished'])}` : "Unknown date"}
            </p>
            <div class="w-fit">
                {#if book['Score (0-10)']}
                    <Ratings value={book['Score (0-10)'] / 2} max={5}>
                        <svelte:fragment slot="empty">
                            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" class="w-5 h-5" fill="currentColor" viewBox="0 0 256 256"><path d="M239.2,97.29a16,16,0,0,0-13.81-11L166,81.17,142.72,25.81h0a15.95,15.95,0,0,0-29.44,0L90.07,81.17,30.61,86.32a16,16,0,0,0-9.11,28.06L66.61,153.8,53.09,212.34a16,16,0,0,0,23.84,17.34l51-31,51.11,31a16,16,0,0,0,23.84-17.34l-13.51-58.6,45.1-39.36A16,16,0,0,0,239.2,97.29Zm-15.22,5-45.1,39.36a16,16,0,0,0-5.08,15.71L187.35,216v0l-51.07-31a15.9,15.9,0,0,0-16.54,0l-51,31h0L82.2,157.4a16,16,0,0,0-5.08-15.71L32,102.35a.37.37,0,0,1,0-.09l59.44-5.14a16,16,0,0,0,13.35-9.75L128,32.08l23.2,55.29a16,16,0,0,0,13.35,9.75L224,102.26S224,102.32,224,102.33Z"></path></svg>
                        </svelte:fragment>
                        <svelte:fragment slot="half">
                            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="currentColor" class="w-5 h-5" viewBox="0 0 256 256">
                                <defs>
                                    <!-- Define a clipping path that covers the left half of the star -->
                                    <clipPath id="left-half">
                                        <rect x="0" y="0" width="128" height="256" />
                                    </clipPath>
                                </defs>
                                <!-- Draw the filled star and clip it to the left half -->
                                <path d="M234.5,114.38l-45.1,39.36,13.51,58.6a16,16,0,0,1-23.84,17.34l-51.11-31-51,31a16,16,0,0,1-23.84-17.34L66.61,153.8,21.5,114.38a16,16,0,0,1,9.11-28.06l59.46-5.15,23.21-55.36a15.95,15.95,0,0,1,29.44,0h0L166,81.17l59.44,5.15a16,16,0,0,1,9.11,28.06Z" clip-path="url(#left-half)"/>
                                <!-- Draw the outlined star on top -->
                                <path d="M239.2,97.29a16,16,0,0,0-13.81-11L166,81.17,142.72,25.81h0a15.95,15.95,0,0,0-29.44,0L90.07,81.17,30.61,86.32a16,16,0,0,0-9.11,28.06L66.61,153.8,53.09,212.34a16,16,0,0,0,23.84,17.34l51-31,51.11,31a16,16,0,0,0,23.84-17.34l-13.51-58.6,45.1-39.36A16,16,0,0,0,239.2,97.29Zm-15.22,5-45.1,39.36a16,16,0,0,0-5.08,15.71L187.35,216v0l-51.07-31a15.9,15.9,0,0,0-16.54,0l-51,31h0L82.2,157.4a16,16,0,0,0-5.08-15.71L32,102.35a.37.37,0,0,1,0-.09l59.44-5.14a16,16,0,0,0,13.35-9.75L128,32.08l23.2,55.29a16,16,0,0,0,13.35,9.75L224,102.26S224,102.32,224,102.33Z"></path>
                            </svg>                
                        </svelte:fragment>
                        <svelte:fragment slot="full">
                            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" class="w-5 h-5" fill="currentColor" viewBox="0 0 256 256"><path d="M234.5,114.38l-45.1,39.36,13.51,58.6a16,16,0,0,1-23.84,17.34l-51.11-31-51,31a16,16,0,0,1-23.84-17.34L66.61,153.8,21.5,114.38a16,16,0,0,1,9.11-28.06l59.46-5.15,23.21-55.36a15.95,15.95,0,0,1,29.44,0h0L166,81.17l59.44,5.15a16,16,0,0,1,9.11,28.06Z"></path></svg>
                        </svelte:fragment>
                    </Ratings>
                {:else}
                    <span class="text-surface-400">Unknown score</span>
                {/if}
            </div>
        </div>
        <!--Body-->
        <div class="p-6 sm:p-8 flex flex-col justify-start w-full">
            <!--
            <div class="mx-auto w-fit mb-2">
                <img class="w-32 h-48 object-cover rounded-container" src={"/book_covers/" + book.ISBN + ".webp"} alt={book.Title ? book.Title : ""} />
            </div>
            -->
            <h2 class="text-lg font-semibold pb-2">{book.Title ? book.Title : ""}</h2>

            <span class="text-surface-700-200-token text-lg">{book.Author}</span>
            <!--
                { "ISBN": 62407805, "Added": 1672444800000, "Title": "Never Split the Difference: Negotiating As If Your Life Depended On It", "Author": "Chris Voss", "Date Published": 2016, "Score (0-10)": 4, "Review": null, "Notes": null, "Date Finished": 1688256000000, "Category": "Nonfiction" }
            -->
            <p class="pt-4 text-surface-600-300-token">First published in <i>{book['Date Published'] ? book['Date Published'] : "Unknown date"}</i>.</p>
            {#if book.Notes}
                <div class="px-6 py-4 space-y-2">
                    <p>{book.Notes}</p>
                </div>
            {/if}
        </div>
    </div>
    <!--Footer-->
    <div class="px-6 py-4 flex items-center justify-between flex-wrap gap-y-2 place-items-end"> 
        <span class="inline-flex px-3 py-2 items-center text-token border border-surface-200-700-token rounded-container">
            {book.Category}
        </span>
        <span class="items-center inline-flex py-[0.05rem] px-2">
            {#if book.ISBN}
                <b class="mr-2">ISBN:</b> {book.ISBN}
            {:else}
                No ISBN
            {/if}
        </span>
    </div>
</div>