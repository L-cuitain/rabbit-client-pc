<template>
  <div class="xtx-dialog" v-if="isRendering" v-fade>
    <div class="wrapper" v-fade>
      <div class="header">
        <h3>{{ title }}</h3>
        <a
          @click="destroy"
          href="JavaScript:"
          class="iconfont icon-close-new"
        ></a>
      </div>
      <div class="body"><slot name="default" /></div>
      <div class="footer">
        <!--        <XtxButton type="gray" style="margin-right: 20px">取消</XtxButton>-->
        <!--        <XtxButton type="primary">确认</XtxButton>-->
        <slot name="footer" />
      </div>
    </div>
  </div>
</template>
<script>
import { useVModel } from "@vueuse/core";
export default {
  name: "XtxDialog",
  props: {
    title: {
      type: String,
      default: "",
    },
    visible: {
      type: Boolean,
      default: false,
    },
  },
  setup(props, { emit }) {
    const isRendering = useVModel(props, "visible", emit);
    const render = () => {
      isRendering.value = true;
    };
    const destroy = () => {
      isRendering.value = false;
    };
    return { isRendering, render, destroy };
  },
  directives: {
    fade: {
      mounted(el) {
        setTimeout(() => {
          el.classList.add("fade");
        }, 0);
      },
    },
  },
};
</script>
<style scoped lang="less">
.xtx-dialog {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  z-index: 8887;
  background: rgba(0, 0, 0, 0);
  &.fade {
    transition: all 0.4s;
    background: rgba(0, 0, 0, 0.5);
  }
  .wrapper {
    width: 600px;
    background: #fff;
    border-radius: 4px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -60%);
    opacity: 0;
    &.fade {
      transition: all 0.4s;
      transform: translate(-50%, -50%);
      opacity: 1;
    }
    .body {
      padding: 20px 40px;
      font-size: 16px;
      .icon-warning {
        color: @priceColor;
        margin-right: 3px;
        font-size: 16px;
      }
    }
    .footer {
      text-align: center;
      padding: 10px 0 30px 0;
    }
    .header {
      position: relative;
      height: 70px;
      line-height: 70px;
      padding: 0 20px;
      border-bottom: 1px solid #f5f5f5;
      h3 {
        font-weight: normal;
        font-size: 18px;
      }
      a {
        position: absolute;
        right: 25px;
        top: 25px;
        font-size: 24px;
        width: 20px;
        height: 20px;
        line-height: 20px;
        text-align: center;
        color: #999;
        &:hover {
          color: #666;
        }
      }
    }
  }
}
</style>
