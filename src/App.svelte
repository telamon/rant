<script>
// imports
import { derived, writable } from 'svelte/store'
import marked from 'marked'
import Purify from 'dompurify'
import IdentityPane from './IdentityPane.svelte'
import EncryptionSettings from './EncryptionSettings.svelte'
// props
export let card
export let uid
export let rant
export let theme
export let secret
export let editMode
// code
const pickle = derived(card, s => s.pickle || '')
const mdHtml = derived([rant, card], ([$md, $card]) => {
  const subdiv = (k, n) => {
    return !n-- ? k : [
      subdiv(k.slice(0, k.length >> 1), n),
      subdiv(k.slice(k.length >> 1), n)
    ].flat()
  }
  const preprocessed = $md
    // 10kB accurate version: https://github.com/mathiasbynens/emoji-regex/blob/master/index.js
    .replace(/!([^!\n ]{1,6})!/g, '<span class="imgmoji">$1</span>')
  return marked(Purify.sanitize(preprocessed))
    .replace(/\{\{DATE\}\}/gi, new Date($card.date))
    .replace(/\{\{KEY\}\}/gi, `<pre class="pksig">${
        subdiv(uid.sig.pub.toString('hex'), 3).join('\n')
      }</pre>`)
})

const themes = {
  0: 'cyborg',
  1: 'love-letter',
  5: 'spooky',
  6: 'morpheus'
  // ['happy-birthday', 2],
  // ['invitation', 3],
  // ['robin', 4],
}



const toggleState = editMode.update.bind(editMode, s => !s)
const mainClass = derived([theme, editMode], ([t, s]) => `${themes[t] ? themes[t] : themes[0]} ${s ? 'edit' : 'show'}`)

const encVisible = writable(false)
</script>

<main class={$mainClass} on:keyUp={e => e.key === 'Tab' && toggleState()}>
  <!-- controls -->
  <nav>
    <div class="flex row xcenter"><!-- left -->
      <h3 class="brand">1024R</h3>
      <small id="version">v0.6.0-alpha</small>
      {#if $editMode}
        <!-- capacity and indicator -->
        <div class="flex column xcenter">
          <samp>{$card.size} / 1024</samp>
          <div id="capacity">
            <span style={`width: ${$card.size / 10.24}%;`}></span>
          </div>
        </div>
        <code class="compression-indicator">[🗜️{Math.round($card.ratio * 100)}%]</code>
      {:else}
        <IdentityPane identity={uid}/>
      {/if}
    </div>

    <div><!-- middle -->
      <button on:click={toggleState}
              class="uline moss state-toggle">{$editMode ? 'Preview' : 'Edit'}</button>
      {#if $editMode}
        <!--<button class="uline red emo">🌼</button>-->
        <!-- encryption button + indicator-->
        <button class="uline"
                class:cobalt={!$secret.type}
                class:purp={$secret.type}
                on:click={() => $encVisible = true }>Encrypt <span>{$secret.type ? '🔒' : '🔓'}</span></button>
      {/if}
    </div>

    <div class="flex row xcenter"><!-- right -->
      {#if $editMode}
        <!-- Theme choose -->
        <select class="uline red" bind:value={$theme}>
          {#each Object.keys(themes) as t}
            <option value={t}>
            {themes[t]}
            </option>
          {/each}
        </select>
      {:else}
        <button class="uline orange">Share</button>
        <button class="uline red">Clipboard</button>
      {/if}
    </div>
  </nav>

  <!-- editor -->
  {#if $editMode}
    <section id="editor">
      <!-- rows attr is a chrome workaround -->
      <textarea bind:value={$rant} rows="60"></textarea>
    </section>
  {/if}

  <!-- content -->
  <section id="render">
    {@html $mdHtml}
  </section>
  <footer><a href="http://decentlabs.se">1k.Rant copyright © DecentLabs 2020 - License GNU AGPLv3</a> <a href="https://github.com/telamon/rant/">Source</a></footer>
  <EncryptionSettings secret={secret} visible={encVisible}/>
</main>

<style>
  /*
	main {
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}*/
</style>
