<svelte:options tag="double-slider" />

<script>
  import { clamp } from 'yootils'
  let start = 0
  let end = 1
  let leftHandle
  let body
  let slider
  function draggable(node) {
    let x
    let y
    function handleMousedown(event) {
      if (event.type === 'touchstart') {
        event = event.touches[0]
      }
      x = event.clientX
      y = event.clientY
      node.dispatchEvent(
        new CustomEvent('dragstart', {
          detail: { x, y }
        })
      )
      window.addEventListener('mousemove', handleMousemove)
      window.addEventListener('mouseup', handleMouseup)
      window.addEventListener('touchmove', handleMousemove)
      window.addEventListener('touchend', handleMouseup)
    }
    function handleMousemove(event) {
      if (event.type === 'touchmove') {
        event = event.changedTouches[0]
      }
      const dx = event.clientX - x
      const dy = event.clientY - y
      x = event.clientX
      y = event.clientY
      node.dispatchEvent(
        new CustomEvent('dragmove', {
          detail: { x, y, dx, dy }
        })
      )
    }
    function handleMouseup(event) {
      x = event.clientX
      y = event.clientY
      node.dispatchEvent(
        new CustomEvent('dragend', {
          detail: { x, y }
        })
      )
      window.removeEventListener('mousemove', handleMousemove)
      window.removeEventListener('mouseup', handleMouseup)
      window.removeEventListener('touchmove', handleMousemove)
      window.removeEventListener('touchend', handleMouseup)
    }
    node.addEventListener('mousedown', handleMousedown)
    node.addEventListener('touchstart', handleMousedown)
    return {
      destroy() {
        node.removeEventListener('mousedown', handleMousedown)
        node.removeEventListener('touchstart', handleMousedown)
      }
    }
  }
  function setHandlePosition(which) {
    return function (evt) {
      const { left, right } = slider.getBoundingClientRect()
      const parentWidth = right - left
      const p = Math.min(Math.max((evt.detail.x - left) / parentWidth, 0), 1)
      if (which === 'start') {
        start = p
        end = Math.max(end, p)
      } else {
        start = Math.min(p, start)
        end = p
      }
    }
  }
  function setHandlesFromBody(evt) {
    const { width } = body.getBoundingClientRect()
    const { left, right } = slider.getBoundingClientRect()
    const parentWidth = right - left
    const leftHandleLeft = leftHandle.getBoundingClientRect().left
    const pxStart = clamp(
      leftHandleLeft + event.detail.dx - left,
      0,
      parentWidth - width
    )
    const pxEnd = clamp(pxStart + width, width, parentWidth)
    const pStart = pxStart / parentWidth
    const pEnd = pxEnd / parentWidth
    start = pStart
    end = pEnd
  }

  const nice = d => {
    if (!d && d !== 0) return ''
    return (d * 1000).toFixed(0)
  }
</script>

<main>
  <div class="double-range-container">
    <div class="slider" bind:this={slider}>
      <div
        class="body"
        bind:this={body}
        use:draggable
        on:dragmove|preventDefault|stopPropagation={setHandlesFromBody}
        style="
          left: {100 * start}%;
          right: {100 * (1 - end)}%;
        "
      />
      <div
        class="handle"
        bind:this={leftHandle}
        data-which="start"
        use:draggable
        on:dragmove|preventDefault|stopPropagation={setHandlePosition('start')}
        style="
          left: {100 * start}%
        "
      />
      <div
        class="handle"
        data-which="end"
        use:draggable
        on:dragmove|preventDefault|stopPropagation={setHandlePosition('end')}
        style="
          left: {100 * end}%
        "
      />
    </div>
  </div>

  <div class="labels">
    <div class="label">R${nice(start)},00</div>
    <div class="label">R${nice(end)},00</div>
  </div>
  <div class="container-filtro">
    <a
      class="filtrar"
      href="{window.location.pathname}/de-{nice(start)}-a-{nice(
        end
      )}?PS=9{'&'}map=c,priceFrom">Filtrar</a
    >
  </div>
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,400;0,500;0,600;0,700;1,300&display=swap');

  .double-range-container {
    width: 100%;
    height: 20px;
    user-select: none;
    box-sizing: border-box;
    white-space: nowrap;
  }
  .slider {
    position: relative;
    width: 100%;
    height: 1px;
    top: 50%;
    transform: translate(0, -50%);
    background-color: #bebebe;
  }
  .handle {
    position: absolute;
    top: 50%;
    width: 0;
    height: 0;
  }
  .handle:after {
    content: ' ';
    box-sizing: border-box;
    position: absolute;
    border-radius: 50%;
    width: 12px;
    height: 12px;
    background-color: red;
    transform: translate(-50%, -50%);
  }
  /* .handle[data-which="end"]:after{
		transform: translate(-100%, -50%);
	} */
  .body {
    top: 0;
    height: 3px;
    position: absolute;
    background-color: red;
    bottom: 0;
  }

  main {
    text-align: center;
    /* padding: 1em; */
    /* max-width: 300px; */
    margin: 0 auto;
  }
  .filtrar {
    background: red;
    border-radius: 30px;
    padding: 7px 20px;
    color: white;
    text-decoration: none;
    font-size: 15px;
  }
  .label {
    font-style: italic;
    font-size: 15px;
    color: #979797;
    font-family: 'Montserrat';
  }
  .label:first-child {
    float: left;
  }
  .label:last-child {
    float: right;
  }
  .container-filtro {
    display: flex;
    justify-content: flex-end;
    margin-top: 30px;
  }
</style>
