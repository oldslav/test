<template>
    <div class="whiteboard">
        <canvas class="canv" :class="{'whiteboard__active': master}" :id="canvasId" width="400" height="400"
                @drop="onDrop"
                @dragover.prevent></canvas>
    </div>
</template>

<script>
  const isPointInRange = (x, y, obj) => {
    return !(x < obj.x ||
        x > obj.x + obj.width ||
        y < obj.y ||
        y > obj.y + obj.height);
  };

  const randomizeId = () => {
    return `_${Math.random().toString(36).substr(2, 9)}`;
  };

  export default {
    props: {
      outerContext: {
        type: Object,
        default: () => ({})
      },
      imageId: {
        type: String,
        default: null
      }
    },
    watch: {
      outerContext: {
        deep: true,
        handler(val) {
          // console.log('val', this.outerContext);
          if (this.canvasId !== val.id) {
            this.master = false;
          }
        }
      }
    },
    data() {
      return {
        master: false,
        canvasId: randomizeId(),
        context: null,
        canvas: null,
        object: null
      };
    },

    mounted() {
      this.canvas = document.getElementById(this.canvasId);
      this.context = this.canvas.getContext('2d');
      this.renderContent();
    },

    computed: {
      imageContext() {
        return this.master ? this.object : this.outerContext;
      },
      image() {
        return document.getElementById(this.imageId);
      }
    },

    methods: {
      throwOutside() {
        const ctx = Object.assign({}, this.object);
        this.$emit('ctx', ctx);
      },
      renderContent() {
        const draw = () => {
          if (!this.master && this.imageContext.image) {
            setTimeout(requestAnimationFrame, 1000, draw);
          } else {
            requestAnimationFrame(draw);
          }

          this.context.clearRect(0, 0, this.canvas.width, this.canvas.height);

          this.imageContext.image && this.drawImage();

          this.canvas.onmousedown = (e) => {
            if (this.master) {
              const downX = e.offsetX;
              const downY = e.offsetY;

              if (isPointInRange(downX, downY, this.object)) {
                startMove(downX, downY);
              }
            }
          };

          const startMove = (downX, downY) => {
            const origX = this.object.x;
            const origY = this.object.y;

            this.canvas.onmousemove = (e) => {
              const moveX = e.offsetX;
              const moveY = e.offsetY;
              const diffX = moveX - downX;
              const diffY = moveY - downY;

              this.object.x = origX + diffX;
              this.object.y = origY + diffY;

            };

            this.canvas.onmouseup = () => {
              this.canvas.onmousemove = function () {
              };
            };
          };
        };
        draw();
      },

      drawImage() {
        // let a;
        // if (this.master) {
        //   a = Object.assign({}, this.object);
        // } else {
        //   a = Object.assign({}, this.outerContext);
        // }
        this.context.drawImage(this.image, this.imageContext.x, this.imageContext.y, this.imageContext.width, this.imageContext.height);
        this.throwOutside();
      },

      setAsMaster() {
        this.master = true;
        this.$emit('master', this.canvasId);
        console.log('setAsMaster', this.canvasId);
      },

      onDrop(e) {
        // e.stopPropagation();
        e.preventDefault();

        // const image = document.getElementById(e.dataTransfer.getData('image_id'));
        const image = document.getElementById(this.imageId);

        var mouse_position_x = e.dataTransfer.getData('mouse_position_x');
        var mouse_position_y = e.dataTransfer.getData('mouse_position_y');

        const img_posX = e.clientX - this.canvas.offsetLeft - mouse_position_x;
        const img_posY = e.clientY - this.canvas.offsetTop - mouse_position_y;

        this.object = {
          id: this.canvasId,
          context: this.context,
          image: image,
          x: img_posX,
          y: img_posY,
          width: image.offsetWidth,
          height: image.offsetHeight
        };

        this.drawImage();
        this.setAsMaster();
      },
    }
  };
</script>

<style lang="stylus">
    .whiteboard {
        .canv {
            border solid 1px
            margin 2px
        }

        .whiteboard__active {
            border-color red
        }
    }
</style>
