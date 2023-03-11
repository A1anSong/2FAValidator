<template>
  <el-row justify="center">
    <el-col :span="22">
      <el-card shadow="hover">
        <template #header>
          <div class="card-header">
            <span>2FA生成器 by A1an</span>
            <el-progress type="circle" :stroke-width="4" :width="48" :percentage="secondsLeft/30*100" :color="colors">
              <template #default="percentage">
                <span style="font-size: var(--el-font-size-base)">{{ secondsLeft }}s</span>
              </template>
            </el-progress>
          </div>
        </template>
        <el-form label-position="right" label-width="100px">
          <el-form-item label="密钥：">
            <el-input v-model.trim="secret"/>
          </el-form-item>
          <el-form-item label="OTP：">
            <el-tag effect="dark" v-if="token !== ''">{{ token }}</el-tag>
          </el-form-item>
          <el-form-item label="暗黑模式：">
            <el-switch style="--el-switch-on-color: #73767a; --el-switch-off-color: #f4f4f5" v-model="isDark"/>
          </el-form-item>
        </el-form>
      </el-card>
    </el-col>
  </el-row>
</template>

<script setup>
import {computed, ref, watch} from "vue";
import * as tfa from 'node-2fa';
import {useDark} from '@vueuse/core'

const isDark = useDark()

const colors = [
  {color: '#F56C6C', percentage: 20},
  {color: '#E6A23C', percentage: 50},
  {color: '#67C23A', percentage: 100}
]

const secondsNow = ref(0)
const secondsLeft = computed(() => 30 - secondsNow.value % 30)
const secret = ref('')
const token = ref('')
watch(secret, () => {
  getToken()
})

secondsNow.value = Math.floor(Date.now() / 1000)
setInterval(() => {
  secondsNow.value += 1
  if (secondsLeft.value === 30) {
    getToken()
  }
}, 1000)

const params = new URLSearchParams(location.search)
const secretParams = params.get('secret')
if (secretParams !== null) {
  secret.value = secretParams
}

const getToken = () => {
  if (secret.value === '') {
    token.value = ''
    return
  }
  token.value = tfa.generateToken(secret.value).token
}
</script>

<style scoped>
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>