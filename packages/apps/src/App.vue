<template>
  <internal-app>
    <div class="min-h-screen">
      <app-header />
      <div
        v-for="group in appsGroups"
        class="max-w-960 mx-auto mt-24"
      >
        <h2 class="text-solstice-blue-opacity-80">
          {{ group.name }}
        </h2>
        <p class="text-solstice-blue-opacity-80">
          {{ group.description }}
        </p>
        <div class="flex flex-wrap">
          <a
            v-for="app in group.apps"
            :href="`${isProduction && app.isPublic ? 'https://tools.algolia.com' : ''}${app.url}`"
            class="my-24 mr-24"
          >
            <div
              class="rounded border border-proton-grey-opacity-60 bg-white p-24 w-200 text-center text-lg shadow text-nova-grey"
            >
              {{ app.name }}
            </div>
          </a>
        </div>
      </div>
    </div>
  </internal-app>
</template>

<script>
    import InternalApp from "common/components/app/InternalApp";
    import AppHeader from "common/components/header/AppHeader";
    import AppManagement from "common/components/configuration/AppManagement";

    const isProduction = process.env.NODE_ENV === 'production';

    const appsGroups = [
        {
            name: 'Relevance tools',
            description: 'Bootstrap, iterate, and fine tune relevance',
            apps: [
                {name: 'Metaparams', url: '/metaparams', isPublic: true},
                {name: 'Relevance Testing', url: '/relevance-testing', isPublic: true},
                {name: 'Index Differ', url: '/index-differ', isPublic: true},
            ],
        },
        {
            name: 'Management tools',
            description: 'Manage your apps and indices',
            apps: [
                {name: 'Realtime Logs', url: '/logs', isPublic: true},
                {name: 'Analyse Logs', url: 'https://logger.algolia.com/'},
            ]
        },
        {
            name: 'Data tools',
            description: 'Play with data',
            apps: [
                {name: 'Transform', url: '/transform', isPublic: true},
                {name: 'Index analyzer', url: '/index-analyzer', isPublic: true},
                {name: 'Index size', url: '/index-size'},
            ]
        },
        {
            name: 'Info tools',
            description: 'Get information on engine features',
            apps: [
                {name: 'Dictionaries', url: '/dictionaries'},
                {name: 'Infra Watch', url: '/infra-watch'},
            ]
        },
        {
            name: 'Feature Testing tools',
            description: 'Test features without code',
            apps: [
                {name: 'Insights UI', url: '/insights-ui', isPublic: true},
            ]
        },
    ];

    export default {
        name: 'Home',
        components: {InternalApp, AppHeader, AppManagement},
        data: function () {
            return {
                appsGroups: appsGroups,
                isProduction: isProduction,
            };
        },
    }
</script>
