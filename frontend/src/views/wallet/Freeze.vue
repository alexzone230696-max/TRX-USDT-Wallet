<template>
  <van-notice-bar left-icon="info-o" color="#1989fa" background="#ecf9ff">
    质押可赚带宽/能量抵交易费, 解押需3天到账
  </van-notice-bar>

  <div class="wrapper">
    <van-form @submit="onSubmit">
      <van-cell-group inset>
        <van-field
          v-model="stakeTypeText"
          is-link
          readonly
          label="操作"
          placeholder="请选择操作"
          @click="showStakePicker = true"
          :rules="[{ required: true, message: '请选择操作' }]"
          :disabled="loading"
        >
          <template #button>
            <span v-if="stakeType.length && purposeType.length && !loading">
              余额: {{ balance }}
            </span>
          </template>
        </van-field>

        <van-field
          v-model="purposeTypeText"
          is-link
          readonly
          label="类型"
          placeholder="请选择类型"
          @click="showPurposePicker = true"
          :rules="[{ required: true, message: '请选择操作' }]"
          :disabled="loading"
        />

        <van-field
          v-model="amount"
          type="number"
          label="数量"
          placeholder="请输入代币数量"
          :rules="[{ required: true, message: '请输入代币数量' }]"
          :disabled="loading"
        >
          <template #button>
            <span v-if="balance" @click="amount = balanceAmount / 10 ** 6">全部</span>
          </template>
        </van-field>
      </van-cell-group>

      <div style="margin: 16px" v-if="estimated">
        <van-notice-bar color="#1989fa" background="#ecf9ff">
          预计消耗 {{ fee }} 个TRX,
          {{ stakeType[0].value === 'freeze' ? '获得' : '失去' }}
          {{ benifit }} 个 {{ purposeType[0].value === 'ENERGY' ? '能量' : '带宽' }}
          (可抵扣 {{ deduct }} 个TRX 的
          {{ purposeType[0].value === 'ENERGY' ? '智能合约' : '普通' }} 交易费用)
        </van-notice-bar>
      </div>

      <div style="margin: 16px">
        <van-button
          block
          type="primary"
          native-type="submit"
          :loading="loading"
          :disabled="loading || balance == 0 || amount == 0 || parseFloat(amount) > balanceAmount / 10 ** 6"
        >
          {{ estimated ? '提交请求' : '估算费用' }}
        </van-button>
      </div>
    </van-form>
  </div>

  <!-- 质押操作选择 -->
  <van-popup :show="showStakePicker && !loading" destroy-on-close position="bottom">
    <van-picker
      :columns="stakeColumns"
      @confirm="onStakeConfirm"
      @cancel="showStakePicker = false"
      destroy-on-close
    />
  </van-popup>

  <!-- 带宽/能量选择 -->
  <van-popup :show="showPurposePicker && !loading" destroy-on-close position="bottom">
    <van-picker
      :columns="purposeColumns"
      @confirm="onPurposeConfirm"
      @cancel="showPurposePicker = false"
      destroy-on-close
    />
  </van-popup>

  <!-- 支付密码 -->
  <van-popup v-model:show="showPassword" position="bottom" :style="{ height: '80%' }">
    <Password @password="onPassword" @cancel="cancelPassword" />
  </van-popup>

  <!-- 2FA -->
  <van-popup v-model:show="show2FACode" position="bottom" :style="{ height: '80%' }">
    <Password info="请输入验证器上的数字" @password="on2FACode" @cancel="cancel2FACode" />
  </van-popup>
</template>

<script setup>
import { onMounted, ref, watch } from "vue";
import { useRouter } from "vue-router";
import { showToast } from "vant";
import request from "@/common/request";
import utils from "@/common/utils";
import Password from "@/components/wallet/Password.vue";

const router = useRouter();
const wallet = ref(null);

const showStakePicker = ref(false);
const showPurposePicker = ref(false);

const stakeType = ref([]);
const stakeTypeText = ref("");

const purposeType = ref([]);
const purposeTypeText = ref("");

const amount = ref("");
const loading = ref(false);
const balance = ref(0);
let balanceAmount = 0;

const estimated = ref(false);
const fee = ref(0);
const benifit = ref(0);
const deduct = ref(0);

const showPassword = ref(false);
const password = ref(null);
const code2fa = ref(null);
const show2FACode = ref(false);

const stakeColumns = ref([
  { text: "质押", value: "freeze" },
  { text: "解押", value: "unfreeze" }
]);

const purposeColumns = ref([
  { text: "带宽", value: "BANDWIDTH" },
  { text: "能量", value: "ENERGY" }
]);

onMounted(() => {
  wallet.value = JSON.parse(localStorage.getItem("wallet") || "null");
});