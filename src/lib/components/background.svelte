<script lang="ts">
  import debounce from 'lodash/debounce'
  import times from 'lodash/times'
  import random from 'random'
  import { rgba } from 'polished'
  import { onMount } from 'svelte'
  import { mode } from 'mode-watcher'

  /**
   * Draws an hexagone
   * @param ctx canvas context
   * @param x x coordinate
   * @param y y coordinate
   * @param r radius or edge length
   * @param fillStyle fill style
   * @param strokeStyle stroke style
   */
  const drawHexagone = (
    ctx: CanvasRenderingContext2D,
    x: number,
    y: number,
    r: number,
    fillStyle: CanvasFillStrokeStyles['fillStyle'],
    strokeStyle: CanvasFillStrokeStyles['strokeStyle'],
  ) => {
    ctx.beginPath()

    ctx.moveTo(x, y - r)

    const unit = Math.PI / 3
    // anti-clockwise
    times(6, (n: number) => {
      const angle = Math.PI / 2 + unit * (n + 1)
      ctx.lineTo(x + r * Math.cos(angle), y - r * Math.sin(angle))
    })
    ctx.closePath()
    ctx.fillStyle = fillStyle
    ctx.strokeStyle = strokeStyle
    ctx.lineWidth = 1
    ctx.fill()
    ctx.stroke()
  }

  const normal = random.normal(0.75, 0.25)

  /**
   * get a random x,y coordinate with provide width and height
   * @param w width
   * @param h height
   */
  const getRandomXY = (w: number, h: number) => {
    const x = Math.floor(w * Math.random())
    const y = Math.floor(h * normal())

    return { x, y }
  }

  let canvas: HTMLCanvasElement


  onMount(() => {
    const drawCanvas = () => {
      const foreground = `hsl(${getComputedStyle(document.documentElement).getPropertyValue(
        '--foreground',
      )})`

      if (!canvas) {
        return
      }
      const ctx = canvas.getContext('2d')!
      const pr = window.devicePixelRatio || 1
      const w = window.innerWidth
      const h = window.innerHeight

      canvas.width = w * pr
      canvas.height = h * pr
      canvas.style.width = `${w}px`
      canvas.style.height = `${h}px`
      ctx.scale(pr, pr)
      ctx.clearRect(0, 0, w, h)

      times(30, () => {
        const { x, y } = getRandomXY(w, h)

        drawHexagone(ctx, x, y, 50, `${rgba(foreground, 0.1)}`, 'transparent')
      })
      times(30, () => {
        const { x, y } = getRandomXY(w, h)

        drawHexagone(ctx, x, y, 50, 'transparent', `${rgba(foreground, 0.1)}`)
      })
    }

    const debouncedDrawCanvas = debounce(drawCanvas, 100)
    requestAnimationFrame(drawCanvas)
    window.addEventListener('resize', debouncedDrawCanvas)

    const unsubscribe =mode.subscribe(drawCanvas)

    return () => {
      window.removeEventListener('resize', debouncedDrawCanvas)
      unsubscribe()
    }
  })
</script>

<canvas bind:this={canvas} class="-z-1 t-0 l-0 fixed bg-background"></canvas>
