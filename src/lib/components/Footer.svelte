<script>
    import * as info from "$lib/js/info.js"
    import ToTop from "$lib/components/ToTopButton.svelte" 
</script>

<div class="gap-2 mb-6 mt-12 flex w-full mx-auto justify-between px-6 sm:px-8 items-start">
    <div class="">
        <a href="/" class="hover:underline h-fit font-semibold">Koen Raijer</a>
        <div class="flex flex-col justify-between text-surface-500 text-base">
            <div class="flex flex-col sm:flex-row gap-x-4">
                <a href="/books" class="hover:underline h-fit">Books</a>
                <a href="/changelog" class="hover:underline h-fit">Changelog</a>
                <a href="https://github.com/koenraijer/koenraijer.2.0" class="hover:underline" target="_self" rel="nofollow noopener noreferrer">Source code</a>
            </div>
            <span class="whitespace-nowrap sm:w-fit w-full text-xs mt-2 text-surface-400">
                © {new Date().getFullYear()} <a href="/" class="hover:underline">{info.title}</a>
            </span>
        </div>
    </div>
    <div class="text-surface-500 space-y-0">
        <ToTop/>
    </div>
</div>