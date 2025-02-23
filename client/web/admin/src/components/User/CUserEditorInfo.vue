<template>
  <b-card
    class="shadow-sm"
    data-test-id="card-user-info"
    header-bg-variant="white"
    footer-bg-variant="white"
    footer-class="d-flex flex-wrap flex-fill-child gap-1"
  >
    <b-form
      @submit.prevent="$emit('submit', user)"
    >
      <b-row>
        <b-col
          cols="12"
          lg="6"
        >
          <b-form-group
            :label="$t('email')"
            label-class="text-primary"
          >
            <b-form-input
              v-model="user.email"
              data-test-id="input-email"
              required
              :state="emailState"
              type="email"
            />
          </b-form-group>
        </b-col>

        <b-col
          cols="12"
          lg="6"
        >
          <b-form-group
            :label="$t('name')"
            label-class="text-primary"
          >
            <b-form-input
              v-model="user.name"
              data-test-id="input-name"
              required
            />
          </b-form-group>
        </b-col>

        <b-col
          cols="12"
          lg="6"
        >
          <b-form-group
            :label="$t('handle')"
            :class="{ 'mb-0': !user.userID }"
            label-class="text-primary"
          >
            <b-form-input
              v-model="user.handle"
              data-test-id="input-handle"
              :placeholder="$t('placeholder-handle')"
              :state="handleState"
            />
            <b-form-invalid-feedback :state="handleState">
              {{ $t('invalid-handle-characters') }}
            </b-form-invalid-feedback>
          </b-form-group>
        </b-col>
      </b-row>

      <c-system-fields
        :resource="user"
      />

      <!--
        include hidden input to enable
        trigger submit event w/ ENTER
      -->
      <input
        type="submit"
        class="d-none"
        :disabled="saveDisabled"
      >
    </b-form>

    <template #header>
      <h3 class="m-0">
        {{ $t('title') }}
      </h3>
    </template>

    <template #footer>
      <confirmation-toggle
        v-if="!fresh && user.canDeleteUser"
        :data-test-id="deletedButtonStatusCypressId"
        @confirmed="$emit('delete')"
      >
        {{ getDeleteStatus }}
      </confirmation-toggle>

      <confirmation-toggle
        v-if="!fresh"
        :data-test-id="suspendButtonStatusCypressId"
        cta-class="light"
        @confirmed="$emit('status')"
      >
        {{ getSuspendStatus }}
      </confirmation-toggle>

      <confirmation-toggle
        v-if="!fresh"
        data-test-id="button-sessions-revoke"
        :disabled="user.userID === userID"
        cta-class="secondary"
        @confirmed="$emit('sessionsRevoke')"
      >
        {{ $t('revokeAllSession') }}
      </confirmation-toggle>

      <b-button
        v-if="!fresh && !user.emailConfirmed"
        variant="light"
        @click="$emit('patch', '/emailConfirmed', true)"
      >
        {{ $t('confirmEmail') }}
      </b-button>

      <c-corredor-manual-buttons
        ui-page="user/editor"
        ui-slot="infoFooter"
        resource-type="system:user"
        default-variant="secondary"
        @click="dispatchCortezaSystemUserEvent($event, { user })"
      />

      <c-button-submit
        :disabled="saveDisabled"
        :processing="processing"
        :success="success"
        :text="$t('admin:general.label.submit')"
        class="ml-auto"
        @submit="$emit('submit', user)"
      />
    </template>
  </b-card>
</template>

<script>
import { NoID } from '@cortezaproject/corteza-js'
import { handle } from '@cortezaproject/corteza-vue'
import ConfirmationToggle from 'corteza-webapp-admin/src/components/ConfirmationToggle'
import { getSystemFields } from 'corteza-webapp-admin/src/lib/sysFields'

export default {
  name: 'CUserEditorInfo',

  i18nOptions: {
    namespaces: 'system.users',
    keyPrefix: 'editor.info',
  },

  components: {
    ConfirmationToggle,
  },

  props: {
    user: {
      type: Object,
      required: true,
    },

    processing: {
      type: Boolean,
      value: false,
    },

    success: {
      type: Boolean,
      value: false,
    },

    canCreate: {
      type: Boolean,
      required: true,
    },
  },

  computed: {
    getDeleteStatus () {
      return this.user.deletedAt ? this.$t('undelete') : this.$t('delete')
    },

    getSuspendStatus () {
      return this.user.suspendedAt ? this.$t('unsuspend') : this.$t('suspend')
    },

    userID () {
      if (this.$auth.user) {
        return this.$auth.user.userID
      }
      return undefined
    },

    fresh () {
      return !this.user.userID || this.user.userID === NoID
    },

    editable () {
      return this.fresh ? this.canCreate : this.user.canUpdateUser
    },

    emailState () {
      const { email } = this.user
      return email ? null : false
    },

    handleState () {
      return handle.handleState(this.user.handle)
    },

    saveDisabled () {
      return !this.editable || [this.emailState, this.handleState].includes(false)
    },

    deletedButtonStatusCypressId () {
      return `button-${this.getDeleteStatus.toLowerCase()}`
    },

    suspendButtonStatusCypressId () {
      return `button-${this.getSuspendStatus.toLowerCase()}`
    },

    systemFields () {
      return getSystemFields(this.role)
    },
  },
}
</script>
