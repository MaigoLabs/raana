<script lang="ts">
  import { onMount } from "svelte";
  import rana from "./rana.png"
  import Icon from "@iconify/svelte";

  const duration = 0.3

  // Pos: % from 0 to 1 on the axis perpendicular to the dir
  let pos = 0.5

  // top | bottom | left | right
  const dirs = ["top", "bottom", "left", "right"]
  let dir = "bottom"
  
  // 0: idle, 1: entering, 2: leaving
  let state = 0

  // Reference to the image element
  let mainEl: HTMLElement
  let divEl: HTMLDivElement
  let imgEl: HTMLImageElement

  let quotes = [
    ['不畏迷茫，迷茫着也要砥砺前行。', '迷子でもいい、迷子でも進め！'],
    ['那... 能陪我写一辈子的代码吗？', 'じゃ……一生 コードを書いてくれる？'],
    ['因为我的代码，就是内心的呐喊！', 'だって 私のコードは 心の叫びだから！']
  ]

  // Load and store the quote index
  let quoteIdx = localStorage.quoteIdx ? +localStorage.quoteIdx : -1
  quoteIdx = (quoteIdx + 1) % quotes.length
  localStorage.quoteIdx = quoteIdx

  function getInitTrans() {
    const dirC = dir === 'top' || dir === 'bottom' ? 'Y' : 'X'
    const dirN = dir === 'top' || dir === 'left' ? '-' : ''
    const dirV = dir === 'top' || dir === 'bottom' ? '300px' : '200px'
    return `translate${dirC}(${dirN}${dirV})`
  }

  const sleep = (s: number) => new Promise(resolve => setTimeout(resolve, s * 1000))

  async function enter() {
    console.log("Entering")
    state = 1

    divEl.style.transform = "translate(0, 0)"
    await sleep(duration)

    state = 0
  }

  async function onHover() {
    if (state !== 0 || divEl === undefined) return

    // Animate the image to leave
    state = 2
    divEl.style.transform = getInitTrans()
    await sleep(duration)

    // Randomize the dir and position
    state = 1
    dir = dirs[Math.floor(Math.random() * dirs.length)]
    // Pos between 0.05 and 0.95
    pos = Math.random() * 0.9 + 0.05
    await divStyles()
  }

  async function divStyles() {
    if (divEl === undefined) return

    let obj: any = {}
 
    obj.transform = getInitTrans()
    obj[dir] = '-50px'
    let perpendicular = dir === 'bottom' || dir === 'top' ? 'left' : 'top'
    obj[perpendicular] = `calc(${pos * 100}% - 100px)`
    obj.display = 'block'
    
    // Disable transition for the initial position
    divEl.style.transition = `none`
    divEl.style.cssText = Object.keys(obj).map(key => `${key}: ${obj[key]}`).join('; ')
    await sleep(0.1)
    divEl.style.transition = `transform ${duration}s`

    imgStyles()
    enter()
  }

  function imgStyles() {
    if (imgEl === undefined) return

    let obj: any = {}
    
    obj['rotate'] = dir === 'top' ? '180deg' 
      : dir === 'left' ? '30deg'
      : dir === 'right' ? '-30deg'
      : '0deg'

    if (dir === 'right') {
      obj['transform'] = 'scaleX(-1)'
    }

    imgEl.style.cssText = Object.keys(obj).map(key => `${key}: ${obj[key]}`).join('; ')
  }

  onMount(() => setTimeout(divStyles, 100))

  // This function creates a heart at the click position.
  function createHeart(event: MouseEvent) {
    console.log("Creating heart")
    const heart = document.createElement('div');
    heart.className = 'heart';
    // Position the heart at the cursor's location.
    heart.style.left = `${event.clientX + Math.random() * 20 - 10}px`;
    heart.style.top = `${event.clientY + Math.random() * 20 - 10}px`;
    mainEl.appendChild(heart);
    // Remove the heart element after the animation duration (e.g., 2s)
    setTimeout(() => heart.remove(), 2000);
  }
</script>

<main bind:this={mainEl}>
  <div class="info">
    <h1>半熟迷子工作室</h1>
    <div class="sub">
      <span>Maigo Labs</span>
      <div class="flex-1"></div>
      <a href="https://github.com/MaigoLabs"><Icon icon="fa6-brands:github"/></a>
    </div>
    <div class="quote">
      <div>{quotes[quoteIdx][0]}</div>
      <div>{quotes[quoteIdx][1]}</div>
    </div>
  </div>
  <div class="img" bind:this={divEl}>
    <img bind:this={imgEl} src={rana} alt="rana" on:mouseenter={onHover}
      on:click={createHeart} role="none" on:touchstart={onHover} />
  </div>
</main>

<style lang="sass">
  main
    position: relative
    width: 100%
    max-width: 100%
    height: 100%
    max-height: 100%
    overflow: hidden

    // background: #f0f0f0
    background: #f9f2e0
    display: flex
    justify-content: center
    align-items: center

  .img
    position: absolute
    display: none
    
    img 
      width: 200px

  .info
    // color: rgb(136 130 130 / 70%)
    color: rgb(206 182 160)
    display: flex
    flex-direction: column
    align-items: center
    
    h1
      font-size: 3rem
      color: transparent
      // background-color: rgb(136 130 130 / 70%)
      background-color: rgb(183 133 101 / 57%)
      -webkit-background-clip: text
      background-clip: text
      // text-shadow: rgba(255, 255, 255, .7) 0 3px 6px
      text-shadow: rgba(255, 255, 255, .5) 0 3px 6px
      margin-bottom: 0

      overflow: hidden
      white-space: nowrap

    .sub
      display: flex
      align-items: center
      font-size: 1.5rem
      width: 100%

      a
        display: flex
        align-items: center

    .quote
      font-size: 1.2rem
      position: relative
      margin-top: 1rem
      opacity: 0.7
      width: max-content

      &::before, &::after
        font-size: 2rem
        margin-right: 0.2rem
        position: absolute       

      &::before
        content: '「'
        top: -0.5rem
        left: -2.5rem

      &::after
        content: '」'
        bottom: -0.5rem
        right: -2.5rem
  
  @media screen and (max-width: 400px)
    .info
      h1
        font-size: 2rem

      div
        font-size: 1.2rem

    .img
      img
        width: 150px
</style>
