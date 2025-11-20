<template>
  <div class="wrapper">
    <van-image :src="tron" class="logo">
      <template #error>加载失败</template>
    </van-image>

    <div class="title">接收TRC20网络代币</div>
    <div class="desc">同一个地址就能接收TRX和USDT</div>

    <div class="qrcode-wrapper">
      <canvas class="qrcode" ref="canvas" />
    </div>

    <div class="address_wrapper" v-if="wallet" @click="copyAddress">
      <div class="address">{{ wallet.address }}</div>
      <div class="copy">
        <van-icon name="notes-o" size="1.5rem" color="#c0c0c0" />
      </div>
    </div>

    <van-button class="send" block type="primary" @click="copyAddress">
      复制地址
    </van-button>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import copy from 'copy-to-clipboard'
import { showToast } from 'vant'
import QRCode from 'qrcode'
import tron from '@/assets/icons/tron.svg'

const canvas = ref(null)
const wallet = ref(null)

onMounted(() => {
  wallet.value = localStorage.getItem('wallet')
    ? JSON.parse(localStorage.getItem('wallet'))
    : null

  if (!wallet.value) return

  QRCode.toCanvas(
    canvas.value,
    wallet.value.address,
    {
      margin: 0,
      width: 200,
      color: {
        dark: '#000',
        light: '#eff2f5'
      }
    },
    function (error) {
      if (error) console.error(error)
    }
  )
})

const copyAddress = () => {
  try {
    copy(wallet.value.address)
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
.wrapper {
  display: flex;
  flex-direction: column;
  padding: 30px 10px 10px 10px;
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