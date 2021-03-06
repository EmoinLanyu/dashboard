<!--
Copyright (c) 2020 by SAP SE or an SAP affiliate company. All rights reserved. This file is licensed under the Apache Software License, v. 2 except as noted otherwise in the LICENSE file

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
 -->

<template>
  <v-container fluid class="shootlist">
    <v-card class="mr-extra">
      <v-toolbar flat height="72" color="cyan darken-2">
        <img src="../assets/certified_kubernetes_white.svg" height="60" class="ml-1 mr-3">
        <v-toolbar-title class="white--text">
          <div class="headline">Kubernetes Clusters</div>
          <div class="subtitle-1">{{headlineSubtitle}}</div>
        </v-toolbar-title>
        <v-spacer></v-spacer>
        <v-text-field v-if="search || items.length > 3"
          prepend-inner-icon="search"
          color="cyan darken-2"
          label="Search"
          clearable
          hide-details
          flat
          solo
          v-model="search"
          @keyup.esc="search=''"
          class="search_textfield"
        ></v-text-field>
        <v-menu :nudge-bottom="20" :nudge-right="20" left v-model="tableMenu" absolute>
          <template v-slot:activator="{ on: menu }">
            <v-tooltip open-delay="500" top>
              <template v-slot:activator="{ on: tooltip }">
                <v-btn v-on="{ ...menu, ...tooltip}" icon>
                  <v-icon class="cursor-pointer" color="white">more_vert</v-icon>
                </v-btn>
              </template>
              Table Options
            </v-tooltip>
          </template>
          <v-list subheader dense>
            <v-subheader>Column Selection</v-subheader>
            <v-list-item v-for="item in headers" :key="item.text" @click.stop="setColumnChecked(item)">
              <v-list-item-action>
                <v-icon :color="checkboxColor(item.checked)" v-text="checkboxIcon(item.checked)"/>
              </v-list-item-action>
              <v-list-item-content class="grey--text text--darken-2">
                <v-list-item-title>{{ item.text }}</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <v-list-item>
              <v-list-item-content>
                <v-tooltip top style="width: 100%">
                  <template v-slot:activator="{ on }">
                    <v-btn v-on="on" block text class="text-center cyan--text text--darken-2" @click.stop="resetColumnsChecked">
                      Reset
                    </v-btn>
                  </template>
                  <span>Reset to Defaults</span>
                </v-tooltip>
              </v-list-item-content>
            </v-list-item>
          </v-list>
          <v-list subheader dense v-if="!projectScope">
            <v-subheader>Filter Table</v-subheader>
            <v-list-item @click.stop="showOnlyShootsWithIssues = !showOnlyShootsWithIssues">
              <v-list-item-action>
                <v-icon :color="checkboxColor(showOnlyShootsWithIssues)" v-text="checkboxIcon(showOnlyShootsWithIssues)"/>
              </v-list-item-action>
              <v-list-item-content class="grey--text text--darken-2">
                <v-list-item-title>Show only clusters with issues</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <template v-if="isAdmin">
              <v-list-item
                @click.stop="toggleFilter('progressing')"
                :disabled="filtersDisabled"
                :class="disabledFilterClass">
                <v-list-item-action>
                  <v-icon :color="checkboxColor(isFilterActive('progressing'))" v-text="checkboxIcon(isFilterActive('progressing'))"/>
                </v-list-item-action>
                <v-list-item-content class="grey--text text--darken-2">
                  <v-list-item-title>Hide progressing clusters</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
              <v-list-item
                @click.stop="toggleFilter('userIssues')"
                :disabled="filtersDisabled"
                :class="disabledFilterClass">
                <v-list-item-action>
                  <v-icon :color="checkboxColor(isFilterActive('userIssues'))" v-text="checkboxIcon(isFilterActive('userIssues'))"/>
                </v-list-item-action>
                <v-list-item-content class="grey--text text--darken-2">
                  <v-list-item-title>Hide user issues</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
              <v-list-item
                @click.stop="toggleFilter('deactivatedReconciliation')"
                :disabled="filtersDisabled"
                :class="disabledFilterClass">
                <v-list-item-action>
                  <v-icon :color="checkboxColor(isFilterActive('deactivatedReconciliation'))" v-text="checkboxIcon(isFilterActive('deactivatedReconciliation'))"/>
                </v-list-item-action>
                <v-list-item-content class="grey--text text--darken-2">
                  <v-list-item-title>Hide clusters with deactivated reconciliation</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
              <v-list-item
                @click.stop="toggleFilter('hasJournals')"
                :disabled="filtersDisabled"
                :class="disabledFilterClass"
                v-if="!!gitHubRepoUrl">
                <v-list-item-action>
                  <v-icon :color="checkboxColor(isFilterActive('hasJournals'))" v-text="checkboxIcon(isFilterActive('hasJournals'))"/>
                </v-list-item-action>
                <v-list-item-content class="grey--text text--darken-2">
                  <v-list-item-title>Hide clusters with journal entries</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
            </template>
          </v-list>
        </v-menu>
      </v-toolbar>
      <v-data-table
        class="shootListTable"
        :headers="visibleHeaders"
        :items="items"
        :options.sync="options"
        must-sort
        :loading="shootsLoading"
        :footer-props="{ 'items-per-page-options': [5,10,20] }"
      >
        <template v-slot:item="{ item }">
          <shoot-list-row
            :shootItem="item"
            :visibleHeaders="visibleHeaders"
            @showDialog="showDialog"
            :key="item.metadata.uid"
          ></shoot-list-row>
        </template>
      </v-data-table>

      <v-dialog v-model="clusterAccessDialog" max-width="600">
        <v-card>
          <v-card-title class="teal darken-1 grey--text text--lighten-4">
            <div class="headline">Cluster Access <code class="cluster_name">{{currentName}}</code></div>
            <v-spacer></v-spacer>
            <v-btn icon class="grey--text text--lighten-4" @click.native="hideDialog">
              <v-icon>close</v-icon>
            </v-btn>
          </v-card-title>
          <shoot-access-card ref="clusterAccess" :shootItem="selectedItem"></shoot-access-card>
        </v-card>
      </v-dialog>
    </v-card>
    <v-fab-transition v-if="canCreateShoots">
      <v-btn v-if="projectScope" class="cyan darken-2" dark fab fixed bottom right v-show="floatingButton" :to="{ name: 'NewShoot', params: {  namespace } }">
        <v-icon dark ref="add">add</v-icon>
      </v-btn>
    </v-fab-transition>
  </v-container>
</template>

<script>
import { mapGetters, mapActions, mapState } from 'vuex'
import zipObject from 'lodash/zipObject'
import map from 'lodash/map'
import get from 'lodash/get'
import pick from 'lodash/pick'
import join from 'lodash/join'
import ShootListRow from '@/components/ShootListRow'
import ShootAccessCard from '@/components/ShootDetails/ShootAccessCard'

export default {
  name: 'shoot-list',
  components: {
    ShootListRow,
    ShootAccessCard
  },
  data () {
    return {
      floatingButton: false,
      search: '',
      allHeaders: [
        { text: 'PROJECT', value: 'project', align: 'left', checked: false, defaultChecked: true, hidden: false },
        { text: 'NAME', value: 'name', align: 'left', checked: false, defaultChecked: true, hidden: false },
        { text: 'INFRASTRUCTURE', value: 'infrastructure', align: 'left', checked: false, defaultChecked: true, hidden: false },
        { text: 'SEED', value: 'seed', align: 'left', checked: false, defaultChecked: false, hidden: false },
        { text: 'TECHNICAL ID', value: 'technicalId', align: 'left', checked: false, defaultChecked: false, hidden: false, adminOnly: true },
        { text: 'CREATED BY', value: 'createdBy', align: 'left', checked: false, defaultChecked: false, hidden: false },
        { text: 'CREATED AT', value: 'createdAt', align: 'left', checked: false, defaultChecked: false, hidden: false },
        { text: 'PURPOSE', value: 'purpose', align: 'center', checked: false, defaultChecked: true, hidden: false },
        { text: 'STATUS', value: 'lastOperation', align: 'left', checked: false, defaultChecked: true, hidden: false },
        { text: 'VERSION', value: 'k8sVersion', align: 'center', checked: false, defaultChecked: true, hidden: false },
        { text: 'READINESS', value: 'readiness', sortable: true, align: 'center', checked: false, defaultChecked: true, hidden: false },
        { text: 'JOURNAL', value: 'journal', sortable: false, align: 'left', checked: false, defaultChecked: false, hidden: false, adminOnly: true },
        { text: 'JOURNAL LABELS', value: 'journalLabels', sortable: false, align: 'left', checked: false, defaultChecked: true, hidden: false, adminOnly: true },
        { text: 'ACTIONS', value: 'actions', sortable: false, align: 'right', checked: false, defaultChecked: true, hidden: false }
      ],
      dialog: null,
      tableMenu: false,
      options: this.$localStorage.getObject('dataTable_options') || { itemsPerPage: 10 },
      cachedItems: null,
      clearSelectedShootTimerID: undefined
    }
  },
  watch: {
    options (value) {
      if (value) {
        this.$localStorage.setObject('dataTable_options', pick(value, ['sortBy', 'sortDesc', 'itemsPerPage']))
        this.setShootListSortParams(value)
      }
    },
    search (value) {
      this.setShootListSearchValue(value)
    },
    canGetSecrets () {
      this.setColumnVisibility()
    },
    canDeleteShoots () {
      this.setColumnVisibility()
    },
    projectScope () {
      this.setColumnVisibility()
    }
  },
  methods: {
    ...mapActions({
      setSelectedShootInternal: 'setSelectedShoot',
      setShootListSortParams: 'setShootListSortParams',
      setShootListSearchValue: 'setShootListSearchValue',
      setOnlyShootsWithIssues: 'setOnlyShootsWithIssues',
      setShootListFilters: 'setShootListFilters',
      setShootListFilter: 'setShootListFilter'
    }),
    async showDialog (args) {
      switch (args.action) {
        case 'access':
          try {
            await this.setSelectedShoot(args.shootItem.metadata)
            this.dialog = args.action
          } catch (error) {
            // Currently not handled
          }
      }
    },
    hideDialog () {
      this.dialog = null
      // Delay resetting shoot so that the dialog does not lose context during closing animation
      this.clearSelectedShootWithDelay()
    },
    checkboxColor (checked) {
      return checked ? 'cyan darken-2' : ''
    },
    checkboxIcon (checked) {
      return checked ? 'mdi-checkbox-marked' : 'mdi-checkbox-blank-outline'
    },
    setColumnChecked (header) {
      header.checked = !header.checked
      this.saveColumnsChecked()
    },
    saveColumnsChecked () {
      const keys = map(this.allHeaders, 'value')
      const checkedValues = map(this.allHeaders, 'checked')
      const checkedColumns = zipObject(keys, checkedValues)

      this.$localStorage.setObject('dataTable_checkedColumns', checkedColumns)
    },
    resetColumnsChecked () {
      for (const header of this.allHeaders) {
        header.checked = header.defaultChecked
      }
      this.saveColumnsChecked()

      this.options = { itemsPerPage: 10 }
    },
    loadColumnsChecked () {
      const checkedColumns = this.$localStorage.getObject('dataTable_checkedColumns') || {}
      for (const header of this.allHeaders) {
        header.checked = get(checkedColumns, header.value, header.defaultChecked)
      }
    },
    setColumnVisibility () {
      for (const header of this.allHeaders) {
        switch (header.value) {
          case 'journalLabels':
          case 'journal':
            header.hidden = !(this.gitHubRepoUrl && this.isAdmin)
            break
          case 'actions':
            header.hidden = !(this.canDeleteShoots || this.canGetSecrets)
            break
          case 'project':
            header.hidden = !!this.projectScope
            break
          default:
            if (get(header, 'adminOnly', false)) {
              header.hidden = !this.isAdmin
            }
        }
      }
    },
    toggleFilter (key) {
      if (this.showOnlyShootsWithIssues) {
        const filters = this.getShootListFilters
        this.setShootListFilter({ filter: key, value: !filters[key] })
      }
    },
    isFilterActive (key) {
      const filters = this.getShootListFilters
      return get(filters, key, false)
    },
    setSelectedShoot (selectedShoot) {
      clearTimeout(this.clearSelectedShootTimerID)
      return this.setSelectedShootInternal(selectedShoot)
    },
    clearSelectedShootWithDelay () {
      this.clearSelectedShootTimerID = setTimeout(() => {
        this.setSelectedShootInternal(null)
      }, 500)
    }
  },
  computed: {
    ...mapGetters({
      mappedItems: 'shootList',
      item: 'shootByNamespaceAndName',
      selectedItem: 'selectedShoot',
      isAdmin: 'isAdmin',
      getShootListFilters: 'getShootListFilters',
      canPatchShoots: 'canPatchShoots',
      canDeleteShoots: 'canDeleteShoots',
      canCreateShoots: 'canCreateShoots',
      canGetSecrets: 'canGetSecrets'
    }),
    ...mapState([
      'shootsLoading',
      'onlyShootsWithIssues',
      'cfg',
      'namespace'
    ]),
    clusterAccessDialog: {
      get () {
        return this.dialog === 'access'
      },
      set (value) {
        if (!value) {
          this.hideDialog()
        }
      }
    },
    currentName () {
      return get(this.selectedItem, 'metadata.name')
    },
    headers () {
      return this.allHeaders.filter(e => e.hidden === false)
    },
    visibleHeaders () {
      return this.headers.filter(e => e.checked === true)
    },
    projectScope () {
      return this.namespace !== '_all'
    },
    showOnlyShootsWithIssues: {
      get () {
        return this.onlyShootsWithIssues
      },
      set (value) {
        this.setOnlyShootsWithIssues(value)
      }
    },
    items () {
      return this.cachedItems || this.mappedItems
    },
    filtersDisabled () {
      return !this.showOnlyShootsWithIssues
    },
    disabledFilterClass () {
      return this.filtersDisabled ? 'disabled_filter' : ''
    },
    headlineSubtitle () {
      const subtitle = []
      if (!this.projectScope && this.showOnlyShootsWithIssues) {
        subtitle.push('Hide: Healthy Clusters')
        if (this.isFilterActive('progressing')) {
          subtitle.push('Progressing Clusters')
        }
        if (this.isFilterActive('userIssues')) {
          subtitle.push('User Errors')
        }
        if (this.isFilterActive('deactivatedReconciliation')) {
          subtitle.push('Deactivated Reconciliation')
        }
        if (this.isFilterActive('hasJournals')) {
          subtitle.push('Has Journals')
        }
      }
      return join(subtitle, ', ')
    },
    gitHubRepoUrl () {
      return this.cfg.gitHubRepoUrl
    }
  },
  mounted () {
    this.floatingButton = true
    this.loadColumnsChecked()
    this.setColumnVisibility()
    this.setShootListFilters({
      progressing: true,
      userIssues: this.isAdmin,
      deactivatedReconciliation: this.isAdmin,
      hasJournals: false
    })
  },
  beforeRouteEnter (to, from, next) {
    next(vm => {
      vm.cachedItems = null
    })
  },
  beforeRouteUpdate (to, from, next) {
    this.search = null
    next()
  },
  beforeRouteLeave (to, from, next) {
    this.cachedItems = this.mappedItems.slice(0)
    this.search = null
    next()
  }
}
</script>

<style lang="scss" scoped >

  .dashboard {
    padding-top: 10px;
    padding-bottom: 10px;
  }

  .cluster_name {
    color: rgb(0, 137, 123);
  }

  .shootListTable table.table {
    thead, tbody {
      th, td {
        padding: 10px;
      }
    }
  }

  .shootListTable table {
    tbody, thead {
      td:first-child, th:first-child {
        padding-left: 24px;
      }
      td:last-child, th:last-child {
        padding-right: 24px;
      }
    }
  }

  .disabled_filter {
    opacity: 0.5;
  }

  .search_textfield {
    min-width: 125px;
  }

  .v-input__slot {
    margin: 0px;
  }

</style>
