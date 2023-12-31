<script>
  import { onMount } from 'svelte'
  import {active_heading, active_tracker} from '$lib/js/stores'
  import {browser} from '$app/environment'
  export let allowedHeadings = ['h2', 'h3', 'h4', 'h5', 'h6']

  let scrollY
  let headings = []
  let scrollDirection = 'down';
  let progressBar;

  // Store the previous scroll position
  let lastScrollY = scrollY;

  // Update scrollY and scrollDirection whenever the window scrolls
  $: if(browser){
    scrollY = window.scrollY;
    scrollDirection = scrollY > lastScrollY ? 'down' : 'up';
    lastScrollY = scrollY;
  }

  onMount(() => {
    updateHeadings()
    setActiveHeading()
  })

  function updateHeadings() {
    const nodes = [
      // Exclude h1 as those should be reserved for the post title
      ...document.querySelectorAll(`article :where(${allowedHeadings.join(', ')}):not(#__sections)`)
    ]
    const depths = nodes.map((node) => Number(node.nodeName[1]))
    const minDepth = Math.min(...depths)

    // Initialize values for "depths" of headings
    let d0 = 1
    let d1 = 1
    let d2 = 1

    for (let i = 0; i < nodes.length; i++) {
        let prefix;
        let depth = depths[i] - minDepth;

        if(depth === 0) {
          // Reset levels 1 and 2 if previous depth was not 0
          if(depths[i - 1] !== 0) {
            d1 = 1
            d2 = 1
          }
          prefix = d0.toString()
          d0+=1
        } else if(depth === 1) {
          prefix = (d0 - 1).toString() + '.' + d1.toString()
          d1+=1
        } else if(depth === 2) {
          prefix = (d0 - 1).toString() + '.' + (d1 - 1) + '.' + d2
          d2+=1
        }

        headings[i] = {
          title: nodes[i].innerText,
          depth,
          node: nodes[i],
          prefix: prefix,
        }
    }
  }

const scrollMarginTop = 105; // Adjust this value as needed
const setProgressBar = () => {
        if(browser) {
          let scrollDist = document.documentElement.scrollTop || document.body.scrollTop;
          let progressWidth = (scrollDist / (document.body.scrollHeight - document.documentElement.clientHeight)) * 100;
          if (progressBar) {
            progressBar.style.width = progressWidth + "%";
            }
        }
      }

function setActiveHeading() {
  let activeIndex = 0;
  for (let i = 0; i < headings.length; i++) {
    if (headings[i].node.getBoundingClientRect().top <= scrollMarginTop) {
      activeIndex = i;
    } else {
      break;
    }
  }
  $active_heading = headings[activeIndex];
  active_tracker.set(headings[0].node.getBoundingClientRect().top <= scrollMarginTop);
  setProgressBar();
}
</script>

<nav class="fixed top-0 left-1/2 right-1/2 z-10 -translate-x-1/2 w-screen bg-surface-100-800-token border-surface-200-700-token border-b transition-transform duration-300 ease-out {$active_tracker ? "translate-y-0" : "translate-y-[-100%]"}">
  <div class="text-sm p-4 mx-auto w-fit"> {$active_heading.title} </div>
  <div id="progressBar" class="absolute top-full h-0.5 bg-secondary-500" bind:this={progressBar}></div>
</nav>

<svelte:window on:load={setProgressBar} on:scroll={setActiveHeading} bind:scrollY={scrollY} />

{#if headings.length > 0}
    <nav class="border border-surface-200-700-token bg-surface-100-800-token p-4 text-surface-900-50-token transition-colors duration-100 rounded-container">
      <ul class="collapse-content">
        {#each headings as heading}
            <li
            class="heading list-none my-1 text-sm font-normal transition-all"
            style={`--depth: ${heading.depth}`}
            >
            <span class="font-bold">{heading.prefix}</span>
            <a class="hover:underline" href={`#${heading.node.id}`}>{heading.title}</a>
            </li>
        {/each}
        </ul>  
    </nav>
{/if}

<style>
  .heading {
        margin-left: calc(var(--depth, 0) * 0.75rem);
    }
</style>
  
<!-- SVElTE TOC (https://github.com/janosh/svelte-toc/issues/36)
  <script lang="ts">
  import { onMount } from 'svelte'
  import { blur, type BlurParams } from 'svelte/transition'

  export let activeHeading: HTMLHeadingElement | null = null
  export let activeHeadingScrollOffset: number = 100
  export let activeTocLi: HTMLLIElement | null = null
  export let breakpoint: number = 1000
  export let desktop: boolean = true
  export let flashClickedHeadingsFor: number = 1500
  export let getHeadingIds = (node: HTMLHeadingElement): string => node.id
  export let getHeadingLevels = (node: HTMLHeadingElement): number =>
    Number(node.nodeName[1]) // get the number from H1, H2, ...
  export let getHeadingTitles = (node: HTMLHeadingElement): string =>
    node.textContent ?? ``
  // the result of document.querySelectorAll(headingSelector). can be useful for binding
  export let headings: HTMLHeadingElement[] = []
  export let headingSelector: string = `:is(h2, h3, h4):not(.toc-exclude)`
  export let hide: boolean = false
  export let autoHide: boolean = true
  export let keepActiveTocItemInView: boolean = true
  export let minItems: number = 0
  export let open: boolean = false
  export let openButtonLabel: string = `Open table of contents`
  export let pageBody: string | HTMLElement = `body`
  export let scrollBehavior: 'auto' | 'smooth' = `smooth`
  export let title: string = `On this page`
  export let titleTag: string = `h2`
  export let tocItems: HTMLLIElement[] = []
  export let warnOnEmpty: boolean = true
  export let blurParams: BlurParams | null = { duration: 200 }

  let window_width: number

  let aside: HTMLElement
  let nav: HTMLElement
  let scroll_id: number
  $: levels = headings.map(getHeadingLevels)
  $: minLevel = Math.min(...levels)
  $: desktop = window_width > breakpoint

  function close(event: MouseEvent) {
    if (!aside.contains(event.target as Node)) open = false
  }

  // (re-)query headings on mount and on route changes
  function requery_headings() {
    if (typeof document === `undefined`) return // for SSR
    headings = [...document.querySelectorAll(headingSelector)] as HTMLHeadingElement[]
    set_active_heading()
    if (headings.length === 0) {
      if (warnOnEmpty) {
        console.warn(
          `svelte-toc found no headings for headingSelector='${headingSelector}'. ${
            autoHide ? `Hiding` : `Showing empty`
          } table of contents.`
        )
      }
      if (autoHide) hide = true
    } else if (hide && autoHide) {
      hide = false
    }
  }

  requery_headings()

  onMount(() => {
    if (typeof pageBody === `string`) {
      pageBody = document.querySelector(pageBody) as HTMLElement

      if (!pageBody) {
        throw new Error(`Could not find page body element: ${pageBody}`)
      }
    }
    const mutation_observer = new MutationObserver(requery_headings)
    mutation_observer.observe(pageBody, { childList: true, subtree: true })
    return () => mutation_observer.disconnect()
  })
  function set_active_heading() {
    let idx = headings.length
    while (idx--) {
      const { top } = headings[idx].getBoundingClientRect()

      // loop through headings from last to first until we find one that the viewport already
      // scrolled past. if none is found, set make first heading active
      if (top < activeHeadingScrollOffset || idx === 0) {
        activeHeading = headings[idx]
        activeTocLi = tocItems[idx]
        // this annoying hackery to wait for scroll end is necessary because scrollend event only has 2%
        // browser support https://stackoverflow.com/a/57867348 and Chrome doesn't support multiple
        // simultaneous scrolls, smooth or otherwise (https://stackoverflow.com/a/63563437)
        clearTimeout(scroll_id)
        scroll_id = window.setTimeout(() => {
          if (keepActiveTocItemInView && activeTocLi) {
            // get the currently active ToC list item
            // scroll the active ToC item into the middle of the ToC container
            nav.scrollTo?.({
              top: activeTocLi?.offsetTop - nav.offsetHeight / 2,
              behavior: `smooth`,
            })
          }
        }, 50)
        return // exit while loop if updated active heading
      }
    }
  }

  const handler = (node: HTMLHeadingElement) => (event: MouseEvent | KeyboardEvent) => {
    if (event instanceof KeyboardEvent && ![`Enter`, ` `].includes(event.key)) return
    open = false
    node.scrollIntoView({ behavior: scrollBehavior, block: `start` })

    const id = getHeadingIds && getHeadingIds(node)
    if (id) history.replaceState({}, ``, `#${id}`)

    if (flashClickedHeadingsFor) {
      node.classList.add(`toc-clicked`)
      setTimeout(() => node.classList.remove(`toc-clicked`), flashClickedHeadingsFor)
    }
  }
</script>

<svelte:window
  bind:innerWidth={window_width}
  on:scroll={set_active_heading}
  on:click={close}
/>

<aside
  class="toc"
  class:desktop
  class:hidden={hide}
  class:mobile={!desktop}
  bind:this={aside}
  hidden={hide}
  aria-hidden={hide}
>
  {#if !open && !desktop && headings.length >= minItems}
    <button
      on:click|preventDefault|stopPropagation={() => (open = true)}
      aria-label={openButtonLabel}
    >
      <slot name="open-toc-icon">
        <svg class="w-1em" viewBox="0 0 20 20" fill="currentColor">
          <path
            d="M3 5a1 1 0 0 1 1-1h12a1 1 0 1 1 0 2H4a1 1 0 0 1-1-1zm0 5a1 1 0 0 1 1-1h12a1 1 0 1 1 0 2H4a1 1 0 0 1-1-1zm0 5a1 1 0 0 1 1-1h12a1 1 0 1 1 0 2H4a1 1 0 0 1-1-1z"
          />
        </svg>
      </slot>
    </button>
  {/if}
  {#if open || (desktop && headings.length >= minItems)}
    <nav transition:blur={blurParams} bind:this={nav}>
      {#if title}
        <slot name="title">
          <svelte:element this={titleTag} class="toc-title toc-exclude">
            {title}
          </svelte:element>
        </slot>
      {/if}
      <ol>
        {#each headings as heading, idx}
          <li
            style:margin="0 0 0 {levels[idx] - minLevel}em"
            style:font-size="{2 - 0.2 * (levels[idx] - minLevel)}ex"
            class:active={activeHeading === heading}
            on:click={handler(heading)}
            on:keyup={handler(heading)}
            bind:this={tocItems[idx]}
            role="menuitem"
          >
            <slot name="toc-item" {heading} {idx}>
              {getHeadingTitles(heading)}
            </slot>
          </li>
        {/each}
      </ol>
    </nav>
  {/if}
</aside>

<style>
  :where(aside.toc) {
    box-sizing: border-box;
    height: max-content;
    overflow-wrap: break-word;
    font-size: var(--toc-font-size);
    min-width: var(--toc-min-width);
    width: var(--toc-width);
    z-index: var(--toc-z-index, 1);
  }
  :where(aside.toc > nav) {
    overflow: var(--toc-overflow, auto);
    overscroll-behavior: contain;
    max-height: var(--toc-max-height, 90vh);
    padding: var(--toc-padding, 1em 1em 0);
  }
  :where(aside.toc > nav > ol) {
    list-style: var(--toc-ol-list-style, none);
    padding: var(--toc-ol-padding, 0);
    margin: var(--toc-ol-margin);
  }
  :where(.toc-title) {
    padding: var(--toc-title-padding);
    margin: var(--toc-title-margin);
  }
  :where(aside.toc > nav > ol > li) {
    cursor: pointer;
    color: var(--toc-li-color);
    border: var(--toc-li-border);
    border-radius: var(--toc-li-border-radius);
    margin: var(--toc-li-margin);
    padding: var(--toc-li-padding, 2pt 4pt);
  }
  :where(aside.toc > nav > ol > li:hover) {
    color: var(--toc-li-hover-color, cornflowerblue);
    background: var(--toc-li-hover-bg);
  }
  :where(aside.toc > nav > ol > li.active) {
    background: var(--toc-active-bg, cornflowerblue);
    color: var(--toc-active-color, white);
    font-weight: var(--toc-active-font-weight);
    border: var(--toc-active-border);
    border-width: var(--toc-active-border-width);
    border-radius: var(--toc-active-border-radius, 2pt);
  }
  :where(aside.toc > button) {
    border: none;
    bottom: 0;
    cursor: pointer;
    font-size: 2em;
    line-height: 0;
    position: absolute;
    right: 0;
    z-index: 2;
    padding: var(--toc-mobile-btn-padding, 2pt 3pt);
    border-radius: var(--toc-mobile-btn-border-radius, 4pt);
    background: var(--toc-mobile-btn-bg, rgba(255, 255, 255, 0.2));
    color: var(--toc-mobile-btn-color, black);
  }
  :where(aside.toc > nav) {
    position: relative;
  }
  :where(aside.toc > nav > .toc-title) {
    margin-top: 0;
  }

  :where(aside.toc.mobile) {
    position: fixed;
    bottom: var(--toc-mobile-bottom, 1em);
    right: var(--toc-mobile-right, 1em);
  }
  :where(aside.toc.mobile > nav) {
    border-radius: 3pt;
    right: 0;
    z-index: -1;
    box-sizing: border-box;
    background: var(--toc-mobile-bg, white);
    width: var(--toc-mobile-width, 18em);
    box-shadow: var(--toc-mobile-shadow);
    border: var(--toc-mobile-border);
  }

  :where(aside.toc.desktop) {
    margin: var(--toc-desktop-aside-margin);
  }
  :where(aside.toc.desktop) {
    position: sticky;
    background: var(--toc-desktop-bg);
    margin: var(--toc-desktop-nav-margin);
    max-width: var(--toc-desktop-max-width);
    top: var(--toc-desktop-sticky-top, 2em);
  }
</style>

-->