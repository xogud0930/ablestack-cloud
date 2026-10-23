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
  <a-row :gutter="12" class="infra-summary-root">
    <a-col :md="24">
      <a-card class="breadcrumb-card">
        <a-col :md="24" style="display: flex">
          <breadcrumb style="padding-top: 6px; padding-left: 8px" />
          <a-button
            style="margin-left: 12px; margin-top: 4px"
            :loading="loading"
            size="small"
            shape="round"
            @click="fetchData()" >
            <template #icon><ReloadOutlined /></template>
            {{ $t('label.refresh') }}
          </a-button>
          <a-button
            style="margin-left: 12px; margin-top: 4px"
            size="small"
            shape="round"
            @click="sslFormVisible = true">
            <template #icon><SafetyCertificateOutlined /></template>
            {{ $t('label.sslcertificates') }}
          </a-button>
          <a-modal
            v-if="sslFormVisible"
            :title="$t('label.sslcertificates')"
            :visible="sslFormVisible"
            :footer="null"
            :maskClosable="false"
            @cancel="sslModalClose">
            <p>
              {{ $t('message.update.ssl') }}
            </p>
            <a-form
              layout="vertical"
              :ref="formRef"
              :model="form"
              :rules="rules"
              @finish="handleSslFormSubmit"
              v-ctrl-enter="handleSslFormSubmit"
             >
              <a-form-item name="root" ref="root" :required="true">
                <template #label>
                  <tooltip-label :title="$t('label.root.certificate')" :tooltip="apiParams.name.description" tooltipPlacement="bottom"/>
                </template>
                <a-textarea
                  id="rootCert"
                  rows="2"
                  :placeholder="apiParams.name.description"
                  v-focus="true"
                  name="rootCert"
                  v-model:value="form.root"
                ></a-textarea>
              </a-form-item>

              <transition-group name="fadeInUp" tag="div">
                <a-form-item
                  v-for="(item, index) in intermediateCertificates"
                  :key="`key-${index}`"
                  :name="`intermediate${index + 1}`"
                  :ref="`intermediate${index + 1}`"
                  class="intermediate-certificate">
                  <template #label>
                    <tooltip-label :title="$t('label.intermediate.certificate') + ` ${index + 1} `" :tooltip="apiParams.id.description" tooltipPlacement="bottom"/>
                  </template>
                  <a-textarea
                    :id="`intermediateCert${index}`"
                    rows="2"
                    :placeholder="$t('label.intermediate.certificate') + ` ${index + 1}`"
                    :name="`intermediateCert${index}`"
                    v-model:value="form[`intermediate${index + 1}`]"
                  ></a-textarea>
                </a-form-item>
              </transition-group>

              <a-form-item>
                <a-button @click="addIntermediateCert">
                  <plus-circle-outlined />
                  {{ $t('label.add.intermediate.certificate') }}
                </a-button>
              </a-form-item>

              <a-form-item name="server" ref="server" :required="true">
                <template #label>
                  <tooltip-label :title="$t('label.server.certificate')" :tooltip="apiParams.certificate.description" tooltipPlacement="bottom"/>
                </template>
                <a-textarea
                  id="serverCert"
                  rows="2"
                  :placeholder="apiParams.certificate.description"
                  name="serverCert"
                  v-model:value="form.server"
                ></a-textarea>
              </a-form-item>

              <a-form-item name="pkcs" ref="pkcs" :required="true">
                <template #label>
                  <tooltip-label :title="$t('label.pkcs.private.certificate')" :tooltip="apiParams.privatekey.description" tooltipPlacement="bottom"/>
                </template>
                <a-textarea
                  id="pkcsKey"
                  rows="2"
                  :placeholder="apiParams.privatekey.description"
                  name="pkcsKey"
                  v-model:value="form.pkcs"
                ></a-textarea>
              </a-form-item>

              <a-form-item name="dns" ref="dns" :required="true">
                <template #label>
                  <tooltip-label :title="$t('label.domain.suffix')" :tooltip="apiParams.domainsuffix.description" tooltipPlacement="bottom"/>
                </template>
                <a-input
                  id="dnsSuffix"
                  :placeholder="apiParams.domainsuffix.description"
                  name="dnsSuffix"
                  v-model:value="form.dns"
                ></a-input>
              </a-form-item>
              <div :span="24" class="action-button">
                <a-button @click="sslModalClose" class="close-button">
                  {{ $t('label.cancel' ) }}
                </a-button>
                <a-button type="primary" ref="submit" :loading="sslFormSubmitting" @click="handleSslFormSubmit">
                  {{ $t('label.submit' ) }}
                </a-button>
              </div>
            </a-form>
          </a-modal>
        </a-col>
      </a-card>
    </a-col>
    <a-col :xs="24">
      <div class="infra-summary-layout" :class="{ 'is-rack-expanded': rackVisualizationExpanded }">
        <div class="summary-cards-pane" :aria-hidden="rackVisualizationExpanded">
          <section
            v-for="group in visibleSummaryGroups"
            :key="group.key"
            class="summary-group"
          >
            <div class="summary-group-title">{{ getSummaryGroupTitle(group) }}</div>
            <div class="summary-tiles">
              <router-link
                v-for="section in group.items"
                :key="section"
                class="summary-tile"
                :class="{ 'is-alert': section === 'alerts' && stats[section] > 0 }"
                :to="{ name: section === 'backuprepositories' ? 'backuprepository' : section.substring(0, section.length - 1) }"
              >
                <span class="summary-tile__icon"><render-icon :icon="routes[section].icon" /></span>
                <span class="summary-tile__body">
                  <span class="summary-tile__label" :title="$t(routes[section].title)">{{ $t(routes[section].title) }}</span>
                  <span v-if="loading" class="summary-tile__value is-loading" />
                  <span v-else-if="stats[section] !== undefined" class="summary-tile__value">{{ stats[section] }}</span>
                </span>
                <right-outlined class="summary-tile__arrow" />
              </router-link>
            </div>
          </section>
        </div>
        <div class="rack-visualization-pane">
          <a-card :bordered="false" class="rack-visualization-card">
            <template #title>{{ $t('rackDiagram.title') }}</template>
            <rack-diagram-tab
              @toggle-expand="rackVisualizationExpanded = $event"
            />
          </a-card>
        </div>
      </div>
    </a-col>
  </a-row>
</template>

<script>
import { ref, reactive, toRaw } from 'vue'
import { RightOutlined } from '@ant-design/icons-vue'
import { getAPI, postAPI } from '@/api'
import router from '@/router'

import Breadcrumb from '@/components/widgets/Breadcrumb'
import TooltipLabel from '@/components/widgets/TooltipLabel'
import RackDiagramTab from '@/views/infra/zone/RackDiagramTab.vue'

export default {
  name: 'InfraSummary',
  components: {
    Breadcrumb,
    RightOutlined,
    TooltipLabel,
    RackDiagramTab
  },
  data () {
    return {
      loading: true,
      rackVisualizationExpanded: false,
      routes: {},
      sections: ['zones', 'pods', 'clusters', 'hosts', 'storagepools', 'imagestores', 'backuprepositories', 'objectstores', 'systemvms', 'routers', 'cpusockets', 'managementservers', 'alerts', 'ilbvms', 'metrics'],
      summaryGroups: [
        { key: 'configuration', titleParts: ['label.infrastructure', 'label.configuration'], items: ['zones', 'pods', 'clusters'] },
        { key: 'compute', title: 'label.compute', items: ['hosts', 'cpusockets', 'systemvms', 'metrics'] },
        { key: 'network', title: 'label.network', items: ['routers', 'ilbvms'] },
        { key: 'storage', title: 'label.storage', items: ['storagepools', 'imagestores', 'backuprepositories', 'objectstores'] },
        { key: 'incident', title: 'label.alerts', items: ['alerts'] }
      ],
      sslFormVisible: false,
      stats: {},
      intermediateCertificates: [],
      sslFormSubmitting: false,
      maxCerts: 0
    }
  },
  beforeCreate () {
    this.apiParams = this.$getApiParams('uploadCustomCertificate')
  },
  created () {
    this.initForm()
    this.fetchData()
  },
  computed: {
    visibleSummaryGroups () {
      return this.summaryGroups
        .map(group => {
          const items = group.items.filter(section => this.routes[section])
          return { ...group, items }
        })
        .filter(group => group.items.length)
    }
  },
  methods: {
    getSummaryGroupTitle (group) {
      if (group.titleParts) {
        return group.titleParts.map(key => this.$t(key)).join(' ')
      }
      return this.$t(group.title)
    },
    initForm () {
      this.formRef = ref()
      this.form = reactive({})
      this.rules = reactive({
        root: [{ required: true, message: this.$t('label.required') }],
        server: [{ required: true, message: this.$t('label.required') }],
        pkcs: [{ required: true, message: this.$t('label.required') }],
        dns: [{ required: true, message: this.$t('label.required') }]
      })
    },
    fetchData () {
      this.routes = {}
      for (const section of this.sections) {
        const route = section === 'backuprepositories' ? 'backuprepository' : section.substring(0, section.length - 1)
        // Skip sections the current user cannot access (route not registered)
        if (!router.hasRoute(route)) {
          continue
        }
        if (router.resolve('/' + route).matched[0].redirect === '/exception/404') {
          continue
        }
        const node = router.resolve({ name: route })
        this.routes[section] = {
          title: node.meta.title,
          icon: node.meta.icon
        }
      }
      this.listInfra()
    },
    listInfra () {
      this.loading = true
      getAPI('listInfrastructure').then(json => {
        this.stats = []
        if (json && json.listinfrastructureresponse && json.listinfrastructureresponse.infrastructure) {
          this.stats = json.listinfrastructureresponse.infrastructure
        }
      }).finally(f => {
        this.loading = false
      })
    },

    resetSslFormData () {
      this.formRef.value.resetFields()
      this.intermediateCertificates = []
      this.sslFormSubmitting = false
      this.sslFormVisible = false
    },

    sslModalClose () {
      this.resetSslFormData()
    },

    addIntermediateCert () {
      this.intermediateCertificates.push('')
    },

    pollActionCompletion (jobId, count) {
      getAPI('queryAsyncJobResult', { jobid: jobId }).then(json => {
        const result = json.queryasyncjobresultresponse
        if (result.jobstatus === 1 && this.maxCerts === count) {
          this.$message.success(`${this.$t('label.certificate.upload')}: ${result.jobresult.customcertificate.message}`)
          this.$notification.success({
            message: this.$t('label.certificate.upload'),
            description: result.jobresult.customcertificate.message || this.$t('message.success.certificate.upload')
          })
        } else if (result.jobstatus === 2) {
          this.$notification.error({
            message: this.$t('label.certificate.upload.failed'),
            description: result.jobresult.errortext || this.$t('label.certificate.upload.failed.description'),
            duration: 0
          })
        } else if (result.jobstatus === 0) {
          this.$message
            .loading(`${this.$t('message.certificate.upload.processing')}: ${count}`, 2)
            .then(() => this.pollActionCompletion(jobId, count))
        }
      }).catch(e => {
        console.log(this.$t('error.fetching.async.job.result') + e)
      })
    },

    handleSslFormSubmit () {
      if (this.sslFormSubmitting) return
      this.sslFormSubmitting = true

      this.formRef.value.validate().then(() => {
        const formValues = toRaw(this.form)

        this.maxCerts = 2 + Object.keys(formValues).length
        let count = 1
        let data = {
          id: count,
          certificate: formValues.root,
          domainsuffix: formValues.dns,
          name: 'root'
        }
        postAPI('uploadCustomCertificate', data).then(response => {
          this.pollActionCompletion(response.uploadcustomcertificateresponse.jobid, count)
        }).then(() => {
          this.sslModalClose()
        })

        Object.keys(formValues).forEach(key => {
          if (key.includes('intermediate')) {
            count = count + 1
            const data = {
              id: count,
              certificate: formValues[key],
              domainsuffix: formValues.dns,
              name: key
            }
            postAPI('uploadCustomCertificate', data).then(response => {
              this.pollActionCompletion(response.uploadcustomcertificateresponse.jobid, count)
            }).then(() => {
              this.sslModalClose()
            })
          }
        })

        count = count <= 2 ? 3 : count + 1
        data = {
          id: count,
          certificate: formValues.server,
          domainsuffix: formValues.dns,
          privatekey: formValues.pkcs
        }
        postAPI('uploadCustomCertificate', data).then(response => {
          this.pollActionCompletion(response.uploadcustomcertificateresponse.jobid, count)
        }).then(() => {
          this.sslModalClose()
        })
      }).catch(error => {
        this.formRef.value.scrollToField(error.errorFields[0].name)
      }).finally(() => { this.sslFormSubmitting = false })
    }
  }
}
</script>

<style lang="scss" scoped>

  .breadcrumb-card {
    margin-left: -24px;
    margin-right: -24px;
    margin-top: -16px;
    margin-bottom: 12px;
  }

  .infra-summary-layout {
    display: flex;
    align-items: flex-start;
    gap: 12px;
    transition: gap 0.28s cubic-bezier(0.22, 1, 0.36, 1);
  }
  .summary-cards-pane {
    width: 34%;
    max-width: 34%;
    flex: 0 0 34%;
    max-height: 5000px;
    padding: 16px;
    overflow: hidden;
    background: var(--ui-bg-surface);
    border: 1px solid var(--ui-border);
    border-radius: 12px;
    transition:
      width 0.28s cubic-bezier(0.22, 1, 0.36, 1),
      max-width 0.28s cubic-bezier(0.22, 1, 0.36, 1),
      flex-basis 0.28s cubic-bezier(0.22, 1, 0.36, 1),
      max-height 0.28s cubic-bezier(0.22, 1, 0.36, 1),
      padding 0.28s cubic-bezier(0.22, 1, 0.36, 1),
      opacity 0.2s ease,
      transform 0.28s cubic-bezier(0.22, 1, 0.36, 1),
      visibility 0s linear;
  }

  .infra-summary-layout.is-rack-expanded {
    gap: 0;
  }

  .infra-summary-layout.is-rack-expanded .summary-cards-pane {
    width: 0;
    max-width: 0;
    flex: 0 0 0;
    max-height: 0;
    padding: 0;
    border-width: 0;
    opacity: 0;
    visibility: hidden;
    pointer-events: none;
    transform: translateX(-16px);
    transition-delay: 0s, 0s, 0s, 0s, 0s, 0s, 0s, 0.28s;
  }
  .summary-group + .summary-group {
    margin-top: 18px;
  }
  .summary-group-title {
    margin: 0 2px 8px;
    color: var(--ui-text-muted);
    font-size: 12px;
    font-weight: 600;
    line-height: 18px;
  }
  .summary-tiles {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(148px, 1fr));
    gap: 8px;
  }
  .summary-tile {
    display: flex;
    align-items: center;
    gap: 10px;
    min-width: 0;
    min-height: 60px;
    padding: 10px 10px 10px 12px;
    color: var(--ui-text-primary);
    background: var(--ui-bg-page);
    border: 1px solid transparent;
    border-radius: 10px;
    transition: background-color 0.15s ease, border-color 0.15s ease;

    &:hover {
      background: var(--ui-bg-hover);
      border-color: var(--ui-border);

      .summary-tile__arrow {
        opacity: 1;
      }
    }

    &:focus-visible {
      outline: 2px solid var(--ui-focus);
      outline-offset: 1px;
    }
  }
  .summary-tile__icon {
    display: inline-flex;
    flex: none;
    align-items: center;
    justify-content: center;
    width: 32px;
    height: 32px;
    color: var(--ui-info-icon);
    font-size: 16px;
    background: var(--ui-info-bg);
    border-radius: 8px;
  }
  .summary-tile__body {
    display: flex;
    flex: 1;
    flex-direction: column;
    min-width: 0;
  }
  .summary-tile__label {
    overflow: hidden;
    color: var(--ui-text-muted);
    font-size: 12px;
    line-height: 18px;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
  .summary-tile__value {
    font-size: 20px;
    font-weight: 600;
    line-height: 26px;
    font-variant-numeric: tabular-nums;

    &.is-loading {
      width: 28px;
      height: 16px;
      margin: 5px 0;
      background: var(--ui-border);
      border-radius: 4px;
    }
  }
  .summary-tile__arrow {
    flex: none;
    color: var(--ui-text-placeholder);
    font-size: 11px;
    opacity: 0;
    transition: opacity 0.15s ease;
  }
  .summary-tile.is-alert {
    .summary-tile__icon {
      color: var(--ui-warning-icon);
      background: var(--ui-warning-bg);
    }
    .summary-tile__value {
      color: var(--ui-warning-icon);
    }
  }
  .rack-visualization-pane {
    width: 66%;
    flex: 0 1 66%;
    min-width: 320px;
    transition:
      width 0.28s cubic-bezier(0.22, 1, 0.36, 1),
      flex-basis 0.28s cubic-bezier(0.22, 1, 0.36, 1);
  }
  .infra-summary-layout.is-rack-expanded .rack-visualization-pane {
    width: 100%;
    flex: 1 1 100%;
    min-width: 0;
  }
  .rack-visualization-card {
    background: var(--ui-bg-surface);
    border: 1px solid var(--ui-border);
    border-radius: 12px;

    :deep(.ant-card-head) {
      color: var(--ui-text-primary);
      background: transparent;
      border-bottom-color: var(--ui-border);
    }

    :deep(.ant-card-body) {
      padding: 12px;
      background: transparent;
    }
  }
  @media (max-width: 1200px) {
    .infra-summary-layout {
      flex-direction: column;
    }
    .summary-cards-pane,
    .rack-visualization-pane {
      width: 100%;
      max-width: 100%;
      flex: 1 1 100%;
      min-width: 0;
    }
  }

  @media (prefers-reduced-motion: reduce) {
    .infra-summary-layout,
    .summary-cards-pane,
    .rack-visualization-pane {
      transition: none;
    }
  }
  .intermediate-certificate {
    opacity: 1;
    transform: none;
    transition: opacity 0.2s ease 0s, transform 0.5s ease;
    will-change: transform;
  }
  .intermediate-certificate.fadeInUp-enter-active {
    opacity: 0;
    transform: translateY(10px);
    transition: none;
  }
  .controls {
    display: flex;
    justify-content: flex-end;
  }
  .close-button {
    margin-right: 20px;
  }
  .ant-form-item {
    margin-bottom: 10px;
  }
</style>
