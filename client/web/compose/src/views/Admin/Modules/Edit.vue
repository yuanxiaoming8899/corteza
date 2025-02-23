<template>
  <div class="py-3">
    <portal to="topbar-title">
      {{ title }}
    </portal>
    <portal to="topbar-tools">
      <b-button-group
        v-if="isEdit"
        data-test-id="button-all-records"
        size="sm"
        class="mr-1"
      >
        <b-button
          variant="primary"
          :to="allRecords"
          class="d-flex align-items-center"
        >
          {{ $t('allRecords.label') }}
          <font-awesome-icon
            :icon="['fas', 'columns']"
            class="ml-2"
          />
        </b-button>

        <module-translator
          v-if="module"
          :module.sync="trModule"
          style="margin-left:2px;"
        />
      </b-button-group>
    </portal>

    <div
      v-if="!module"
      class="d-flex align-items-center justify-content-center h-100"
    >
      <b-spinner />
    </div>

    <b-container
      v-else
      tag="form"
      fluid="xl"
      @submit.prevent="handleSave"
    >
      <b-row no-gutters>
        <b-col>
          <b-card
            no-body
            class="shadow-sm"
          >
            <b-card-header
              v-if="isEdit"
              header-bg-variant="white"
              class="py-3"
            >
              <b-row
                no-gutters
                class="d-flex align-items-center flex-fill-child gap-1"
              >
                <b-button
                  v-if="federationEnabled"
                  data-test-id="button-federation-settings"
                  variant="light"
                  size="lg"
                  class="mr-1"
                  @click="federationSettings.modal = true"
                >
                  <font-awesome-icon
                    :icon="['fas', 'share-alt']"
                  />

                  {{ $t('edit.federationSettings.title') }}
                </b-button>

                <b-button
                  v-if="discoveryEnabled"
                  data-test-id="button-discovery-settings"
                  variant="light"
                  size="lg"
                  class="mr-1"
                  @click="discoverySettings.modal = true"
                >
                  <font-awesome-icon
                    :icon="['fas', 'search-location']"
                  />
                  {{ $t('edit.discoverySettings.title') }}
                </b-button>

                <export
                  :list="[module]"
                  type="module"
                  class="mr-1"
                />

                <b-dropdown
                  v-if="module.canGrant"
                  data-test-id="dropdown-permissions"
                  size="lg"
                  variant="light"
                  class="permissions-dropdown mr-1"
                >
                  <template #button-content>
                    <font-awesome-icon :icon="['fas', 'lock']" />
                    <span>
                      {{ $t('general.label.permissions') }}
                    </span>
                  </template>

                  <b-dropdown-item>
                    <c-permissions-button
                      :title="module.name || module.handle || module.moduleID"
                      :target="module.name || module.handle || module.moduleID"
                      :resource="`corteza::compose:module/${namespace.namespaceID}/${module.moduleID}`"
                      :button-label="$t('general:label.module.single')"
                      :show-button-icon="false"
                      button-variant="white text-left w-100"
                    />
                  </b-dropdown-item>

                  <b-dropdown-item>
                    <c-permissions-button
                      :title="module.name || module.handle || module.moduleID"
                      :target="module.name || module.handle || module.moduleID"
                      :resource="`corteza::compose:module-field/${namespace.namespaceID}/${module.moduleID}/*`"
                      :button-label="$t('general:label.field')"
                      :show-button-icon="false"
                      all-specific
                      button-variant="white text-left w-100"
                    />
                  </b-dropdown-item>

                  <b-dropdown-item>
                    <c-permissions-button
                      :title="module.name || module.handle || module.moduleID"
                      :target="module.name || module.handle || module.moduleID"
                      :resource="`corteza::compose:record/${namespace.namespaceID}/${module.moduleID}/*`"
                      :button-label="$t('general:label.record')"
                      :show-button-icon="false"
                      all-specific
                      button-variant="white text-left w-100"
                    />
                  </b-dropdown-item>
                </b-dropdown>

                <related-pages
                  :namespace="namespace"
                  :module="module"
                  size="lg"
                  class="d-flex ml-auto"
                />
              </b-row>
            </b-card-header>

            <b-tabs
              v-model="activeTab"
              :nav-wrapper-class="`bg-white white border-bottom ${isEdit ? 'rounded-0' : ''}`"
              card
            >
              <b-tab
                :title="$t('edit.fields.label')"
                active
              >
                <h5 class="mb-3">
                  {{ $t('edit.moduleInfo') }}
                </h5>

                <b-row>
                  <b-col
                    cols="12"
                    md="6"
                    xl="4"
                  >
                    <b-form-group
                      :label="$t('newLabel')"
                      label-class="text-primary"
                    >
                      <b-form-input
                        v-model="module.name"
                        data-test-id="input-module-name"
                        required
                        :state="nameState"
                        :placeholder="$t('newPlaceholder')"
                      />
                    </b-form-group>
                  </b-col>

                  <b-col
                    cols="12"
                    md="6"
                    xl="4"
                  >
                    <b-form-group
                      :label="$t('general.label.handle')"
                      label-class="text-primary"
                    >
                      <b-form-input
                        v-model="module.handle"
                        data-test-id="input-module-handle"
                        :state="handleState"
                        :placeholder="$t('general.placeholder.handle')"
                        class="mb-2"
                      />
                      <b-form-invalid-feedback :state="handleState">
                        {{ $t('general.placeholder.invalid-handle-characters') }}
                      </b-form-invalid-feedback>
                    </b-form-group>
                  </b-col>
                </b-row>

                <hr>

                <h5 class="mb-3">
                  {{ $t('edit.manageRecordFields') }}
                </h5>

                <b-row no-gutters>
                  <b-form-group class="w-100">
                    <table
                      data-test-id="table-module-fields"
                      class="table table-sm table-borderless table-responsive-lg"
                    >
                      <thead>
                        <tr>
                          <th />

                          <th
                            class="text-primary"
                          >
                            <div
                              class="d-flex align-items-center"
                            >
                              {{ $t('general.label.name') }}
                              <div
                                v-b-tooltip.hover.topright="{ title: $t('edit.tooltip.name'), container: '#body' }"
                                class="ml-1"
                              >
                                <font-awesome-icon
                                  :icon="['far', 'question-circle']"
                                />
                              </div>
                            </div>
                          </th>

                          <th
                            class="text-primary"
                          >
                            <div
                              class="d-flex align-items-center"
                            >
                              {{ $t('general.label.title') }}
                              <div
                                v-b-tooltip.hover.topright="{ title: $t('edit.tooltip.title'), container: '#body' }"
                                class="ml-1"
                              >
                                <font-awesome-icon
                                  :icon="['far', 'question-circle']"
                                />
                              </div>
                            </div>
                          </th>

                          <th class="text-primary">
                            {{ $t('general:label.type') }}
                          </th>

                          <th />
                          <th />

                          <th class="text-primary text-center pr-3">
                            {{ $t('general:label.required') }}
                          </th>

                          <th class="text-primary text-center pl-2">
                            {{ $t('general:label.multi') }}
                          </th>

                          <th />
                        </tr>
                      </thead>

                      <draggable
                        v-model="module.fields"
                        handle=".handle"
                        tag="tbody"
                      >
                        <field-row-edit
                          v-for="(field, index) in module.fields"
                          :key="index"
                          v-model="module.fields[index]"
                          :can-grant="namespace.canGrant"
                          :has-records="hasRecords"
                          :module="module"
                          :is-duplicate="!!duplicateFields[index]"
                          @edit="handleFieldEdit(module.fields[index])"
                          @delete="module.fields.splice(index, 1)"
                          @updateKind="handleFieldKindUpdate(index)"
                        />
                      </draggable>

                      <tr>
                        <td colspan="1" />
                        <td colspan="7">
                          <b-button
                            data-test-id="button-field-add"
                            class="mb-5"
                            variant="primary"
                            @click="handleNewField"
                          >
                            + {{ $t('edit.newField') }}
                          </b-button>
                        </td>
                      </tr>
                      <tr>
                        <td
                          colspan="7"
                          class="font-weight-bold"
                        >
                          {{ $t('edit.systemFields') }}
                        </td>
                      </tr>

                      <field-row-view
                        v-for="(field, index) in systemFields"
                        :key="index"
                        :field="field"
                        class="mt-4"
                      />
                    </table>
                  </b-form-group>
                </b-row>
              </b-tab>

              <b-tab :title="$t('edit.config.dal.title')">
                <dal-settings
                  :module="module"
                />
              </b-tab>

              <b-tab
                :title="$t('edit.config.uniqueValues.title')"
              >
                <unique-values
                  :module="module"
                />
              </b-tab>

              <b-tab :title="$t('edit.config.record-revisions.title')">
                <record-revisions-settings
                  :module="module"
                />
              </b-tab>

              <b-tab :title="$t('edit.config.privacy.title')">
                <data-privacy-settings
                  v-if="connection"
                  :resource="module"
                  :connection="connection"
                  :sensitivity-levels="sensitivityLevels"
                  :max-level="maxLevelID"
                  :translations="{
                    sensitivity: {
                      label: $t('edit.config.privacy.sensitivity-level.label'),
                      description: $t('edit.config.privacy.sensitivity-level.description'),
                      placeholder: $t('edit.config.privacy.sensitivity-level.placeholder'),
                    },
                    usage: {
                      label: $t('edit.config.privacy.usage-disclosure.label'),
                    },
                  }"
                />
              </b-tab>

              <b-tab
                v-if="module.issues.length > 0"
                :title="$t('edit.issues.label', { count: module.issues.length })"
                title-link-class="text-danger"
                @click="checkAlterations"
              >
                <b-alert
                  v-for="(issue, index) in module.issues"
                  :key="index"
                  show
                  variant="danger"
                >
                  {{ issue.issue }}
                </b-alert>
              </b-tab>
            </b-tabs>
          </b-card>
        </b-col>
      </b-row>

      <b-modal
        v-if="updateField"
        :title="editModalTitle"
        :ok-title="$t('general.label.saveAndClose')"
        ok-only
        ok-variant="primary"
        size="lg"
        :visible="!!updateField"
        body-class="p-0 border-top-0"
        header-class="p-3 pb-0 border-bottom-0"
        no-fade
        @ok="handleFieldSave(updateField)"
        @hide="updateField=null"
      >
        <field-configurator
          :field.sync="updateField"
          :namespace="namespace"
          :module="module"
          :connection="connection"
          :sensitivity-levels="sensitivityLevels"
        />
      </b-modal>

      <dal-schema-alterations
        :batch="dalSchemaAlterations.batchID"
        :module="module"
        @hide="onDalAlterationsHide"
      />

      <federation-settings
        v-if="federationEnabled"
        :modal="federationSettings.modal"
        :module="module"
        @change="federationSettings.modal = ($event || false)"
      />

      <discovery-settings
        v-if="discoveryEnabled"
        :modal.sync="discoverySettings.modal"
        :module="module"
        @save="onDiscoverySettingsSave"
      />
    </b-container>

    <portal to="admin-toolbar">
      <editor-toolbar
        :processing="processing"
        :processing-save="processingSave"
        :processing-save-and-close="processingSaveAndClose"
        :processing-delete="processingDelete"
        :hide-delete="hideDelete"
        :hide-clone="!isEdit"
        :hide-save="hideSave"
        :disable-save="disableSave"
        @delete="handleDelete"
        @save="handleSave()"
        @saveAndClose="handleSave({ closeOnSuccess: true })"
        @clone="handleClone"
        @back="$router.push(previousPage || { name: 'admin.modules' })"
      />
    </portal>
  </div>
</template>

<script>
import { isEqual } from 'lodash'
import { mapGetters, mapActions } from 'vuex'
import draggable from 'vuedraggable'
import FieldConfigurator from 'corteza-webapp-compose/src/components/ModuleFields/Configurator'
import FieldRowEdit from 'corteza-webapp-compose/src/components/Admin/Module/FieldRowEdit'
import FieldRowView from 'corteza-webapp-compose/src/components/Admin/Module/FieldRowView'
import FederationSettings from 'corteza-webapp-compose/src/components/Admin/Module/FederationSettings'
import DalSchemaAlterations from 'corteza-webapp-compose/src/components/Admin/Module/DalSchemaAlterations'
import DiscoverySettings from 'corteza-webapp-compose/src/components/Admin/Module/DiscoverySettings'
import DalSettings from 'corteza-webapp-compose/src/components/Admin/Module/DalSettings'
import RecordRevisionsSettings from 'corteza-webapp-compose/src/components/Admin/Module/RecordRevisionsSettings'
import DataPrivacySettings from 'corteza-webapp-compose/src/components/Admin/Module/DataPrivacySettings'
import ModuleTranslator from 'corteza-webapp-compose/src/components/Admin/Module/ModuleTranslator'
import UniqueValues from 'corteza-webapp-compose/src/components/Admin/Module/UniqueValues'
import RelatedPages from 'corteza-webapp-compose/src/components/Admin/Module/RelatedPages'
import { compose, NoID } from '@cortezaproject/corteza-js'
import EditorToolbar from 'corteza-webapp-compose/src/components/Admin/EditorToolbar'
import Export from 'corteza-webapp-compose/src/components/Admin/Export'
import { handle } from '@cortezaproject/corteza-vue'

export default {
  name: 'ModulesEdit',

  i18nOptions: {
    namespaces: 'module',
  },

  components: {
    draggable,
    FieldConfigurator,
    FieldRowEdit,
    FieldRowView,
    FederationSettings,
    DalSchemaAlterations,
    DiscoverySettings,
    DalSettings,
    RecordRevisionsSettings,
    DataPrivacySettings,
    ModuleTranslator,
    RelatedPages,
    EditorToolbar,
    Export,
    UniqueValues,
  },

  props: {
    namespace: {
      type: compose.Namespace,
      required: true,
    },

    moduleID: {
      type: String,
      required: false,
      default: NoID,
    },
  },

  data () {
    return {
      activeTab: 0,

      connection: undefined,
      sensitivityLevels: [],

      updateField: null,
      module: undefined,
      initialModuleState: undefined,
      hasRecords: true,
      processing: false,
      processingSave: false,
      processingSaveAndClose: false,
      processingDelete: false,

      federationSettings: {
        modal: false,
      },

      dalSchemaAlterations: {
        modal: false,
        batchID: undefined,
      },

      discoverySettings: {
        modal: false,
      },

      abortableRequests: [],
    }
  },

  computed: {
    ...mapGetters({
      pages: 'page/set',
      previousPage: 'ui/previousPage',
    }),

    title () {
      return this.$route.name === 'admin.modules.edit' ? this.$t('edit.edit') : this.$t('edit.create')
    },

    isNew () {
      return this.moduleID === NoID
    },

    trModule: {
      get () {
        if (!this.module) {
          return new compose.Module()
        }
        return this.module
      },
      set (v) {
        this.module = v
        this.updateModuleSet(v)
      },
    },

    nameState () {
      return this.module.name.length > 0 ? null : false
    },

    handleState () {
      return handle.handleState(this.module.handle)
    },

    duplicateFields () {
      const rtr = {}
      const ix = new Set()
      const { fields = [] } = this.module || {}

      fields.forEach((f, i) => {
        if (ix.has(f.name)) {
          rtr[i] = f
        }
        ix.add(f.name)
      })

      return rtr
    },

    fieldsValid () {
      const { fields = [] } = this.module || {}
      const valid = !fields.some(f => {
        return f.fieldID === NoID && !f.isValid
      })

      const unique = Object.keys(this.duplicateFields).length === 0

      return valid && unique
    },

    systemFields () {
      const systemFieldEncoding = this.module.config.dal.systemFieldEncoding || {}

      return this.module.systemFields().map(sf => {
        if (!sf) return
        sf.label = this.$t(`field:system.${sf.name}`)
        return { ...sf, ...(systemFieldEncoding[sf.name] || {}) }
      }).filter(sf => sf)
    },

    editModalTitle () {
      if (!this.updateField) {
        return
      }

      const { name } = this.updateField
      return name ? this.$t('edit.specificFieldSettings', { name: this.updateField.name }) : this.$t('edit.moduleFieldSettings')
    },

    federationEnabled () {
      return this.isEdit && this.$Settings.get('federation.enabled', false)
    },

    discoveryEnabled () {
      return this.$Settings.get('discovery.enabled', false)
    },

    hideDelete () {
      return !this.isEdit || !this.module.canDeleteModule || !!this.module.deletedAt
    },

    disableSave () {
      return !this.module || [this.fieldsValid, this.nameState, this.handleState].includes(false)
    },

    hideSave () {
      return this.isEdit && !this.module.canUpdateModule
    },

    isEdit () {
      return this.module && this.module.moduleID !== NoID
    },

    allRecords () {
      return { name: 'admin.modules.record.list', params: { moduleID: this.moduleID } }
    },

    maxLevelID () {
      const { sensitivityLevelID = NoID } = this.connection.config.privacy || {}
      return sensitivityLevelID
    },
  },

  watch: {
    moduleID: {
      immediate: true,
      async handler (moduleID) {
        await this.fetchModule(moduleID)
        this.fetchSensitivityLevels()
        this.checkAlterations()
      },
    },

    'module.config.dal.connectionID': {
      handler (connectionID) {
        this.fetchConnection(connectionID)
      },
    },
  },

  beforeDestroy () {
    this.abortRequests()
    this.setDefaultValues()
  },

  beforeRouteUpdate (to, from, next) {
    this.checkUnsavedModule(next)
  },

  beforeRouteLeave (to, from, next) {
    this.checkUnsavedModule(next)
  },

  methods: {
    ...mapActions({
      findModuleByID: 'module/findByID',
      updateModule: 'module/update',
      updateModuleSet: 'module/updateSet',
      createModule: 'module/create',
      deleteModule: 'module/delete',
      deletePage: 'page/delete',
    }),

    checkUnsavedModule (next) {
      if (this.isNew) {
        next(true)
      } else if (!this.module.deletedAt) {
        const moduleState = this.module ? this.module.clone() : {}
        const initialModuleState = this.initialModuleState ? this.initialModuleState.clone() : {}

        next(!isEqual(moduleState, initialModuleState) ? window.confirm(this.$t('general.unsavedChanges')) : true)
      } else {
        next(true)
      }
    },

    handleNewField () {
      this.module.fields.push(new compose.ModuleFieldString())
    },

    handleFieldEdit (field) {
      this.updateField = compose.ModuleFieldMaker({ ...field })
    },

    handleFieldKindUpdate (index) {
      const field = this.module.fields[index]
      this.module.fields.splice(index, 1, compose.ModuleFieldMaker({ ...field }))
    },

    handleFieldSave (field) {
      const i = this.module.fields.findIndex(f => f.name === field.name)
      if (i > -1) {
        this.module.fields.splice(i, 1, field)
      }
    },

    onDiscoverySettingsSave (changes) {
      this.module.config = { ...this.module.config, ...changes }
    },

    onPrivacySettingsSave (changes) {
      this.module.config = { ...this.module.config, ...changes }
    },

    handleSave ({ closeOnSuccess = false, module = this.module } = {}) {
      /**
       * Pass a special tag alongside payload that
       * instructs store layer to add content-language header to the API request
       */
      const resourceTranslationLanguage = this.currentLanguage
      this.processing = true

      if (closeOnSuccess) {
        this.processingSaveAndClose = true
      } else {
        this.processingSave = true
      }

      if (module.moduleID === NoID) {
        // Filter out record fields that reference this not yet created module
        let fields = []
        const toBeUpdatedFields = []
        module.fields.forEach(f => {
          if (f.kind === 'Record' && f.options.moduleID === '-1') {
            toBeUpdatedFields.push(f)
          } else {
            fields.push(f)
          }
        })

        // If such fields exist , after module is created add fields, map moduleID and update module
        // Unfortunately this ruins the initial field order, but we can improve this later
        this.createModule({ ...module, fields, resourceTranslationLanguage }).then(async module => {
          if (toBeUpdatedFields.length) {
            fields = [
              ...module.fields,
              ...toBeUpdatedFields.map(f => {
                f.options.moduleID = module.moduleID
                return f
              }),
            ]

            module = await this.updateModule({ ...module, fields })
          }

          this.module = new compose.Module({ ...module }, this.namespace)
          this.initialModuleState = this.module.clone()

          this.toastSuccess(this.$t('notification:module.created'))
          if (closeOnSuccess) {
            this.$router.push(this.previousPage || { name: 'admin.modules' })
          } else {
            this.$router.push({ name: 'admin.modules.edit', params: { moduleID: this.module.moduleID } })
          }
        }).catch(this.toastErrorHandler(this.$t('notification:module.saveFailed')))
          .finally(() => {
            this.processing = false

            if (closeOnSuccess) {
              this.processingSaveAndClose = false
            } else {
              this.processingSave = false
            }
          })
      } else {
        this.updateModule({ ...module, resourceTranslationLanguage }).then(module => {
          this.module = new compose.Module({ ...module }, this.namespace)
          this.initialModuleState = this.module.clone()

          this.toastSuccess(this.$t('notification:module.saved'))
          if (closeOnSuccess) {
            this.$router.push({ name: 'admin.modules' })
          }
        }).catch(this.toastErrorHandler(this.$t('notification:module.saveFailed')))
          .finally(() => {
            this.processing = false

            if (closeOnSuccess) {
              this.processingSaveAndClose = false
            } else {
              this.processingSave = false
            }
          })
      }
    },

    async onDalAlterationsHide () {
      await this.fetchModule(this.moduleID)
      this.dalSchemaAlterations.batchID = undefined
    },

    async fetchModule (moduleID = this.moduleID) {
      this.module = undefined
      this.initialModuleState = undefined

      /**
       * Every time module changes we switch to the 1st tab
       */
      this.activeTab = 0

      if (moduleID === NoID) {
        this.module = new compose.Module(
          { fields: [new compose.ModuleFieldString({ fieldID: NoID, name: this.$t('general.placeholder.sample') })] },
          this.namespace,
        )
      } else {
        const params = {
          // make sure module is loaded from the API every time!
          force: true,
          namespace: this.namespace,
          moduleID: moduleID,
        }

        await this.findModuleByID(params).then((module) => {
          // Make a copy so that we do not change store item by ref
          this.module = module.clone()

          const { moduleID, namespaceID, issues = [] } = this.module
          if (issues.length > 0) {
            // do not proceed with record search as it's
            // likely to fail due to issues on a module
            return
          }

          // Count existing records to see what we can do with this module
          const { response, cancel } = this.$ComposeAPI
            .recordListCancellable({ moduleID, namespaceID, limit: 1 })

          this.abortableRequests.push(cancel)

          response()
            .then(({ set }) => { this.hasRecords = (set.length > 0) })
        })
      }

      this.initialModuleState = this.module.clone()
    },

    checkAlterations () {
      const { issues = [] } = this.module || {}

      if (!issues.length) {
        return
      }

      // Check if module has Alterations to resolve
      this.dalSchemaAlterations.batchID = undefined
      for (const i of this.module.issues) {
        if (i.meta.batchID) {
          this.dalSchemaAlterations.batchID = i.meta.batchID
          break
        }
      }
    },

    handleDelete () {
      this.processing = true
      this.processingDelete = true

      this.deleteModule(this.module).then(() => {
        this.module.deletedAt = new Date()

        const moduleRecordPage = this.pages.find(p => p.moduleID === this.module.moduleID)
        if (moduleRecordPage) {
          return this.deletePage({ ...moduleRecordPage, strategy: 'rebase' })
        }
      })
        .catch(this.toastErrorHandler(this.$t('notification:module.deleteFailed')))
        .finally(() => {
          this.toastSuccess(this.$t('notification:module.deleted'))
          this.processing = false
          this.processingDelete = false
          this.$router.push({ name: 'admin.modules' })
        })
    },

    handleClone () {
      const module = this.module.clone()
      module.moduleID = NoID
      module.name = `${this.module.name} (copy)`
      module.handle = this.module.handle ? `${this.module.handle}_copy` : ''

      this.handleSave({ module })
    },

    async fetchConnection (connectionID) {
      if (connectionID && connectionID !== NoID) {
        this.$SystemAPI.dalConnectionRead({ connectionID })
          .then(connection => {
            this.connection = connection
            this.initialModuleState.config.dal.connectionID = connection.connectionID
          })
          .catch(this.toastErrorHandler(this.$t('notification:connection.read-failed')))
          .finally(() => {
            this.processing = false
          })
      }
    },

    async fetchSensitivityLevels () {
      this.processing = true

      return this.$SystemAPI.dalSensitivityLevelList()
        .then(({ set = [] }) => {
          this.sensitivityLevels = set
        })
        .catch(this.toastErrorHandler(this.$t('notification:sensitivity-level.fetch-failed')))
        .finally(() => {
          this.processing = false
        })
    },

    setDefaultValues () {
      this.activeTab = 0
      this.connection = undefined
      this.sensitivityLevels = []
      this.updateField = null
      this.module = undefined
      this.initialModuleState = undefined
      this.hasRecords = true
      this.processing = false
      this.processingSaveAndClose = false
      this.processingSave = false
      this.federationSettings = {}
      this.discoverySettings = {}
      this.abortableRequests = []
    },

    abortRequests () {
      this.abortableRequests.forEach((cancel) => {
        cancel()
      })
    },
  },
}
</script>

<style scoped lang="scss">
@media (max-width: 576px) {
  .flex-fill-child > * {
    flex-grow: 1;
  }
}
</style>
