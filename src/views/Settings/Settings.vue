<script lang="ts" setup>
import { ref, computed, watch, onMounted } from 'vue'
import { SettingsOutline, LanguageOutline } from '@vicons/ionicons5'
import EnginesEditor from './EnginesEdit.vue'
import ModeSetting from './ModeSetting.vue'
import EnginesSort from './EnginesSort.vue'
import SettingsActions from './SettingsActions.vue'
import VisibilitySettings from './VisibilitySettings.vue'
import BackgroundSetting from './BackgroundSetting.vue'
import TargetSetting from './TargetSetting.vue'
import { useStore } from 'vuex'
import { useI18n } from 'vue-i18n'
import { MutationType } from '@/store/mutations'
import { useMessage } from 'naive-ui'
import type { VisibleList } from '@/types'

const message = useMessage()
const { t, locale } = useI18n()
const store = useStore()
const drawerVisible = ref(false)

const homeSettingButtonVisible = computed(() =>
    store.getters.GetVisibleList.includes('homeSettingButton')
)

const homeLangButtonVisible = computed(() =>
    store.getters.GetVisibleList.includes('homeLangButton')
)

function changeLang() {
    const currentLang = store.getters.GetLang
    store.commit(MutationType.UpdateLang, currentLang === 'en' ? 'zh-CN' : 'en')
}

watch(
    () => store.getters.GetLang,
    (v) => {
        locale.value = v
        message.info(t('message.toggleLang'))
    }
)

onMounted(() => {
    locale.value = store.getters.GetLang
})
</script>

<template>
    <div class="!absolute top-6 right-6 flex">
        <div class="mr-2 text-center w-8 h-8 leading-10 cursor-pointer" @click="changeLang">
            <div :class="{ hidden: !homeLangButtonVisible }">
                <n-icon size="20" class="text-gray-500">
                    <LanguageOutline></LanguageOutline>
                </n-icon>
            </div>
        </div>
        <div
            class="group text-center w-8 h-8 leading-10 cursor-pointer"
            @click="drawerVisible = true"
        >
            <div class="group-hover:block" :class="{ hidden: !homeSettingButtonVisible }">
                <n-icon size="20" class="text-gray-500">
                    <settings-outline></settings-outline>
                </n-icon>
            </div>
        </div>
    </div>

    <n-drawer placement="right" class="cus-drawer" v-model:show="drawerVisible" :width="400">
        <n-drawer-content body-content-style="padding:0;" closable>
            <template #header>
                <div class="flex items-center">
                    <span>{{ t('title.settings') }}</span>
                </div>
            </template>

            <engines-editor />
            <engines-sort />
            <mode-setting />
            <background-setting />
            <target-setting />
            <visibility-settings />
            <settings-actions />
        </n-drawer-content>
    </n-drawer>
</template>

<style>
/* windcss doesn't work <md:!w-full */
@media (max-width: 767.9px) {
    .cus-drawer {
        width: 100% !important;
    }
}
</style>
