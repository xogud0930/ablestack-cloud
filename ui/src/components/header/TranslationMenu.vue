// Licensed to the Apache Software Foundation (ASF) under one
// or more contributor license agreements.  See the NOTICE file
// distributed with this work for additional information
// regarding copyright ownership.  The ASF licenses this file
// to you under the Apache License, Version 2.0 (the
// "License"); you may not use this file except in compliance
// with the License.  You may obtain a copy of the License at
//
//   http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing,
// software distributed under the License is distributed on an
// "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
// KIND, either express or implied.  See the License for the
// specific language governing permissions and limitations
// under the License.

<template>
  <a-dropdown overlayClassName="translation-menu-dropdown">
    <span
      class="action ant-dropdown-link translation-menu"
      :class="{ 'translation-menu--labeled': showLabel }">
      <TranslationOutlined />
      <template v-if="showLabel">
        <span class="translation-menu__label">{{ currentLanguageName }}</span>
        <DownOutlined class="translation-menu__caret" />
      </template>
    </span>
    <template #overlay>
      <a-menu
        :selectedKeys="[language]"
        @click="onClick">
        <a-menu-item
          v-for="item in languages"
          :key="item.key"
          :value="item.value">{{ item.name }}</a-menu-item>
      </a-menu>
    </template>
  </a-dropdown>
</template>

<script>
import moment from 'moment'
import 'moment/locale/zh-cn'
import { loadLanguageAsync } from '@/locales'
import { vueProps } from '@/vue-app'

moment.locale('ko_KR')

const LANGUAGES = [
  { key: 'ko_KR', value: 'koKR', name: '한국어' },
  { key: 'en', value: 'enUS', name: 'English' },
  { key: 'hi', value: 'hi', name: 'हिन्दी' },
  { key: 'te', value: 'te', name: 'తెలుగు' },
  { key: 'ja_JP', value: 'jpJP', name: '日本語' },
  { key: 'zh_CN', value: 'zhCN', name: '简体中文' },
  { key: 'ar', value: 'arEG', name: 'Arabic' },
  { key: 'ca', value: 'caES', name: 'Catalan' },
  { key: 'de_DE', value: 'deDE', name: 'Deutsch' },
  { key: 'es', value: 'esES', name: 'Español' },
  { key: 'fr_FR', value: 'frFR', name: 'Français' },
  { key: 'it_IT', value: 'itIT', name: 'Italiano' },
  { key: 'hu', value: 'huHU', name: 'Magyar' },
  { key: 'nl_NL', value: 'nlNL', name: 'Nederlands' },
  { key: 'nb_NO', value: 'nbNO', name: 'Norsk' },
  { key: 'pl', value: 'plPL', name: 'Polish' },
  { key: 'pt_BR', value: 'ptBR', name: 'Português brasileiro' },
  { key: 'ru_RU', value: 'ruRU', name: 'Русский' },
  { key: 'el_GR', value: 'elGR', name: 'Ελληνικά' }
]

export default {
  name: 'TranslationMenu',
  props: {
    // show the current language name next to the icon (login screen)
    showLabel: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      language: 'ko_KR',
      languages: LANGUAGES
    }
  },
  computed: {
    currentLanguageName () {
      return LANGUAGES.find(item => item.key === this.language)?.name || this.language
    }
  },
  mounted () {
    this.language = this.$localStorage.get('LOCALE') || vueProps.$config?.defaultLanguage || 'ko_KR'
    this.setLocale(this.language)
  },
  methods: {
    moment,
    onClick (e) {
      let localeValue = e.key
      if (!localeValue) {
        localeValue = 'ko_KR'
      }
      this.setLocale(localeValue)
    },
    setLocale (localeValue) {
      this.$i18n.locale = localeValue
      this.language = localeValue
      moment.locale(localeValue)
      this.$localStorage.set('LOCALE', localeValue)
      loadLanguageAsync(localeValue)
    }
  }
}
</script>

<style lang="less" scoped>
.translation-menu {
  font-size: 18px;
  line-height: 1;
}

.translation-menu--labeled {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  height: 32px;
  padding: 0 10px;
  color: var(--ui-text-secondary);
  font-size: 14px;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s, color 0.2s;

  &:hover {
    color: var(--ui-text-primary);
    background: var(--ui-bg-hover);
  }

  .translation-menu__caret {
    font-size: 10px;
    opacity: 0.6;
  }
}

</style>

<style lang="less">
// 19 languages: keep the list short enough to open below or above the trigger instead of covering the page
.translation-menu-dropdown .ant-dropdown-menu {
  max-height: ~"min(60vh, 360px)";
  overflow-y: auto;
}
</style>
