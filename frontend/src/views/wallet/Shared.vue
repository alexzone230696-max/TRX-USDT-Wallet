<template>
  <div class="outter">
    <van-notice-bar color="#fff" background="#1989fa" left-icon="passed">
      妙蛙钱包独立运行，无需安装即可使用
    </van-notice-bar>

    <div class="wrapper">
      <van-image :src="tron" class="logo">
        <template #error>加载失败</template>
      </van-image>

      <div class="title">TRC20网络地址</div>
      <div class="desc">由妙蛙钱包提供技术支持</div>

      <div class="qrcode-wrapper">
        <canvas class="qrcode" ref="canvas" />
      </div>

      <div class="address_wrapper" v-if="sharedAddress" @click="copyAddress">
        <div class="address">{{ sharedAddress }}</div>
        <div class="copy">
          <van-icon name="notes-o" size="1.5rem" color="#c0c0c0" />
        </div>
      </div>

      <van-button class="send" block type="primary" @click="copyAddress">
        复制地址
      </van-button>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import copy from 'copy-to-clipboard'
import { showToast } from 'vant'
import QRCode from 'qrcode'
import tron from '@/assets/icons/tron.svg'

const canvas = ref(null)
const sharedAddress = ref(null)

onMounted(() => {
  sharedAddress.value = localStorage.getItem('shared') || null

  QRCode.toCanvas(
    canvas.value,
    sharedAddress.value,
    {
      margin: 0,
      width: 130,
      color: {
        dark: '#000',
        light: '#eff2f5'
      }
    },
    (err) => {
      if (err) {
        console.error(err)
        showToast(err.message)
      }
    }
  )
})

const copyAddress = () => {
  try {
    copy(sharedAddress.value)
    showToast({
      message: '地址已复制到剪贴板',
      position: 'bottom'
    })
  } catch (e) {
    showToast({
      message: '复制失败',
      position: 'bottom'
    })
  }
}
</script>

<style lang="scss" scoped>
.outter {
  height: 100vh;
  width: 100vw;
  overflow-y: scroll;
  overflow-x: hidden;
}
.wrapper {
  display: flex;
  flex-direction: column;
  padding: 10px;
  justify-content: center;
  align-items: center;

  .logo {
    width: 60px;
    padding: 20px;
  }

  .title {
    font-size: 28px;
    font-weight: bolder;
  }

  .desc {
    color: #bbbbbb;
    font-size: 16px;
    padding: 5px;
  }

  .qrcode-wrapper {
    padding: 20px;
  }

  .address_wrapper {
    width: 80%;
    padding: 10px;
    border-radius: 5px;
    border: 1px solid #c0c0c0;
    display: flex;
    align-items: center;
    justify-content: center;

    .address {
      flex-grow: 1;
      width: 80%;
      color: #bbbbbb;
      font-size: 14px;
      overflow-wrap: break-word;
    }

    .copy {
      padding-left: 10px;
    }
  }

  .send {
    width: 86%;
    margin-top: 15px;
  }
}
</style>