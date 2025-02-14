<script lang="ts">
  import { onMount } from "svelte";
  import rana from "./rana.png"

  const duration = 0.3

  // Pos: % from 0 to 1 on the axis perpendicular to the direction
  let pos = 0.5

  // top | bottom | left | right
  const directions = ["top", "bottom", "left", "right"]
  let direction = "bottom"
  
  // 0: idle, 1: entering, 2: leaving
  let state = 0

  // Reference to the image element
  let divEl: HTMLDivElement
  let imgEl: HTMLImageElement

  function getInitTrans() {
    const dirC = direction === 'top' || direction === 'bottom' ? 'Y' : 'X'
    const dirN = direction === 'top' || direction === 'left' ? '-' : ''
    const dirV = direction === 'top' || direction === 'bottom' ? '300px' : '200px'
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

    // Randomize the direction
    state = 1
    direction = directions[Math.floor(Math.random() * directions.length)]
    console.log("New direction:", direction)
    await divStyles()
  }

  async function divStyles() {
    if (divEl === undefined) return

    let obj: any = {}
 
    obj.transform = getInitTrans()
    obj[direction] = '-50px'
    for (let dir of directions) {
      if (dir !== direction) obj[dir] = 'unset'
    }
    let perpendicular = direction === 'bottom' || direction === 'top' ? 'left' : 'top'
    obj[perpendicular] = `calc(${pos * 100}% - 100px)`
    
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
    
    obj['rotate'] = direction === 'top' ? '180deg' 
      : direction === 'left' ? '30deg'
      : direction === 'right' ? '-30deg'
      : '0deg'

    if (direction === 'right') {
      obj['transform'] = 'scaleX(-1)'
    }

    imgEl.style.cssText = Object.keys(obj).map(key => `${key}: ${obj[key]}`).join('; ')
  }

  onMount(() => {
    setTimeout(() => divStyles(), 100)
  })
</script>

<main>
  <div class="img" bind:this={divEl}>
    <img bind:this={imgEl} src={rana} alt="rana" on:mouseenter={onHover}/>
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

  .img
    position: absolute
    
    img 
      width: 200px
</style>
