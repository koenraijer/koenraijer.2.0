<script>
    export let href;
    export let classes="anchor";
    let target = "_self";
    let rel = "";

    // Check if link is external
    let isExternal = href.includes("http") || href.includes("mailto");

    if (isExternal) {
        target = "_blank";
        rel = "noopener noreferrer nofollow";
    }

</script>


<a class={classes + " inline-flex my-0"} href={href} {target} {rel}>
    <slot/>
    {#if isExternal}
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-3 h-3 inline-flex self-start text-inherit stroke-2 mt-1">
            <path stroke-linecap="round" stroke-linejoin="round" d="M13.5 6H5.25A2.25 2.25 0 0 0 3 8.25v10.5A2.25 2.25 0 0 0 5.25 21h10.5A2.25 2.25 0 0 0 18 18.75V10.5m-10.5 6L21 3m0 0h-5.25M21 3v5.25" />
        </svg>
    {/if}
</a>
