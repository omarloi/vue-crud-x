<template>
  <v-container fluid>
    <v-layout wrap>
      <v-flex xs12>
        <v-combobox
          v-model="select"
          :items="itemsz"
          label="Select a favorite activity or create a new one"
          multiple
          chips
        ></v-combobox>
      </v-flex>
    </v-layout>
    <v-card-text>
      <v-autocomplete
        v-model="model"
        :items="items"
        :loading="isLoading"
        :search-input.sync="search"
        color="white"
        hide-no-data
        hide-selected
        item-text="Description"
        item-value="API"
        label="Public APIs"
        placeholder="Start typing to Search"
        prepend-icon="mdi-database-search"
        return-object
      ></v-autocomplete>
    </v-card-text>
    <v-divider></v-divider>
    <v-expand-transition>
      <v-list v-if="model" class="red lighten-3">
        <v-list-tile
          v-for="(field, i) in fields"
          :key="i"
        >
          <v-list-tile-content>
            <v-list-tile-title v-text="field.value"></v-list-tile-title>
            <v-list-tile-sub-title v-text="field.key"></v-list-tile-sub-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
    </v-expand-transition>
    <drawing-canvas />
    <web-cam />
  </v-container>
</template>

<!-- <template>
  <v-card
    color="red lighten-2"
    dark
  >
    <v-card-title class="headline red lighten-3">
      Search for Public APIs
    </v-card-title>
    <v-card-text>
      Explore hundreds of free API's ready for consumption! For more information visit
      <a
        class="grey--text text--lighten-3"
        href="https://github.com/toddmotto/public-apis"
        target="_blank"
      >the Github repository</a>.
    </v-card-text>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn
        :disabled="!model"
        color="grey darken-3"
        @click="model = null"
      >
        Clear
        <v-icon right>mdi-close-circle</v-icon>
      </v-btn>
    </v-card-actions>
  </v-card>
</template> -->

<script>
import axios from 'axios'
import DrawingCanvas from '@/components/DrawingCanvas'
import WebCam from '@/components/WebCam'

export default {
  components: {
    DrawingCanvas,
    WebCam
  },
  data: () => ({
    select: ['Vuetify', 'Programming'],
    itemsz: [
      'Programming',
      'Design',
      'Vue',
      'Vuetify'
    ],
    descriptionLimit: 60,
    entries: [],
    isLoading: false,
    model: null,
    search: null
  }),

  computed: {
    fields () {
      if (!this.model) return []
      return Object.keys(this.model).map(key => {
        return {
          key: key,
          value: this.model[key] || 'n/a'
        }
      })
    },
    items () {
      return this.entries.map(entry => {
        const Description = entry.Description.length > this.descriptionLimit
          ? entry.Description.slice(0, this.descriptionLimit) + '...'
          : entry.Description
        return Object.assign({}, entry, { Description })
      })
    }
  },
  watch: {
    search (val) {
      // Items have already been loaded
      if (this.items.length > 0) return
      this.isLoading = true
      // Lazily load input items
      axios.get('https://api.publicapis.org/entries')
        .then(res => {
          const { count, entries } = res.data
          this.count = count
          this.entries = entries
        })
        .catch(err => {
          console.log(err)
        })
        .finally(() => (this.isLoading = false))
    }
  }
}
</script>
