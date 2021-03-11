<script>
  import { afterUpdate, tick, onMount, onDestroy } from "svelte";
  import { fly, fade } from "svelte/transition";
  export let visible = true;
  export let maxVH = 90;
  export let minVH = 85;
  let stopExpand,
    initialized = false;
  let undraggeble = true;
  let inner, lineWrapper, wrapper, line, actionsDiv, overflow;
  let origHeight,
    origBottom,
    maxWidth,
    lastPageY,
    value,
    per_viewportHeight = 0;
  let touchStartHandler, touchMoveHandler, touchPoint;
  let vh;

  window.addEventListener("resize", () => {
    if (visible) {
      maxWidth = getComputedStyle(inner).width;
      lineWrapper.style.width = maxWidth;
    }
  });

  afterUpdate(() => {
    if (visible && !initialized) {
      origHeight = parseInt(getComputedStyle(inner).maxHeight.split("px")[0]);
      maxWidth = getComputedStyle(inner).width;
      lineWrapper.style.width = maxWidth;
      initialize();
      initialized = true;
      blockDrag();
    } else if (!visible) {
      lastPageY = undefined;
      initialized = false;
      stopExpand = false;
    }

    if (visible) {
      // updated the linewrapper position
      let innerOffset = getOffset(inner);
      lineWrapper.style.top = innerOffset.yPosition + "px";
      lineWrapper.style.left = innerOffset.xPosition + "px";

      actionsDiv.style.width = lineWrapper.style.width;
    }
  });
  onMount(() => {
    visible = false;
    if(typeof wrapper === 'Element')
      vh = getComputedStyle(wrapper).height.split("px")[0] / 100;
    if(typeof inner === 'Element')
    if (getComputedStyle(inner).height.split("px")[0] > vh * 85) {
      overflow = "scroll";
    } else {
      overflow = "inherit";
    }

    initialize();
    undraggeble =
      document.ontouchmove === undefined &&
      typeof window.orientation !== "undefined";
      if(typeof inner === 'Element')
      origBottom = getComputedStyle(inner).bottom;
      setTimeout(() => {
      if(typeof inner === 'Element')
        maxWidth = getComputedStyle(inner).width;
        if(typeof linewrapper !== 'undefined')
      lineWrapper.style.width = maxWidth;
    }, 500);
  });

  function initialize() {
    if(typeof inner === 'Element')
      per_viewportHeight = parseInt(getComputedStyle(inner).minHeight.split("px")[0]) / minVH;
    if (document.ontouchmove === null) {
      // touch move supported
      // passive event listener: checkout https://www.chromestatus.com/feature/5745543795965952
      var supportsPassive = false;
      try {
        var opts = Object.defineProperty({}, "passive", {
          get: function() {
            supportsPassive = true;
          }
        });
        window.addEventListener("testPassive", null, opts);
        window.removeEventListener("testPassive", null, opts);
      } catch (e) {}
      // add event listener
      lineWrapper.addEventListener(
        "touchmove",
        moveElement,
        supportsPassive ? { passive: true } : false
      );
      lineWrapper.addEventListener(
        "touchend",
        closeElement,
        supportsPassive ? { passive: true } : false
      );
      return;
    } else if (undraggeble) {
      // touch move not supported and is mobile version
      if(typeof line !== 'undefined'){
      line.addEventListener("click", () => {
        closeElement(1);
      });
    }
  }
    // desktop version
    lineWrapper.addEventListener("dragstart", e => {
      e = e || window.event;
      e.preventDefault();
      document.addEventListener("mousemove", moveElement);
      document.addEventListener("mouseup", closeElement);
    });
  }
  function moveElement(e) {
    if (!inner) {
      return;
    }
    let touchLocation =
      document.ontouchmove === null ? e.targetTouches[0].pageY : e.clientY;
    let minBottomExpo = 1.5;
    if (lastPageY === undefined) {
      lastPageY = touchLocation;
    }
    if (lastPageY <= touchLocation) {
      // user is dragging down the lineWrapper
      //-- apparently the "if" is not necessary here
      let botton = getComputedStyle(inner).bottom;
      value = parseInt(botton.split("px")[0]);
      inner.style.bottom = value - Math.abs(touchLocation - lastPageY) + "px";
      lastPageY = touchLocation;
      if (Math.abs(value * minBottomExpo) > origHeight) {
        closeElement(1);
      }
    } else {
      // user is dragging up the lineWrapper
      innerHeight = parseInt(inner.style.maxHeight.split("px")[0]);
      innerHeight = isNaN(innerHeight) ? 0 : innerHeight;
      if (innerHeight <= maxVH * per_viewportHeight) {
        let maxHeight = getComputedStyle(inner).maxHeight;
        value = parseInt(maxHeight.split("px")[0]);
        inner.style.maxHeight =
          value + Math.abs(touchLocation - lastPageY) + "px";
        lastPageY = touchLocation;
      } else {
        stopExpand = true;
      }
    }
  }
  function closeElement(e) {
    unblockDrag();
    value = 0;
    lastPageY = undefined;
    document.removeEventListener("mousemove", moveElement);
    if (Math.abs(value * 5) > origHeight || e === 1) {
      visible = false;
    } else if (inner && visible) {
      inner.style.bottom = origBottom;
    }
  }
  function blockDrag() {
    (function() {
      // Only needed for touch events on chrome.
      if (
        (window.chrome || navigator.userAgent.match("CriOS")) &&
        "ontouchstart" in document.documentElement
      ) {
        touchStartHandler = function() {
          // Only need to handle single-touch cases
          touchPoint =
            event.touches.length === 1 ? event.touches[0].clientY : null;
        };

        touchMoveHandler = function(event) {
          var newTouchPoint;

          // Only need to handle single-touch cases
          if (event.touches.length !== 1) {
            touchPoint = null;

            return;
          }

          // We only need to defaultPrevent when scrolling up
          newTouchPoint = event.touches[0].clientY;
          if (newTouchPoint > touchPoint) {
            event.preventDefault();
          }
          touchPoint = newTouchPoint;
        };

        document.addEventListener("touchstart", touchStartHandler, {
          passive: false
        });

        document.addEventListener("touchmove", touchMoveHandler, {
          passive: false
        });
      }
    })();
  }
  function unblockDrag() {
    if (
      (window.chrome || navigator.userAgent.match("CriOS")) &&
      "ontouchstart" in document.documentElement
    ) {
      document.removeEventListener("touchstart", touchStartHandler);

      document.removeEventListener("touchmove", touchMoveHandler);
    }
  }
  function getOffset(element) {
    let xPosition = 0;
    let yPosition = 0;

    while (element) {
      yPosition += element.offsetTop - element.scrollTop + element.clientTop;
      xPosition += element.offsetLeft - element.scrollLeft + element.clientLeft;
      element = element.offsetParent;
    }

    return { xPosition: xPosition, yPosition: yPosition };
  }
</script>

<style>
  .wrapper {
    z-index: 2;
    position: fixed;
    top: 0;
    left: 0;
    background: rgba(255, 255, 255, 0.75);
    min-height: 100vh;
    max-height: 100vh;
    min-width: 100vw;
    max-width: 100vw;
  }
  .inner {
    z-index: 3;
    position: fixed;
    left: 50%;
    transform: translateX(-50%);
    bottom: -15px;
    background: #ffffff;
    box-shadow: 0px 0px 25px rgba(0, 0, 0, 0.25);
    border-radius: 19px;
    max-width: 100vw;
    min-width: 100vw;
    padding: 35px;
    min-height: 65vh;
    max-height: 65vh;
    margin: auto;
    overflow: scroll;
    -ms-overflow-style: none;
  }
  .inner::-webkit-scrollbar {
    display: none;
  }
  .line {
    width: 45px;
    height: 7.14px;
    background: #ececec;
    border-radius: 10px;
    margin: auto;
    width: 100px;
  }
  .crossimg {
    width: 25px;
  }
  .content {
    padding-top: 35px;
    margin: auto;
  }
  .innercontent {
    position: relative;
    min-height: 100%;
  }
  .linewrapper {
    position: fixed;
    z-index: 4;
    left: 0;
    height: 70px;
    display: flex;
    justify-content: center;
    align-items: center;
    transform: translateX(-50%);
    background: white;
    border-radius: 19px;
    cursor: move;
  }
  .actionsDiv {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0px 35px;
  }

  @media only screen and (max-width: 570px) {
    .wrapper .inner {
      min-width: 100vw !important;
    }
  }
  @media only screen and (max-width: 800px) {
    .inner {
      min-width: 70vw !important;
    }
  }
  @media only screen and (max-width: 1000px) {
    .inner {
      min-width: 70vw !important;
    }
  }
  @media only screen and (max-width: 1600px) {
    .inner {
      min-width: 50vw;
    }
  }
  @media only screen and (min-width: 1600px) {
    .inner {
      min-width: 40vw;
    }
  }
</style>

{#if visible}
  <div
    class="wrapper"
    bind:this={wrapper}
    in:fade={{ duration: 250 }}
    out:fade={{ duration: 350 }}>
    <div
      style="overflow-y:{overflow};min-height:{minVH}vh;max-height:{maxVH}vh"
      class="inner"
      in:fly={{ y: 2000, duration: 250 }}
      out:fly={{ y: 1000, duration: 750 }}
      bind:this={inner}>
      <div class="innercontent">
        <div class="content">
          <slot />
        </div>
      </div>
    </div>

    <div
      in:fly={{ y: 2000, duration: 250 }}
      out:fly={{ y: 1000, duration: 750 }}
      class="linewrapper"
      draggable="true"
      bind:this={lineWrapper}>
      <div>
        {#if undraggeble}
          <img
            src="/assets/cross_primary.svg"
            alt="X"
            class="crossimg"
            bind:this={line}
            style="cursor: {undraggeble ? 'pointer' : 'move'};" />
        {:else}
          <div class="line" bind:this={line} />
        {/if}

        <div class="actionsDiv" bind:this={actionsDiv}>
          <slot name="left" />
          <slot name="right" />
        </div>
      </div>
    </div>
  </div>
{/if}
