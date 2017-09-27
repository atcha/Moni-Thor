<template>
  <div class="list">
    <md-whiteframe class="md-theme-default md-toolbar" style="height: 50px;">
            <span>
                <h2>Services</h2>
            </span>
    </md-whiteframe>
    <md-whiteframe>
      <md-layout md-gutter>
        <md-layout md-flex-xsmall="100" md-flex-large="20" md-flex-xlarge="20" md-row>
          <md-input-container>
            <label>Filter</label>
            <md-input v-model="filter"></md-input>
            {{filteredServices.length}} results
          </md-input-container>
        </md-layout>
      </md-layout>
    </md-whiteframe>
    <md-whiteframe>
      <div>
        <md-progress md-indeterminate class="md-accent" v-if="loading"></md-progress>
      </div>
      <md-list v-if="!loading">
        <md-layout md-gutter style="background-color: #f7f7f7;">
          <md-layout md-gutter v-for="service,idx in filteredServices" style="padding-bottom: 15px;">
            <md-card>
              <md-card-header class=" md-theme-default md-toolbar">
                <div class="md-title">{{service.serviceName}}</div>
                <md-button class="md-fab md-mini md-fab-bottom-right md-warn"
                           v-on:click="pingService(service)"
                           style="margin: -7px;">
                  <md-icon>refresh</md-icon>
                </md-button>
              </md-card-header>
              <md-card-content>
                <md-list>
                  <md-list-item v-for="server in serverWrapper.servers">
                    <label>
                      {{server}}
                    </label>
                    <span style="flex: 1 1 0%;"></span>
                    <md-button class="md-raised md-accent"
                               v-if="service.responding && service.info.build"
                    >{{service.info.build.version}}
                      <md-tooltip md-direction="left">
                        {{service.info.build | json}}
                      </md-tooltip>
                    </md-button>
                    <span v-if="service.responding && service.responseTime">{{service.responseTime}} ms</span>
                    <span>
                        <span>
                          <md-icon v-if="!service.pending && service.responding === true" class="green">graphic_eq</md-icon>
                          <md-icon v-if="!service.pending && service.responding === false"
                                   class="red">portable_wifi_off</md-icon>
                          <md-spinner :md-size="20" v-if="service.pending" md-indeterminate></md-spinner>
                        </span>
                    </span>
                  </md-list-item>
                </md-list>
              </md-card-content>
            </md-card>
          </md-layout>
        </md-layout>
      </md-list>
    </md-whiteframe>
  </div>
</template>

<script>
  import axios from 'axios';
  import Vue from 'vue'
  import MdListItem from "../../node_modules/vue-material/src/components/mdList/mdListItemButton.vue";

  export default {
    components: {
      MdListItem
    },
    name: 'list',
    data() {
      return {
        services: null,
        filterResponding: null,
        filter: '',
        server: 'http://intranettestinfo7/',
        serverWrapper: {
          servers: []
        },
        loading: false
      }
    },
    computed: {
      filteredServices() {
        if (this.services) {
          return this.services.filter((service) => {
            return ((this.filterResponding === null) || service.responding === this.filterResponding)
              && ((!this.filter) || this.getUrl(service).includes(this.filter))
          });
        }
        return [];
      }
    },
    methods: {
      getUrl(service) {
        return (service.serverHost || '') + (service.serviceName || '') + (service.serviceToCall || '');
      },
      clearFilters() {
        this.filterResponding = null;
        this.filter = '';
      },
      pingService(service){

      }
    },
    mounted() {
      axios.get(`/api/services`)
        .then((response) => {
          this.services = response.data || [];
        })
        .catch((error) => console.error(error));
      axios.get(`/api/servers`)
        .then((response) => {
          this.serverWrapper = response.data || {
            servers: []
          };
        })
        .catch((error) => console.error(error));

    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }

  .green {
    color: #00cb4b;
  }

  .red {
    color: #cb2832;
  }

  li.md-list-item.odd {
    background-color: #f7f7f7;
  }

  li.md-list-item {
    margin: 0px;
  }
</style>