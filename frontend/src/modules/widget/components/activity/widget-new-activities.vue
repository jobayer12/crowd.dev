<template>
  <query-renderer
    v-if="cubejsApi"
    :cubejs-api="cubejsApi"
    :query="query"
  >
    <template #default="{ resultSet, loading, error }">
      <div class="bg-white px-6 py-5 rounded-lg shadow">
        <!-- Widget Header -->
        <div
          class="flex grow justify-between items-center pb-5 border-b border-gray-100"
          :class="{ 'mb-8': !loading && !error }"
        >
          <div class="flex gap-1">
            <app-widget-granularity
              :template="ACTIVITIES_REPORT.nameAsId"
              :widget="NEW_ACTIVITIES_WIDGET.name"
              :granularity="granularity"
              @on-update="
                (updatedGranularity) =>
                  (granularity = updatedGranularity)
              "
            />
            <app-widget-title
              :title="NEW_ACTIVITIES_WIDGET.name"
            />
          </div>
          <app-widget-period
            :template="ACTIVITIES_REPORT.nameAsId"
            :widget="NEW_ACTIVITIES_WIDGET.name"
            :period="period"
            :granularity="granularity"
            module="reports"
            @on-update="
              (updatedPeriod) => (period = updatedPeriod)
            "
          />
        </div>

        <!-- Loading -->
        <app-widget-loading v-if="loading" />

        <!-- Error -->
        <app-widget-error v-else-if="error" />

        <!-- Widget Chart -->
        <app-widget-area
          v-else
          :datasets="datasets"
          :result-set="resultSet"
          :chart-options="widgetChartOptions"
          :granularity="granularity.value"
        />
      </div>
    </template>
  </query-renderer>
</template>

<script setup>import { computed, ref } from 'vue';
import { QueryRenderer } from '@cubejs-client/vue3';
import { DAILY_GRANULARITY_FILTER, SEVEN_DAYS_PERIOD_FILTER } from '@/modules/widget/widget-constants';
import { chartOptions } from '@/modules/report/templates/template-chart-config';
import { parseAxisLabel } from '@/utils/reports';
import { mapGetters } from '@/shared/vuex/vuex.helpers';
import { ACTIVITIES_QUERY } from '@/modules/widget/widget-queries';
import AppWidgetTitle from '@/modules/widget/components/shared/widget-title.vue';
import AppWidgetPeriod from '@/modules/widget/components/shared/widget-period.vue';
import AppWidgetGranularity from '@/modules/widget/components/shared/widget-granularity.vue';
import AppWidgetArea from '@/modules/widget/components/shared/widget-area.vue';
import AppWidgetLoading from '@/modules/widget/components/shared/widget-loading.vue';
import AppWidgetError from '@/modules/widget/components/shared/widget-error.vue';
import ACTIVITIES_REPORT, { NEW_ACTIVITIES_WIDGET } from '@/modules/report/templates/config/activities';

const props = defineProps({
  filters: {
    type: Object,
    default: null,
  },
});

const period = ref(SEVEN_DAYS_PERIOD_FILTER);
const granularity = ref(DAILY_GRANULARITY_FILTER);

const widgetChartOptions = chartOptions('area', {
  xTicksCallback: (
    value,
  ) => parseAxisLabel(value, granularity.value.value),
});

const { cubejsApi } = mapGetters('widget');

const datasets = computed(() => [
  {
    name: NEW_ACTIVITIES_WIDGET.name,
    borderColor: '#E94F2E',
    measure: 'Activities.count',
    granularity: granularity.value.value,
    tooltipBtn: false,
    showLegend: false,
  },
]);

const query = computed(() => ACTIVITIES_QUERY({
  period: period.value,
  granularity: granularity.value,
  selectedPlatforms: props.filters.platform.value,
  selectedHasTeamActivities: props.filters.teamActivities,
}));
</script>

<script>
export default {
  name: 'AppWidgetNewActivities',
};
</script>
