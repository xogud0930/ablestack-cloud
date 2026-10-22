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
  <div class="dashboard-feed">
    <div v-for="item in items" :key="item.id" class="dashboard-feed__item">
      <span class="dashboard-feed__dot" :class="'is-' + tone(item)" />
      <div class="dashboard-feed__main">
        <div class="dashboard-feed__head">
          <router-link class="dashboard-feed__type" :to="{ path: '/' + kind + '/' + item.id }">
            {{ kind === 'alert' ? item.name : item.type }}
          </router-link>
          <span class="dashboard-feed__time">{{ $toLocaleDate(kind === 'alert' ? item.sent : item.created) }}</span>
        </div>
        <div class="dashboard-feed__desc" :title="item.description">{{ item.description }}</div>
        <div v-if="kind === 'event' && (item.resourcename || item.username)" class="dashboard-feed__meta">
          <resource-label
            v-if="item.resourcename"
            :resourceType="item.resourcetype"
            :resourceId="item.resourceid"
            :resourceName="item.resourcename" />
          <span v-if="item.username && item.username !== item.resourcename">{{ item.username }}</span>
        </div>
      </div>
    </div>
    <a-empty v-if="!items.length" :image="simpleEmptyImage" :description="$t('label.no.data')" />
  </div>
</template>

<script>
import { Empty } from 'ant-design-vue'
import ResourceLabel from '@/components/widgets/ResourceLabel'

export default {
  name: 'DashboardFeed',
  components: {
    // a-empty is not in the global component list
    AEmpty: Empty,
    ResourceLabel
  },
  props: {
    // 'alert' or 'event'; picks the title/time fields and the detail route
    kind: {
      type: String,
      required: true
    },
    items: {
      type: Array,
      default: () => []
    }
  },
  data () {
    return {
      simpleEmptyImage: Empty.PRESENTED_IMAGE_SIMPLE
    }
  },
  methods: {
    tone (item) {
      if (this.kind === 'alert') {
        const text = (item.description || '').toLowerCase()
        return text.includes('fail') || text.includes('error') ? 'error' : 'warning'
      }
      if (item.level === 'ERROR') return 'error'
      if (item.state === 'Completed') return 'success'
      return 'info'
    }
  }
}
</script>

<style lang="less" scoped>
.dashboard-feed__item {
  display: flex;
  gap: 10px;
  padding: 10px 0;

  & + & {
    border-top: 1px solid var(--ui-border);
  }
}

.dashboard-feed__dot {
  flex: none;
  width: 8px;
  height: 8px;
  margin-top: 6px;
  border-radius: 50%;

  &.is-error { background: var(--ui-error-icon); }
  &.is-warning { background: var(--ui-warning-icon); }
  &.is-success { background: var(--ui-success-icon); }
  &.is-info { background: var(--ui-info-icon); }
}

.dashboard-feed__main {
  flex: 1;
  min-width: 0;
}

.dashboard-feed__head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 8px;
}

.dashboard-feed__type {
  overflow: hidden;
  font-size: 12px;
  font-weight: 600;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.dashboard-feed__time {
  flex: none;
  color: var(--ui-text-muted);
  font-size: 11px;
}

.dashboard-feed__desc {
  display: -webkit-box;
  margin-top: 2px;
  overflow: hidden;
  color: var(--ui-text-secondary);
  font-size: 13px;
  line-height: 1.5;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
}

.dashboard-feed__meta {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 4px 10px;
  margin-top: 4px;
  color: var(--ui-text-muted);
  font-size: 12px;

  :deep(a) {
    color: var(--ui-text-secondary);
  }
}

.ant-empty {
  margin: 32px 0;
}
</style>
