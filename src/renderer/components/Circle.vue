<template>
  <div class="rect-box">
    <div class="circle-box" :class="side"
         @mousedown="handle"
         @mouseover="handle"
         @mouseleave="handle"
         @dragover="handle"
         @dragleave="handle"
         @drop="handle">
      <div class="bg"
           :class="circleType"></div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'DCircle',
    methods: {
      handle (ev) {
        switch (ev.type) {
          case 'mouseover':
            // win.setIgnoreMouseEvents(ignore[, options])
            this.$electron.remote.getCurrentWindow().setIgnoreMouseEvents(false, {forward: true})
            break
          case 'mouseleave':
            this.$electron.remote.getCurrentWindow().setIgnoreMouseEvents(true, {forward: true})
            break
        }
        let handle = this.$listeners[ev.type]
        if (handle) handle(ev)
      }
    },
    props: {
      circleType: String,
      side: String
    }
  }
</script>

<style scoped lang="scss">
  .rect-box {
    background-color: transparent !important;
    overflow: hidden;
    position: relative;

    .circle-box {
      height: 100%;
      width: 100%;
      border-radius: 50%;
      border: 1px solid #ccc;
      box-sizing: border-box;
      background-color: #c4c4c4;
      vertical-align: top;
      position: absolute;

      &.left {
        left: -50% !important;
      }
      &.right {
        right: -50% !important;
      }

      &.top {
        top: -50% !important;
      }

      .bg {
        height: 100%;
        width: 100%;
        border-radius: 50%;
        background-position: center;
        background-repeat: no-repeat;
        background-size: 50%;

        &.right {
          margin-left: 50%;
          background-color: blue;
        }

        &.word {
          background-image: url("../assets/img/WORD.png");
        }

        &.excel {
          background-image: url("../assets/img/EXCEL.png");
        }

        &.ppt {
          background-image: url("../assets/img/PPT.png");
        }

        &.dragover {
          background-color: #f00;
        }

      }
    }
  }
</style>
