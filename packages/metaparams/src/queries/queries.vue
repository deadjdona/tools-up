<template>
  <div>
    <div class="flex p-8 border-b border-proton-grey bg-proton-grey-opacity-40 text-telluric-blue text-xs uppercase tracking-wide">
      <div>
        Query management
      </div>
      <div
        class="ml-auto w-8 h-8 leading-none flex items-center justify-center cursor-pointer"
        @click="open = !open"
      >
        <chevron-down-icon
          :class="`flex-grow-0 w-8 h-8 flex-shrink-0 block fill-current ${open ? '' : 'rotate-180'}`"
        />
      </div>
    </div>
    <div
      v-if="open"
      class="border-b border-proton-grey"
    >
      <div>
        <h4 class="p-16">
          Last Query
        </h4>
        <pre
          :class="`px-16 py-4 cursor-pointer ${currentQuery === panelQuery ? 'bg-proton-grey-opacity-50' : ''}`"
          @click="goToQuery(currentQuery)"
        >"{{ currentQuery }}"</pre>
      </div>
      <div>
        <h4 class="p-16">
          Top Queries (last 7 days)
        </h4>
        <div class="max-h-300 overflow-y-scroll pb-24">
          <div
            v-for="(query, index) in topQueries"
            :key="index"
            class="px-16 py-4 cursor-pointer"
            :class="`${query === panelQuery ? 'bg-proton-grey-opacity-50' : ''}`"
            @click="goToQuery(query)"
          >
            <span>{{ index + 1 }}.</span>&nbsp;<span>{{ query }}</span>
          </div>
          <div
            v-if="topQueries.length <= 0"
            class="ml-16 italic"
          >
            No top queries yet
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
    import ChevronDownIcon from "common/icons/chevron-down.svg";
    import panelsMixin from "common/mixins/panelsMixin";

    export default {
        name: 'Queries',
        components: {ChevronDownIcon},
        mixins: [panelsMixin],
        props: ['panelKey'],
        data: function () {
            return {
                open: false,
                panel: 'leftPanel',
                lastQuery: null,
                lastQueryJustSaved: false,
                topQueries: [],
            }
        },
        computed: {
            region: function () {
                return this.$store.state.apps[this.panelAppId].__log_region || 'us';
            },
            currentQuery: function () {
                return this.lastQuery !== null ? this.lastQuery : this.panelQuery;
            },
            panelQuery: {
                get() {
                    return this.$store.state.panels.query;
                },
                set(value) {
                    this.$store.commit("panels/setQuery", value);
                }
            },
        },
        watch: {
            panelAppId: function () { this.fetchTopQueries(); },
            panelIndexName: function () { this.fetchTopQueries(); },
            panelQuery: function (val, oldVal) {
                if (this.lastQueryJustSaved) {
                    this.lastQueryJustSaved = false;

                    if (this.lastQuery === null) this.lastQuery = oldVal;
                } else {
                    this.lastQuery = val;
                }
            }
        },
        mounted: function () {
            this.fetchTopQueries();
        },
        methods: {
            fetchTopQueries: async function () {
                this.topQueries = [];

                try {
                    const topQueriesQuery = await fetch(`https://analytics.${this.region}.algolia.com/2/searches?index=${this.panelIndexName}&limit=100`, {
                        headers: {
                            'X-Algolia-Application-Id': this.panelAppId,
                            'X-Algolia-API-Key': this.panelAdminAPIKey,
                        }
                    });

                    if (topQueriesQuery.status !== 200) return;

                    const topQueries = await topQueriesQuery.json();

                    if (topQueries.searches === null) return;

                    this.topQueries = topQueries.searches.filter((query) => {
                        return query.search.length > 0;
                    }).map((query) => {
                        return query.search;
                    });
                } catch (e) {
                    console.warn('fetchTopQueries ignored due to apiKey restrictions', e)
                }
            },
            goToQuery: function (query) {
                this.lastQueryJustSaved = true;
                this.panelQuery = query;
            }
        }
    }
</script>
