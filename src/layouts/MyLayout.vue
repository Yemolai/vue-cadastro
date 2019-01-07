<template>
  <q-layout view="lHh Lpr lFf">
    <q-layout-header>
      <q-toolbar
        color="primary"
        :glossy="$q.theme === 'mat'"
        :inverted="$q.theme === 'ios'"
      >
        <q-btn
          flat
          dense
          round
          @click="leftDrawerOpen = !leftDrawerOpen"
          aria-label="Menu"
        >
          <q-icon name="menu" />
        </q-btn>

        <q-toolbar-title>
          Cadastro de crianças
          <div slot="subtitle">Feito com Quasar v{{ $q.version }}</div>
        </q-toolbar-title>
      </q-toolbar>
    </q-layout-header>

    <q-layout-drawer
      v-model="leftDrawerOpen"
      :content-class="$q.theme === 'mat' ? 'bg-grey-2' : null"
    >
      <q-list
        no-border
        link
        inset-delimiter
      >
        <q-list-header>Seções</q-list-header>
        <q-item @click.native="navigateTo('/lista')">
          <q-item-side icon="reorder" />
          <q-item-main label="Lista" sublabel="Exibir os registros cadastrados" />
        </q-item>
      </q-list>
    </q-layout-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { openURL } from 'quasar'

export default {
  name: 'MyLayout',
  beforeMount: function () {
    const drawerState = localStorage.getItem('drawerState')
    if (drawerState) {
      this.leftDrawerOpen = JSON.parse(drawerState) === true
    }
  },
  watch: {
    leftDrawerOpen: function (newVal) {
      localStorage.setItem('drawerState', newVal)
    }
  },
  data () {
    return {
      leftDrawerOpen: false
    }
  },
  methods: {
    openURL,
    navigateTo: function (route) {
      this.$router.push(route)
    }
  }
}
</script>

<style>
</style>
