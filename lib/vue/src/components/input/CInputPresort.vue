<template>
  <div>
    <b-table-simple
      v-if="!textInput"
      borderless
      small
      responsive="lg"
      class="mb-0"
    >
      <draggable
        :list.sync="items"
        group="sort"
        handle=".grab"
        tag="tbody"
      >
        <tr
          v-for="(column, index) in items"
          :key="index"
        >
          <td
            class="grab text-center align-middle"
            style="width: 40px;"
          >
            <font-awesome-icon
              :icon="['fas', 'bars']"
            />
          </td>

          <td
            class="align-middle"
            style="min-width: 250px;"
          >
            <b-form-select
              v-model="column.field"
              :options="availableFields"
              text-field="label"
              value-field="name"
              class="rounded"
            >
              <template #first>
                <b-form-select-option
                  :value="undefined"
                  disabled
                >
                  {{ labels.none }}
                </b-form-select-option>
              </template>
            </b-form-select>
          </td>

          <td
            class="text-center align-middle"
            style="min-width: 200px;"
          >
            <b-form-radio-group
              v-model="column.descending"
              :options="sortDirections"
              buttons
              button-variant="outline-primary"
              class="bg-white"
            />
          </td>

          <td
            class="align-middle text-right"
            style="min-width: 80px; width: 80px;"
          >
            <c-input-confirm
              show-icon
              @confirmed="items.splice(index, 1)"
            />
          </td>
        </tr>
      </draggable>
    </b-table-simple>

    <div
      v-else
    >
      <b-form-textarea
        v-model="presortValue"
        :placeholder="labels.placeholder"
      />
      <b-form-text>
        {{ labels.footnote }}
      </b-form-text>
    </div>

    <div
      class="d-flex align-items-center mt-1"
    >
      <b-button
        v-if="!textInput"
        variant="primary"
        size="sm"
        @click="items.push({ field: undefined, descending: false })"
      >
        <font-awesome-icon
          :icon="['fas', 'plus']"
          class="mr-1"
        />
        {{ labels.add }}
      </b-button>

      <b-button
        v-if="allowTextInput"
        variant="link"
        size="sm"
        class="text-decoration-none ml-auto"
        @click="textInput = !textInput"
      >
        {{ labels.toggleInput }}
      </b-button>
    </div>
  </div>
</template>

<script>
import Draggable from 'vuedraggable'

export default {
  components: {
    Draggable,
  },

  props: {
    value: {
      type: String,
      required: true,
    },

    fields: {
      type: Array,
      required: true,
    },

    labels: {
      type: Object,
      required: true,
    },

    allowTextInput: {
      type: Boolean,
      default: false,
    },
  },

  data () {
    return {
      items: [],

      textInput: false,
    }
  },

  computed: {
    presortValue: {
      get () {
        return this.value
      },

      set (value) {
        this.$emit('input', value)
      },
    },

    sortDirections () {
      return [
        { value: false, text: this.labels.ascending },
        { value: true, text: this.labels.descending },
      ]
    },

    availableFields () {
      return this.fields.map(f => ({ ...f,label: `${f.label} (${f.name})` }))
    }
  },

  watch: {
    value: {
      immediate: true,
      handler (value) {
        if (value) {
          const sort = value.includes(',') ? value.split(',') : [value]

          this.items = sort.map(field => {
            let descending = false

            if (field.includes(' ')) {
              field = field.split(' ')[0]
              descending = true
            }

            return {
              field,
              descending: !!descending,
            }
          })
        } else {
          this.items = [{
            field: undefined,
            descending: false,
          }]
        }
      },
    },

    items: {
      deep: true,
      handler (items = [], oldItems = undefined) {
        if (oldItems) {
          this.$emit('input', items.filter(({ field }) => field).map(({ field, descending }) => {
            return descending ? `${field} DESC` : field
          }).join(','))
        }
      },
    },
  },

  beforeDestroy () {
    this.setDefaultValues()
  },

  methods: {
    setDefaultValues () {
      this.items = []
      this.textInput = false
    },
  },
}
</script>
