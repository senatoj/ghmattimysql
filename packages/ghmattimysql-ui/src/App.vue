<template>
  <main
    v-if="showInterface"
    class="flex h-screen bg-opacity-50 bg-black items-center justify-center"
  >
    <div class="w-full bg-white shadow-2xl rounded-md max-w-screen-xl">
      <m-app-bar>
        <m-icon class="mr-2">storage</m-icon>
        <span>MySQL Explorer</span>
        <div class="flex-grow" />
        <m-icon
          @click.native="close()"
          class="
            cursor-pointer rounded-full hover:bg-white hover:bg-opacity-10
            transition duration-300 ease-in-out
          "
        >
          close
        </m-icon>
      </m-app-bar>
      <m-tabs>
        <m-tab-item label="Time-resolved" icon="timeline">
          <div class="p-2" style="height: 480px;">
            <m-chart
              id="time-graph"
              :labels="timeLabels"
              :datasets="timeData"
              :height="540"
            ></m-chart>
          </div>
        </m-tab-item>
        <m-tab-item label="Resources" icon="apps">
          <div class="p-2" style="height: 480px;">
            <m-chart
              id="resource-graph"
              :labels="resourceLabels"
              :datasets="resourceData"
              :height="540"
            ></m-chart>
          </div>
        </m-tab-item>
        <m-tab-item label="Slowest Queries" icon="slow_motion_video">
          <div class="p-2" style="height: 480px;">
            <m-data-table
              :headers="headers"
              :items="slowqueries"
              :items-per-page="7"
            >
              <template v-slot:row="props">
                <td class="text-center h-12">{{ props.item.resource }}</td>
                <td class="text-left h-12">{{ props.item.sql }}</td>
                <td class="text-center h-12">{{ props.item.queryTime }}ms</td>
              </template>
            </m-data-table>
          </div>
        </m-tab-item>
        <m-tab-item label="Server Status" icon="settings_applications">
          <div class="p-2" style="height: 480px;">
            <m-server-status />
          </div>
        </m-tab-item>
      </m-tabs>
      <div class="h-8 bg-black w-full rounded-b-md" />
    </div>
  </main>
</template>

<script>
import MAppBar from './components/MAppBar.vue';
import MChart from './components/MChart.vue';
import MDataTable from './components/MDataTable.vue';
import MIcon from './components/MIcon.vue';
import MServerStatus from './components/MServerStatus.vue';
import MTabs from './components/MTabs.vue';
import MTabItem from './components/MTabItem.vue';

export default {
  components: {
    MAppBar,
    MChart,
    MDataTable,
    MIcon,
    MServerStatus,
    MTabs,
    MTabItem,
  },
  data() {
    return {
      showInterface: false,
      colorGraphLoad: {
        backgroundColor: [
          'rgba(54, 73, 93, 0.5)',
        ],
        borderColor: [
          '#36495d',
        ],
        borderWidth: 3,
      },
      colorGraphAvg: {
        backgroundColor: [
          'rgba(71, 183, 132, 0.5)',
        ],
        borderColor: [
          '#47b784',
        ],
        borderWidth: 3,
      },
      colorGraphCount: {
        backgroundColor: [
          'rgba(62, 128, 113, 0.5)',
        ],
        borderColor: [
          '#3e8071',
        ],
        borderWidth: 3,
      },
      resourceData: [],
      resourceLabels: [],
      timeLabels: [],
      timeData: [],
      slowqueries: [
        {
          resource: 'memes',
          sql: 'SELECT * FROM memes',
          queryTime: 5000,
        },
      ],
      headers: [
        {
          text: 'Resource',
          value: 'resource',
        },
        {
          text: 'Query',
          value: 'sql',
          sortable: false,
          align: 'left',
        },
        {
          text: 'Execution Time (ms)',
          value: 'queryTime',
        },
      ],
    };
  },
  destroyed() {
    window.removeEventListener('message', this.listener);
  },
  methods: {
    close() {
      fetch('http://ghmattimysql/close-explorer', {
        method: 'post',
        body: JSON.stringify({ close: true }),
      });
    },
    onToggleShow() {
      this.showInterface = !this.showInterface;
    },
    onSlowQueryData({ slowQueries }) {
      if (Array.isArray(slowQueries)) {
        this.slowqueries = slowQueries;
      }
    },
    onTimeData({ timeData }) {
      if (Array.isArray(timeData) && timeData.length === 3) {
        this.timeData = [
          { ...this.colorGraphLoad, label: 'Server Load (ms)', ...timeData[0] },
          { ...this.colorGraphAvg, label: 'Average Query Time (ms)', ...timeData[1] },
          { ...this.colorGraphCount, label: 'Query Count', ...timeData[2] },
        ];
        const labels = [];
        for (let i = timeData[0].data.length - 1; i > -1; i -= 1) {
          if (i !== 0) {
            labels.push(`-${i * 5}min`);
          } else {
            labels.push('now');
          }
        }
        this.timeLabels = labels;
      }
    },
    onResourceData({ resourceData }) {
      if (Array.isArray(resourceData) && resourceData.length === 3) {
        this.resourceData = [
          { ...this.colorGraphLoad, label: 'Server Load (ms)', ...resourceData[0] },
          { ...this.colorGraphAvg, label: 'Average Query Time (ms)', ...resourceData[1] },
          { ...this.colorGraphCount, label: 'Query Count', ...resourceData[2] },
        ];
      }
    },
    onResourceLabels({ resourceLabels }) {
      this.resourceLabels = resourceLabels;
    },
  },
  mounted() {
    this.listener = window.addEventListener('message', (event) => {
      const item = event.data || event.detail;
      if (item && this[item.type]) this[item.type](item);
    });
    // For editting in browser
    // setTimeout(this.onToggleShow, 1000);
  },
  name: 'app',
};
</script>

<style lang="scss">
@import './styles/tailwind.css';
@import './styles/mixins';

@include font-face('Alegreya Sans', './assets/fonts/alegreya-sans-v9-latin-300',
  300, normal, woff woff2);
@include font-face('Alegreya Sans', './assets/fonts/alegreya-sans-v9-latin-regular',
  400, normal, woff woff2);
@include font-face('Alegreya Sans', './assets/fonts/alegreya-sans-v9-latin-500',
  500, normal, woff woff2);
@include font-face('Alegreya Sans', './assets/fonts/alegreya-sans-v9-latin-700',
  700, normal, woff woff2);
</style>
