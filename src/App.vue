<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title>
          <strong>GitHub Repos Searcher 🔍</strong>
        </q-toolbar-title>
        <q-space></q-space>
        <q-btn
          label="github-searcher-js source code"
          icon-right="code"
          no-caps
          flat
          color="#ffffff"
          @click="openInNewTab('https://gitlab.com/viniarck/github-searcher-js')"
        />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <Search :baseEndpoint="baseUrl" />
    </q-page-container>
  </q-layout>
</template>

<script>
import Search from "./components/Search.vue";

export default {
  name: "LayoutDefault",

  data: () => {
    return {
      baseUrl: process.env.VUE_APP_BE_BASE_URL,
    };
  },
  methods: {
    openInNewTab(url) {
      if (url !== undefined) {
        var win = window.open(url, "_blank");
        win.focus();
      }
    },
  },
  components: {
    Search,
  },
};
</script>

<style></style>
