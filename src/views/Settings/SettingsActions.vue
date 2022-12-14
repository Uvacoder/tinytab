<script lang="ts" setup>
import { ref, onMounted } from 'vue'
import { saveAs } from 'file-saver'
import { readAsText } from 'promise-file-reader'
import { useStore } from 'vuex'
import type { EngineItem, VisibleList, ISettings } from '@/types'
import Pannel from '../../components/Pannel.vue'
import { LoadingOutlined as Loading } from '@vicons/antd'
import { MutationType } from '../../store/mutations'
import { useNotification, useDialog, useMessage } from 'naive-ui'
import { FileTrayFullOutline } from '@vicons/ionicons5'
import { useI18n } from 'vue-i18n'
import {
    DownloadOutline as Download,
    ArrowUpOutline as ArrowUp,
    CheckmarkCircleSharp as Success,
} from '@vicons/ionicons5'
import {
    theme as _theme,
    primaryColor as _primaryColor,
    homeBackground as _homeBackground,
    followSystemTheme as _followSystemTheme,
    visibleList as _visibleList,
    lang as _lang,
    homeBackgroundMask as _homeBackgroundMask,
    homeBackgroundBlur as _homeBackgroundBlur,
    target as _target,
} from '@/data'

const { t } = useI18n()
const dialog = useDialog()
const store = useStore()
const message = useMessage()
const notification = useNotification()

/**
 * Processing export configuration
 */
function handleExportSettings() {
    const { getters } = store
    const settings: ISettings = {
        theme: getters.GetTheme,
        primaryColor: getters.GetPrimaryColor,
        visibleList: getters.GetVisibleList,
        followSystemTheme: getters.GetFollowSystemTheme,
        lang: getters.GetLang,
        homeBackground: getters.GetHomeBackground,
        enginesData: getters.GetEnginesData,
        homeBackgroundMask: getters.GetHomeBackgroundMask,
        homeBackgroundBlur: getters.GetHomeBackgroundBlur,
        target: getters.GetTarget,
    }

    const blob = new Blob([JSON.stringify(settings)], { type: '' })
    saveAs(blob, `tinytab.settings.${new Date().getTime()}.json`)

    const msg: any = message.info(t('settingActions.generating'), {
        closable: true,
        duration: 5000,
    })
    setTimeout(() => {
        msg.type = 'success'
        msg.content = t('settingActions.generated')
    }, 500)
}

/**
 * Processing import configuration
 */
function handleImportSettings() {
    const fileInput = document.getElementById('file-input')

    dialog.warning({
        title: t('title.warn'),
        content: t('dialog.settingActionsDesc'),
        positiveText: t('button.continue'),
        negativeText: t('button.cancel'),
        onPositiveClick: () => fileInput?.click(),
    })
}

/**
 * Import configuration callback
 */
function generateSettings(e: Event) {
    const target = e.target as HTMLInputElement
    const fileMetaData: File = (target.files as FileList)[0]

    if (fileMetaData) {
        readAsText(fileMetaData)
            .then((res) => {
                const data = JSON.parse(res) as ISettings

                //The new configuration needs to set the default value to prevent the import of the old version configuration file from reporting errors
                //Force the imported configuration object to implement the interface ISettings
                //Consistent with the exported configuration items
                const settings: ISettings = {
                    enginesData: data.enginesData || [],
                    theme: data.theme || _theme,
                    primaryColor: data.primaryColor || _primaryColor,
                    homeBackground: data.homeBackground || _homeBackground,
                    followSystemTheme: data.followSystemTheme || _followSystemTheme,
                    visibleList: data.visibleList || _visibleList,
                    lang: data.lang || _lang,
                    homeBackgroundMask: data.homeBackgroundMask || _homeBackgroundMask,
                    homeBackgroundBlur: data.homeBackgroundBlur || _homeBackgroundBlur,
                    target: data.target || _target,
                }

                store.commit(MutationType.UpdateEnginesData, settings.enginesData)
                store.commit(MutationType.UpdateTheme, settings.theme)
                store.commit(MutationType.UpdatePrimaryColor, settings.primaryColor)
                store.commit(MutationType.UpdateHomeBackground, settings.homeBackground)
                store.commit(MutationType.UpdateFollowSystemTheme, settings.followSystemTheme)
                store.commit(MutationType.UpdateVisibleList, settings.visibleList)
                store.commit(MutationType.UpdateLang, settings.lang)
                store.commit(MutationType.UpdateHomeBackgroundMask, settings.homeBackgroundMask)
                store.commit(MutationType.UpdateHomeBackgroundBlur, settings.homeBackgroundBlur)
                store.commit(MutationType.UpdateTarget, settings.target)

                notification.success({
                    content: t('message.importSuccess'),
                    meta: t('title.tip'),
                    duration: 3000,
                })
            })
            .catch((err: any) => {
                notification.error({
                    content: t('message.unknownError'),
                    meta: t('title.tip'),
                    duration: 3000,
                })
            })
    }
}

onMounted(() => {
    const fileInput = document.getElementById('file-input')
    fileInput?.addEventListener('change', generateSettings, false)
})
</script>

<template>
    <pannel :title="t('settingActions.title')" :desc="t('settingActions.desc')">
        <template #icon>
            <n-icon>
                <FileTrayFullOutline></FileTrayFullOutline>
            </n-icon>
        </template>

        <div class="flex">
            <n-button @click="handleImportSettings" class="flex-1 mr-2">
                <template #icon>
                    <n-icon>
                        <download />
                    </n-icon>
                </template>
                {{ t('settingActions.importButton') }}
            </n-button>
            <n-button type="primary" @click="handleExportSettings" class="flex-1 ml-2">
                <template #icon>
                    <n-icon>
                        <arrow-up />
                    </n-icon>
                </template>
                {{ t('settingActions.exportButton') }}
            </n-button>
        </div>
    </pannel>
    <input type="file" id="file-input" class="hidden" />
</template>

<style scoped>
.spin {
    animation: spin 1s linear infinite;
}
@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}
</style>
