<template>
  <div id="app">
    <d-circle
      @mousedown="mousedown"
      @dragover="dragover()"
      class="circle curr-circle"
      :side="side"
      ref="circle"></d-circle>

    <div class="circle-drag-show" :class="showPosition">
      <d-circle class="circle" @dragleave="dragleave" @dragover="dragover('word')" @drop="drop('word')"
                circle-type="word"></d-circle>
      <d-circle class="circle" @dragleave="dragleave" @dragover="dragover('excel')" @drop="drop('excel')"
                circle-type="excel"></d-circle>
      <d-circle class="circle" @dragleave="dragleave" @dragover="dragover('ppt')" @drop="drop('ppt')"
                circle-type="ppt"></d-circle>
    </div>
  </div>
</template>

<script>
  import DCircle from './components/Circle'

  export default {
    name: 'desktop-levitation-ball',
    components: {DCircle},
    data () {
      return {
        side: '',
        showPosition: ''
      }
    },
    computed: {
      circle () {
        return this.$refs.circle
      },
      app () {
        return this.$root.$refs.app.$el
      }
    },
    mounted () {
      let app = this.app
      this.$electron.remote.getCurrentWindow().setSize(app.offsetWidth, app.offsetHeight)
      window.addEventListener('dragover', () => {
        event.preventDefault()
      })
      window.addEventListener('dragleave', () => {
        let ev = event
        if (!ev.screenX && !ev.screenY) this.showPosition = ''
      })
      window.addEventListener('drop', () => {
        this.showPosition = ''
      })
      let winPos = JSON.parse(localStorage.getItem('winPos'))
      if (winPos) {
        let circle = this.circle.$el

        let sWidth = window.screen.availWidth
        let sHeight = window.screen.availHeight

        let circleX = circle.offsetLeft
        let circleY = circle.offsetTop

        let width = circle.offsetWidth
        let height = circle.offsetHeight

        let inSide = false
        if (winPos.x < -circleX) {
          this.side = 'left'
          inSide = true
          winPos.x = -circleX
        } else if (winPos.x + circleX + width > sWidth - 5) {
          this.side = 'right'
          inSide = true
          winPos.x = sWidth - width - circleX
        } else {
          this.side = ''
        }

        if (winPos.y < -circleY + 5) {
          this.side = 'top'
          winPos.y = -circleY
        } else if (winPos.y + circleY + height > sHeight) {
          winPos.y = sHeight - height - circleY
        } else {
          if (!inSide) this.side = ''
        }
        this.$electron.ipcRenderer.send('setPosition', winPos)
      }
    },
    methods: {
      drop (type) {
        let target = event.target
        if (target.classList.contains('circle-box')) return
        target.classList.remove('dragover')
        let files = event.dataTransfer.files
        let filePaths = []
        for (let i = 0; i < files.length; i++) {
          let file = files[i]
          filePaths.push(file.path)
        }
        this.$electron.remote.dialog.showMessageBox(this.$electron.remote.getCurrentWindow(), {
          title: '系统提示',
          message: `开始将文件：\n${filePaths.join('\n')}\n转换为【${type}】`
        })
      },
      dragleave () {
        let target = event.target
        if (target.classList.contains('circle-box')) return
        target.classList.remove('dragover')
      },
      dragover (type) {
        event.preventDefault()
        if (type) {
          event.dataTransfer.dropEffect = 'copy'
          let target = event.target
          if (target.classList.contains('circle-box')) return
          target.classList.add('dragover')
          return
        }
        let position = ''
        let x = window.screenX
        let y = window.screenY
        let width = window.outerWidth
        let height = window.outerHeight
        let sWidth = window.screen.availWidth
        let sHeight = window.screen.availHeight
        if (x + width < sWidth) {
          position = 'right'
        } else {
          position = 'left'
        }
        if (y + height < sHeight) {
          position += '-bottom'
        } else {
          position += '-top'
        }
        this.showPosition = position
      },
      mousedown () {
        let ev = event
        let [x, y] = this.$electron.remote.getCurrentWindow().getPosition()
        let startX = ev.screenX
        let startY = ev.screenY

        let sWidth = window.screen.availWidth
        let sHeight = window.screen.availHeight

        let circle = this.circle.$el

        let circleX = circle.offsetLeft
        let circleY = circle.offsetTop

        let width = circle.offsetWidth
        let height = circle.offsetHeight

        const move = ev => {
          let inSide = false
          let deltaX = ev.screenX - startX
          let deltaY = ev.screenY - startY

          let newX = x + deltaX
          let newY = y + deltaY

          if (newX < -circleX) {
            this.side = 'left'
            inSide = true
            newX = -circleX
          } else if (newX + circleX + width > sWidth - 5) {
            this.side = 'right'
            inSide = true
            newX = sWidth - width - circleX
          } else {
            this.side = ''
          }

          if (newY < -circleY + 5) {
            this.side = 'top'
            newY = -circleY
          } else if (newY + circleY + height > sHeight) {
            newY = sHeight - height - circleY
          } else {
            if (!inSide) this.side = ''
          }

          localStorage.setItem('winPos', JSON.stringify({x: newX, y: newY}))
          this.$electron.ipcRenderer.send('setPosition', {x: newX, y: newY})
        }
        const up = () => {
          window.removeEventListener('mousemove', move)
          window.removeEventListener('mouseup', up)
        }
        window.addEventListener('mousemove', move)
        window.addEventListener('mouseup', up)
      }
    }
  }
</script>

<style lang="scss">
  body, html {
    margin: 0;
    padding: 0;
    user-select: none;
    cursor: default;
  }

  /* CSS */
</style>

<style scoped lang="scss">
  $circle_size: 80px;
  $circle_space_size: 10px;
  #app {
    background-color: transparent;
    height: calc(#{$circle_size * 5} + #{$circle_space_size * 4});
    width: calc(#{$circle_size * 3} + #{$circle_space_size * 2});
    position: relative;

    .circle {
      height: $circle_size;
      width: $circle_size;
    }

    .curr-circle {
      position: absolute;
      top: calc(#{$circle_size + $circle_space_size} * 2);
      left: calc(#{$circle_size + $circle_space_size});

      /* &.right {
         left: calc(#{$circle_size*3/2 + $circle_space_size});
       }

       &.top {
         top: calc(#{$circle_size*3/4 + $circle_space_size} * 2);
       }*/
    }

    .circle-drag-show {
      position: absolute;
      height: calc(#{$circle_size * 3} + #{$circle_space_size * 2});
      width: $circle_size;
      visibility: hidden;

      &.left-top {
        visibility: visible;
        top: 0;
        left: 0;

        .circle {
          margin-bottom: $circle_space_size;

          &:nth-last-child(1) {
            margin-bottom: 0;
          }
        }
      }

      &.left-bottom {
        visibility: visible;
        left: 0;
        bottom: 0;

        .circle {
          margin-top: $circle_space_size;

          &:nth-child(1) {
            margin-top: 0;
          }
        }
      }

      &.right-top {
        visibility: visible;
        top: 0;
        right: 0;

        .circle {
          margin-bottom: $circle_space_size;

          &:nth-last-child(1) {
            margin-bottom: 0;
          }
        }
      }

      &.right-bottom {
        visibility: visible;
        right: 0;
        bottom: 0;

        .circle {
          margin-top: $circle_space_size;

          &:nth-child(1) {
            margin-top: 0;
          }
        }
      }
    }
  }
</style>
