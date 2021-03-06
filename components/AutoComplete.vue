<template>
  <v-autocomplete
    :value="value"
    :loading="loading"
    :items="items"
    :search-input.sync="autosearch"
    :multiple="multiple"
    :chips="multiple"
    :rules="rules"
    :error-messages="errors"
    :label="text"
    :item-text="displayProp"
    color="blue-grey lighten-2"
    item-value="id"
    @input="updateValue"
  >
    <template v-if="multiple" v-slot:selection="data">
      <v-chip
        v-bind="data.attrs"
        :input-value="data.selected"
        close
        @click="data.select"
        @click:close="removeMultiple(data.item)"
      >
        {{ data.item[displayProp] }}
      </v-chip>
    </template>
    <template v-slot:item="data">
      <template v-if="typeof data.item !== 'object'">
        <v-list-item-content v-text="data.item" />
      </template>
      <template v-else>
        <v-list-item-content>
          <v-list-item-title>
            {{ data.item[displayProp] }}
          </v-list-item-title>
        </v-list-item-content>
      </template>
    </template>
  </v-autocomplete>
</template>

<script>
export default {
  props: {
    value: {
      type: null,
      required: true
    },
    url: {
      type: String,
      required: true
    },
    multiple: {
      type: Boolean,
      default: false
    },
    rules: {
      type: Array,
      default () { return [] }
    },
    displayProp: {
      type: String,
      default: 'name'
    }

  },
  data () {
    return {
      loading: false,
      items: [],
      autosearch: '',
      errors: []
    }
  },
  computed: {
    text () {
      return this.url.charAt(0).toUpperCase() + this.url.slice(1)
    }
  },
  watch: {
    autosearch: {
      handler (val) {
        this.fetchData(val)
      },
      deep: true
    }
  },
  methods: {
    updateValue (value) {
      this.$emit('input', value)
    },
    fetchData (val) {
      if (val === null || typeof val === 'undefined') { val = '' }
      let limit = -1
      if (!val) { limit = 5 }
      const url = `/${this.url}?limit=${limit}&query=${val}&multiple=${this.multiple}`
      this.$axios.get(url).then((resp) => { this.items = resp.data.result })
    },
    removeMultiple (item) {
      const index = this.value.indexOf(item.id)
      if (index >= 0) { this.value.splice(index, 1) }
    }
  }
}
</script>
