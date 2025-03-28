<template>
    <div class="glances-network-interfaces-wrapper" v-if="networks">
      <div class="interface-row" v-for="network in networks" :key="network.name">
        <div class="network-info">
          <p class="network-name">{{ network.name }}</p>
          <p class="network-speed">{{ network.speed | formatSpeed }}</p>
          <p :class="`network-online ${network.online}`">
            {{ network.online }}
          </p>
        </div>
        <div class="current" v-if="network.online === 'up'">
          <span class="upload">
            ↑ <span class="val">{{ network.currentUpload | formatDataSize }}</span>
          </span>
          <span class="separator">|</span>
          <span class="download">
            ↓ <span class="val">{{ network.currentDownload | formatDataSize }}</span>
          </span>
        </div>
        <div class="total">
          <b class="lbl">Total</b> Up
          <span class="val">{{ network.totalUpload | formatDataSize }}</span>
          <span class="separator">|</span>
          Down
          <span class="val">{{ network.totalDownload | formatDataSize }}</span>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import WidgetMixin from '@/mixins/WidgetMixin';
  import GlancesMixin from '@/mixins/GlancesMixin';
  import { convertBytes } from '@/utils/MiscHelpers';
  
  export default {
    mixins: [WidgetMixin, GlancesMixin],
    data() {
      return {
        networks: null,
        previous: null,
      };
    },
    computed: {
      endpoint() {
        return this.makeGlancesUrl('network');
      },
    },
    filters: {
      formatDataSize(data) {
        return convertBytes(data);
      },
      formatSpeed(byteValue) {
        if (!byteValue) return '';
        return `${convertBytes(byteValue)}/s`;
      },
      getArrow(direction) {
        if (direction === 'up') return '↑';
        if (direction === 'down') return '↓';
        return '';
      },
    },
    methods: {
      processData(networkData) {
        // Speichere vorherige Netzwerkinformationen (optional, je nach Bedarf)
        this.previous = this.networks;
        const networks = [];
  
        // Falls eine Interfaces-Konfiguration vorhanden ist, erstellen wir ein Mapping:
        if (this.options.Interfaces && Array.isArray(this.options.Interfaces)) {
          let interfaceMapping = {};
          this.options.Interfaces.forEach(item => {
            const key = Object.keys(item)[0];
            interfaceMapping[key] = item[key];
          });
          // Nur die Interfaces übernehmen, die in der Konfiguration angegeben wurden
          networkData.forEach((network, index) => {
            if (interfaceMapping[network.interface_name] !== undefined) {
              networks.push({
                name: interfaceMapping[network.interface_name],
                speed: network.speed,
                online: network.speed ? 'up' : 'down', // v3 zu v4: "is_up" ist nicht mehr Standard
                currentDownload: network.bytes_recv,
                currentUpload: network.bytes_sent,
                totalDownload: network.bytes_recv_gauge,
                totalUpload: network.bytes_sent_gauge,
                changeDownload: this.previous && network.rx > this.previous[index].rx,
                changeUpload: this.previous && network.tx > this.previous[index].tx,
              });
            }
          });
        } else {
          // Falls keine Konfiguration vorliegt, werden alle Interfaces übernommen
          networkData.forEach((network, index) => {
            networks.push({
              name: network.interface_name,
              speed: network.speed,
              online: network.speed ? 'up' : 'down',
              currentDownload: network.bytes_recv,
              currentUpload: network.bytes_sent,
              totalDownload: network.bytes_recv_gauge,
              totalUpload: network.bytes_sent_gauge,
              changeDownload: this.previous && network.rx > this.previous[index].rx,
              changeUpload: this.previous && network.tx > this.previous[index].tx,
            });
          });
        }
        this.networks = networks;
      },
    },
    created() {
      this.overrideUpdateInterval = 5;
    },
  };
  </script>
  
  <style scoped lang="scss">
  .glances-network-interfaces-wrapper {
    color: var(--widget-text-color);
    .interface-row {
      display: flex;
      flex-direction: column;
      padding: 0.5rem 0;
      .network-info {
        display: flex;
        justify-content: space-between;
        .network-name {
          width: 50%;
          margin: 0.5rem 0;
          overflow: hidden;
          font-weight: bold;
          white-space: nowrap;
          text-overflow: ellipsis;
        }
        .network-speed {
          font-family: var(--font-monospace);
          margin: 0.5rem 0;
        }
        .network-online {
          min-width: 15%;
          margin: 0.5rem 0;
          font-weight: bold;
          text-align: right;
          text-transform: capitalize;
          &.up { color: var(--success); }
          &.down { color: var(--danger); }
        }
      }
      .total, .current {
        display: inline;
        margin: 0.25rem 0;
        b.lbl {
          margin-right: 0.25rem;
        }
        span.val {
          margin-left: 0.25rem;
          font-family: var(--font-monospace);
        }
        span.separator {
          font-weight: bold;
          margin: 0 0.5rem;
        }
        &.total {
          opacity: var(--dimming-factor);
          font-size: 0.85rem;
        }
        &.current {
          text-align: center;
          background: var(--widget-accent-color);
          border-radius: var(--curve-factor);
          padding: 0.2rem 0.5rem;
        }
      }
      &:not(:last-child) {
        border-bottom: 1px dashed var(--widget-text-color);
      }
    }
  }
  </style>
  
