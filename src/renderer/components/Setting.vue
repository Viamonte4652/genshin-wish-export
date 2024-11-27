<template>
  <div class="settings-container bg-gray-900 absolute inset-0 overflow-y-auto">
    <div class="p-6">
      <div class="flex content-center items-center mb-6 justify-between">
        <h3 class="text-lg text-white">{{text.title}}</h3>
        <el-button icon="close" @click="closeSetting" plain circle type="default" class="w-8 h-8 relative -right-2 shadow-md focus:shadow-none focus:outline-none"></el-button>
      </div>
      <div class="settings-form">
        <div class="form-group mb-6">
          <div class="text-sm text-gray-300 mb-2">{{text.language}}</div>
          <el-select @change="saveLang" v-model="settingForm.lang" class="w-64">
            <el-option v-for="item of data.langMap" :key="item[0]" :label="item[1]" :value="item[0]"></el-option>
          </el-select>
          <p class="text-gray-400 text-xs mt-2">{{text.languageHint}}</p>
        </div>

        <div class="form-group mb-6">
          <div class="text-sm text-gray-300 mb-2">{{text.logType}}</div>
          <el-radio-group @change="saveSetting" v-model.number="settingForm.logType">
            <el-radio-button :label="0">{{text.auto}}</el-radio-button>
            <el-radio-button :label="1">{{text.cnServer}}</el-radio-button>
            <el-radio-button :label="2">{{text.seaServer}}</el-radio-button>
            <el-radio-button v-if="settingForm.lang === 'zh-cn'" :label="3">云原神</el-radio-button>
          </el-radio-group>
          <p class="text-gray-400 text-xs mt-2">{{text.logTypeHint}}</p>
        </div>

        <div class="form-group mb-6">
          <div class="text-sm text-gray-300 mb-2">{{text.UIGFLable}}</div>
          <div class="flex space-x-2 items-center">
            <el-button :loading="data.loadingOfUIGFJSON" class="focus:outline-none" plain type="primary" @click="importUIGFJSON">{{ text.UIGFImportButton }}</el-button>
            <el-button :loading="data.loadingOfUIGFJSON" class="focus:outline-none" plain type="success" @click="exportUIGFJSON">{{ text.UIGFButton }}</el-button>
            <el-checkbox v-model="settingForm.readableJSON" @change="saveSetting">{{ text.UIGFReadable }}</el-checkbox>
          </div>
          <p class="text-gray-400 text-xs mt-2 leading-normal">{{ text.UIGFHint }}
            <a class="cursor-pointer text-blue-400"
               @click="openLink(`https://uigf.org/${settingForm.lang.startsWith('zh-') ? 'zh/': 'en/'}`)" >{{ text.UIGFLink }}</a>
          </p>
        </div>

        <div class="form-group mb-6">
          <div class="text-sm text-gray-300 mb-2">{{text.autoUpdate}}</div>
          <el-switch
            @change="saveSetting"
            v-model="settingForm.autoUpdate">
          </el-switch>
        </div>

        <div class="form-group mb-6">
          <div class="text-sm text-gray-300 mb-2">{{text.hideNovice}}</div>
          <el-switch
            @change="saveSetting"
            v-model="settingForm.hideNovice">
          </el-switch>
        </div>

        <div class="form-group mb-6">
          <div class="text-sm text-gray-300 mb-2">{{text.fetchFullHistory}}</div>
          <el-switch
            @change="saveSetting"
            v-model="settingForm.fetchFullHistory">
          </el-switch>
          <p class="text-gray-400 text-xs mt-2">{{text.fetchFullHistoryHint}}</p>
        </div>
      </div>

      <div class="about-section mt-8">
        <h3 class="text-lg mb-4 text-white">{{about.title}}</h3>
        <p class="text-gray-400 text-xs mt-1">{{about.license}}</p>
        <p class="text-gray-400 text-xs mt-1">Github: <a @click="openGithub" class="cursor-pointer text-blue-400">https://github.com/biuuu/genshin-wish-export</a></p>
      </div>
    </div>
  </div>
</template>

<script setup>
const { ipcRenderer, shell } = require('electron')
import { computed, onMounted, reactive, ref } from 'vue'
import { ElMessage } from 'element-plus'

const emit = defineEmits(['close', 'changeLang', 'dataUpdated'])

const props = defineProps({
  i18n: Object
})

const data = reactive({
  langMap: new Map(),
  loadingOfUIGFJSON: false
})

const settingForm = reactive({
  lang: 'zh-cn',
  logType: 1,
  proxyMode: true,
  autoUpdate: true,
  fetchFullHistory: false,
  hideNovice: true,
  gistsToken: '',
  readableJSON: false
})

const text = computed(() => props.i18n.ui.setting)
const about = computed(() => props.i18n.ui.about)

const saveSetting = async () => {
  const keys = ['lang', 'logType', 'proxyMode', 'autoUpdate', 'fetchFullHistory', 'hideNovice', 'gistsToken', 'readableJSON']
  for (let key of keys) {
    await ipcRenderer.invoke('SAVE_CONFIG', [key, settingForm[key]])
  }
}

const saveLang = async () => {
  await saveSetting()
  emit('changeLang')
}

const closeSetting = () => {
  emit('close')
}

const openGithub = () => {
  shell.openExternal('https://github.com/biuuu/genshin-wish-export')
}

const openLink = (link) => {
  shell.openExternal(link)
}

const exportUIGFJSON = async () => {
  try {
    data.loadingOfUIGFJSON = true
    await ipcRenderer.invoke('EXPORT_UIGF_JSON')
    ElMessage.success(text.UIGFSuccess)
  } catch (e) {
    ElMessage.error(text.UIGFFailed)
  } finally {
    data.loadingOfUIGFJSON = false
  }
}

const importUIGFJSON = async () => {
  try {
    data.loadingOfUIGFJSON = true
    await ipcRenderer.invoke('IMPORT_UIGF_JSON')
    ElMessage.success(text.UIGFImportSuccess)
    emit('dataUpdated')
  } catch (e) {
    if (e.message === 'cancel') {
      ElMessage.info(text.UIGFCancel)
    } else {
      ElMessage.error(text.UIGFImportFailed)
    }
  } finally {
    data.loadingOfUIGFJSON = false
  }
}

onMounted(async () => {
  data.langMap = await ipcRenderer.invoke('LANG_MAP')
  const config = await ipcRenderer.invoke('GET_CONFIG')
  Object.assign(settingForm, config)
})
</script>

<style>
.settings-container {
  top: 32px;
  height: calc(100% - 32px);
}

.settings-form :deep(.el-radio-button__inner) {
  background-color: #2d2d2d;
  border-color: #4a4a4a;
  color: #e0e0e0;
  height: 32px;
  padding: 0 15px;
  font-size: 14px;
}

.settings-form :deep(.el-radio-button__original-radio:checked + .el-radio-button__inner) {
  background-color: #409eff;
  border-color: #409eff;
  color: white;
}

.settings-form :deep(.el-switch__core) {
  background-color: #4a4a4a;
  border-color: #4a4a4a;
}

.settings-form :deep(.el-switch.is-checked .el-switch__core) {
  background-color: #409eff;
  border-color: #409eff;
}

.about-section {
  border-top: 1px solid #3f3f3f;
  padding-top: 20px;
}
</style>
