<template>
  <div class="flex flex-col h-full">
    <div class="flex items-center flex-shrink-0 h-10 px-3">
      <input
        type="text"
        v-model="keyword"
        class="py-1 px-2 text-xs rounded-sm dark:text-white dark:bg-gray-900 border-gray-250 dark:border-gray-750 w-full placeholder-gray-600"
        placeholder="Filter"
      />
    </div>
    <div
      v-if="!entries.length"
      class="flex flex-grow justify-center items-center p-4 h-full text-center text-gray-600"
    >Waiting for requests...</div>
    <div
      v-else-if="!filteredEntries.length"
      class="flex flex-grow justify-center items-center p-4 h-full text-center text-gray-600"
    >
      <span>No results found {{ keyword ? `for "${keyword}"` : '' }}</span>
    </div>
    <Scroll v-else :scrollingX="false">
      <ul class="block flex-grow pt-1">
        <li
          v-for="entry in filteredEntries"
          :key="entry.id"
          v-tooltip="{ content: entry.request.url, container: false }"
          class="relative group flex items-center px-3 py-2 cursor-pointer hover:text-gray-800 dark:hover:text-white truncate"
          :class="[
            value === entry.id
              ? 'text-gray-850 bg-gray-150 dark:text-white dark:bg-gray-800'
              : 'text-gray-600 dark:text-gray-400 hover:bg-gray-150 dark:hover:bg-gray-800 dark:hover:text-gray-300',
            {
              'text-red-500 dark:text-red-400 hover:text-red-500 dark:hover:text-red-400':
                entry.response.isError,
            },
          ]"
          @click="$emit('input', entry.id)"
        >
          <CopyButton :value="getFileContent(entry)" />
          <FileExportButton :value="getFileContent(entry)" :name="entry.request.url"/>
          <div
            class="flex justify-center items-center h-5 w-5 rounded-sm uppercase font-semibold flex-shrink-0"
            :class="value === entry.id ? `text-white bg-${getColor(entry)}` : `bg-gray-200 dark:bg-gray-750 group-hover:bg-${getColor(entry)} text-gray-500 dark:text-gray-400 group-hover:text-white`"
          >{{ entry.type === 'GQL' ? entry.request.operationType[0] : entry.type[0] }}</div>
          <span class="font-semibold ml-2">
            {{ entry.type === 'GQL' && !entry.request.name ? entry.request.operations.join(', ') : entry.request.name }}
          </span>
          <span class="font-normal opacity-50">{{ entry.request.queryString }}</span>
        </li>
      </ul>
    </Scroll>
    <div
      class="flex justify-between items-center flex-shrink-0 h-10 px-3 border-t border-gray-200 dark:border-gray-750"
    >
      <ButtonGroup
        v-model="activeView"
        :items="viewButtons"
        activeBgClass="bg-gray-500 dark:bg-gray-700"
        class="flex-shrink-0"
        @input="changeView"
      />
      <button
        v-if="filteredEntries.length"
        class="flex h-10 w-10 justify-center items-center -mr-3 text-gray-500 hover:text-gray-800 dark:hover:text-gray-400 focus:outline-none"
        @click="clear"
      >
        <BanIcon class="h-5 w-5 focus:outline-none" v-tooltip="'Clear'" />
      </button>
    </div>
  </div>
</template>

<script>
import { mapState, mapMutations } from 'vuex'
import fuzzysearch from 'fuzzysearch'

import BanIcon from '@/assets/ban-icon.svg'

const ACTIVE_METHODS = {
  GQL: ['GQL'],
  XHR: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
}

export default {
  components: {
    BanIcon,
  },
  data() {
    return {
      keyword: '',
      activeView: 'ALL',
      viewButtons: [
        {
          title: 'ALL',
          name: 'ALL',
          label: 'All requests',
        },
        {
          title: 'GQL',
          name: 'GQL',
          label: 'GraphQL',
        },
        {
          title: 'XHR',
          name: 'XHR',
          label: 'XHR and Fetch',
        },
      ],
    }
  },
  props: {
    value: {
      type: [Number, String],
    },
    entries: {
      type: Array,
      default: () => [],
    },
  },
  computed: {
    ...mapState(['settings', 'lastState', 'typeColors']),
    filteredEntries() {
      return this.entries
        .filter((entry) => entry)
        .filter(({ type, request }) => {
          if (type === 'GQL')
            return request.operations.some((name) =>
              fuzzysearch(
                this.keyword.toLowerCase(),
                `${request.name}: ${name}`.toLowerCase()
              )
            )
          else
            return fuzzysearch(
              this.keyword.toLowerCase(),
              request.name.toLowerCase()
            )
        })
        .filter(
          ({ type }) =>
            this.activeView === 'ALL' ||
            ACTIVE_METHODS[this.activeView].includes(type)
        )
        .sort((a, b) =>
          this.settings.sortOption === 'newest' ? b.id - a.id : a.id - b.id
        )
    },
  },
  created() {
    this.activeView = this.lastState.activeView
  },
  methods: {
    ...mapMutations(['setSettings', 'setLastState']),
    async getFileContent(entry) {
      // const params = entry.request.params
      // const body = entry.request.body
      // const variables = this.entry.request.variables
      const {data, errors} = await entry.response.getResponse()
      return JSON.stringify({
        request: entry.request,
        response: {
          ...entry.response,
          data,
          errors
        }
      }, null, 2)
    },
    getColor(entry) {
      return this.typeColors[entry.type === 'GQL' ? entry.request.operationType.toUpperCase() : entry.type]
    },
    changeView() {
      const hasTarget = this.filteredEntries.find(
        (item) => item.id === this.value
      )

      this.setLastState({
        activeView: this.activeView,
      })

      if (!hasTarget)
        this.$emit(
          'input',
          this.filteredEntries.length ? this.filteredEntries[0].id : null
        )
    },
    clear() {
      this.$emit('clear')
    },
  },
}
</script>
