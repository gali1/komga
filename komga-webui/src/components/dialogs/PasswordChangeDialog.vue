<template>
  <div>
    <v-dialog v-model="modal"
              max-width="450"
    >
      <v-card>
        <v-card-title>{{ $t('dialog.password_change.dialog_title') }}</v-card-title>

        <v-card-text>
          <form novalidate>
            <v-container fluid>
              <v-row>
                <v-col>
                  <v-text-field :label="$t('dialog.password_change.field_new_password')"
                                v-model="form.newPassword"
                                autocomplete="off"
                                :type="showPassword1 ? 'text' : 'password'"
                                :append-icon="showPassword1 ? 'mdi-eye' : 'mdi-eye-off'"
                                @click:append="showPassword1 = !showPassword1"
                                :error-messages="getErrors('newPassword')"
                                @input="$v.form.newPassword.$touch()"
                                @blur="$v.form.newPassword.$touch()"
                  />
                </v-col>
              </v-row>

              <v-row>
                <v-col>
                  <v-text-field :label="$t('dialog.password_change.field_repeat_password')"
                                v-model="form.repeatPassword"
                                autocomplete="off"
                                :type="showPassword2 ? 'text' : 'password'"
                                :append-icon="showPassword2 ? 'mdi-eye' : 'mdi-eye-off'"
                                @click:append="showPassword2 = !showPassword2"
                                :error-messages="getErrors('repeatPassword')"
                                @input="$v.form.repeatPassword.$touch()"
                                @blur="$v.form.repeatPassword.$touch()"
                                @keydown.enter="dialogConfirm"
                  />
                </v-col>
              </v-row>
            </v-container>
          </form>
        </v-card-text>

        <v-card-actions>
          <v-spacer/>
          <v-btn text @click="dialogCancel">{{ $t('dialog.password_change.button_cancel') }}</v-btn>
          <v-btn text class="primary--text"
                 @click="dialogConfirm"
          >{{ $t('dialog.password_change.button_confirm') }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-snackbar
      v-model="snackbar"
      bottom
      color="error"
    >
      {{ snackText }}
      <v-btn
        text
        @click="snackbar = false"
      >{{ $t('common.close') }}</v-btn>
    </v-snackbar>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import {required, sameAs} from 'vuelidate/lib/validators'

export default Vue.extend({
  name: 'PasswordChangeDialog',
  data: () => {
    return {
      modal: false,
      showPassword1: false,
      showPassword2: false,
      snackbar: false,
      snackText: '',
      form: {
        newPassword: '',
        repeatPassword: '',
      },
      validationFieldNames: new Map([
        ['newPassword', 'Password'],
        ['confirmPassword', 'Password confirmation'],
      ]),
    }
  },
  props: {
    value: Boolean,
    user: {
      type: Object,
      required: true,
    },
  },
  watch: {
    value (val) {
      this.modal = val
    },
    modal (val) {
      !val && this.dialogCancel()
    },
  },
  validations: {
    form: {
      newPassword: { required },
      repeatPassword: { sameAsPassword: sameAs('newPassword') },
    },
  },
  methods: {
    getErrors (fieldName: string): string[] {
      const errors = [] as string[]
      const field = this.$v.form!![fieldName] as any
      if (field && field.$invalid && field.$dirty) {
        if (fieldName === 'newPassword') {
          if (!field.required) errors.push(this.$t('dialog.password_change.field_new_password_error').toString())
        }
        if (fieldName === 'repeatPassword') {
          if (!field.sameAsPassword) errors.push(this.$t('dialog.password_change.field_repeat_password_error').toString())
        }
      }
      return errors
    },
    showSnack (message: string) {
      this.snackText = message
      this.snackbar = true
    },
    formReset () {
      this.form.newPassword = ''
      this.form.repeatPassword = ''
      this.showPassword1 = false
      this.showPassword2 = false
      this.$v.$reset()
    },
    dialogCancel () {
      this.$emit('input', false)
      this.formReset()
    },
    dialogConfirm () {
      this.$v.$touch()

      if (!this.$v.$invalid) {
        this.updatePassword()
        this.$emit('input', false)
        this.formReset()
      }
    },
    async updatePassword () {
      try {
        await this.$store.dispatch('updateMyPassword', { password: this.form.newPassword })
      } catch (e) {
        this.showSnack(e.message)
      }
    },
  },
})
</script>

<style scoped>

</style>
