<template>
    <div id="app">
        <div>
            <img draggable="true" @dragstart="onDrag" src="@/assets/animals.svg" alt="" id="app_image">
        </div>
        <div class="app__container">
            <whiteboard ref="first" :image-id="imageId" :outer-context="getMasterContext" @master="setMaster"
                        @ctx="setMasterContext"/>
            <whiteboard ref="second" :image-id="imageId" :outer-context="getMasterContext" @master="setMaster"
                        @ctx="setMasterContext"/>
            <whiteboard ref="third" :image-id="imageId" :outer-context="getMasterContext" @master="setMaster"
                        @ctx="setMasterContext"/>
            <whiteboard ref="fourth" :image-id="imageId" :outer-context="getMasterContext" @master="setMaster"
                        @ctx="setMasterContext"/>
        </div>
    </div>
</template>

<script>
  import Whiteboard from './components/Whiteboard';

  export default {
    name: 'App',
    components: {
      Whiteboard
    },
    data() {
      return {
        masterId: null,
        masterContext: null,
        imageId: null
      };
    },
    computed: {
      getMasterContext() {
        return Object.assign({}, this.masterContext);
      }
    },
    methods: {
      setMasterContext(ctx) {
        if (ctx && ctx.id === this.masterId) {
          this.masterContext = ctx;
        }
      },
      setMaster(id) {
        this.masterId = id;
      },
      onDrag(e) {
        e.dataTransfer.setData('mouse_position_x', e.clientX - e.target.offsetLeft);
        e.dataTransfer.setData('mouse_position_y', e.clientY - e.target.offsetTop);

        this.imageId = e.target.id;

        // e.dataTransfer.setData('image_id', e.target.id);

      },
    }
  };
</script>

<style lang="stylus">
    body, html {
        height 100vh
        padding 0
        margin 0
    }

    #app {
        max-height 100vh

        display flex
        flex-direction column
        align-items center
        justify-content center

        .app__container {
            display grid
            grid-template-columns 1fr 1fr
            grid-template-rows 1fr 1fr
            flex 1
        }

        #app_image {
            width 50px
            height 50px
        }
    }
</style>
