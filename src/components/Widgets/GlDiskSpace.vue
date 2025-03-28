<template>
    <div class="glances-disk-space-wrapper" v-if="disks">
      <div v-for="(disk, key) in disks" :key="key" class="disk-row">
        <PercentageChart
          :title="disk.device_name"
          :values="[
            { label: $t('widgets.glances.disk-space-used'), size: disk.percent, color: '#f80363' },
            { label: $t('widgets.glances.disk-space-free'), size: 100 - disk.percent, color: '#20e253' }
          ]"
        />
        <p class="info">
          <b>{{ $t('widgets.glances.disk-space-free') }}</b>:
          {{ disk.size - disk.used | formatSize }} out of {{ disk.size | formatSize }}
        </p>
        <p class="info">
          <b>{{ $t('widgets.glances.disk-mount-point') }}</b>: {{ disk.mnt_point }}
        </p>
        <p class="info">
          <b>{{ $t('widgets.glances.disk-file-system') }}</b>: {{ disk.fs_type }}
        </p>
      </div>
    </div>
  </template>
  
  <script>
  import WidgetMixin from '@/mixins/WidgetMixin';
  import GlancesMixin from '@/mixins/GlancesMixin';
  import PercentageChart from '@/components/Charts/PercentageChart';
  import { getValueFromCss, convertBytes } from '@/utils/MiscHelpers';
  
  export default {
    mixins: [WidgetMixin, GlancesMixin],
    components: {
      PercentageChart,
    },
    data() {
      return {
        disks: null,
      };
    },
    computed: {
      endpoint() {
        return this.makeGlancesUrl('fs');
      },
    },
    filters: {
      formatSize(byteValue) {
        return convertBytes(byteValue);
      },
    },
    methods: {
      processData(diskData) {
        // Falls eine Disk-Konfiguration in den Optionen vorhanden ist, filtern und benennen wir die Disks um.
        if (this.options.Disks && Array.isArray(this.options.Disks)) {
          let diskMapping = {};
          this.options.Disks.forEach(item => {
            const key = Object.keys(item)[0];
            diskMapping[key] = item[key];
          });
          this.disks = diskData.filter(disk => {
            if (diskMapping[disk.device_name] !== undefined) {
              // Ersetze den Standardnamen durch den konfigurierten Anzeigenamen
              disk.device_name = diskMapping[disk.device_name];
              return true;
            }
            return false;
          });
        } else {
          this.disks = diskData;
        }
      },
    },
    mounted() {
      this.background = getValueFromCss('widget-accent-color');
    },
  };
  </script>
  
  <style scoped lang="scss">
  .glances-disk-space-wrapper {
    color: var(--widget-text-color);
    .disk-row {
      padding: 0.25rem 0 0.5rem 0;
      &:not(:last-child) {
        border-bottom: 1px dashed var(--widget-text-color);
      }
      p.info {
        font-size: 0.8rem;
        margin: 0.25rem 0;
        color: var(--widget-text-color);
        opacity: var(--dimming-factor);
        font-family: var(--font-monospace);
      }
    }
  }
  </style>
  
