<template>
  <b-tab :title="$t('chart.label')">
    <b-form-group
      :label="$t('chart.display')"
      label-class="text-primary"
    >
      <b-input-group class="d-flex w-100">
        <vue-select
          v-model="block.options.chartID"
          :options="charts"
          :get-option-key="getOptionKey"
          :placeholder="$t('chart.pick')"
          :reduce="option => option.chartID"
          :calculate-position="calculateDropdownPosition"
          label="name"
          append-to-body
          :selectable="c => !c.deletedAt"
          class="bg-white rounded"
          @input="chartSelected"
        />
        <b-input-group-append>
          <b-button
            v-b-tooltip.hover="{ title: $t('chart.openInBuilder'), container: '#body' }"
            :disabled="!selectedChart || (!selectedChart.canUpdateChart && !selectedChart.canDeleteChart)"
            variant="light"
            class="d-flex align-items-center"
            :to="{ name: 'admin.charts.edit', params: { chartID: (selectedChart || {}).chartID }, query: null }"
          >
            <font-awesome-icon :icon="['fas', 'external-link-alt']" />
          </b-button>
        </b-input-group-append>
      </b-input-group>
    </b-form-group>

    <template v-if="isDrillDownAvailable">
      <b-form-group
        :description="$t('chart.drillDown.description')"
        label-class="d-flex align-items-center text-primary"
        class="mb-1"
      >
        <template #label>
          {{ $t('chart.drillDown.label') }}
          <b-form-checkbox
            v-model="options.drillDown.enabled"
            switch
            class="ml-1"
          />
        </template>

        <vue-select
          v-model="options.drillDown.blockID"
          :options="drillDownOptions"
          :get-option-key="getOptionKey"
          :disabled="!options.drillDown.enabled"
          :get-option-label="o => o.title || o.kind"
          :reduce="option => option.blockID"
          :calculate-position="calculateDropdownPosition"
          :clearable="true"
          :placeholder="$t('chart.drillDown.openInModal')"
          append-to-body
          class="bg-white w-100 rounded"
        />
      </b-form-group>
    </template>
  </b-tab>
</template>
<script>
import base from './base'
import { mapGetters } from 'vuex'
import { NoID } from '@cortezaproject/corteza-js'
import { VueSelect } from 'vue-select'

export default {
  i18nOptions: {
    namespaces: 'block',
  },

  name: 'Chart',

  components: {
    VueSelect,
  },

  extends: base,

  computed: {
    ...mapGetters({
      charts: 'chart/set',
    }),

    selectedChart () {
      if (!this.options.chartID || this.options.chartID === NoID) {
        return
      }

      return this.charts.find(({ chartID }) => chartID === this.options.chartID)
    },

    selectedChartModuleID () {
      if (!this.selectedChart) return

      const { moduleID } = (this.selectedChart.config.reports[0] || {})

      return moduleID
    },

    isDrillDownAvailable () {
      if (!this.selectedChart) return

      const { metrics = [] } = (this.selectedChart.config.reports[0] || {})

      return !metrics.some(({ type }) => type === 'gauge')
    },

    drillDownOptions () {
      return this.blocks.filter(({ blockID, kind, options = {} }) => kind === 'RecordList' && blockID !== NoID && options.moduleID === this.selectedChartModuleID)
    },
  },

  methods: {
    chartSelected () {
      this.options.drillDown = {
        enabled: false,
        blockID: '',
      }
    },

    getOptionKey ({ chartID }) {
      return chartID
    },
  },
}
</script>
