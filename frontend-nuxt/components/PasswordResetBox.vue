<template>
  <b-form @submit.stop.prevent="onSubmit">
    <b-form-group id="group-email" label label-for="input-email" description>
      <b-form-input
        id="input-password"
        v-model="form.password"
        type="password"
        required
        placeholder="Enter password"
        :state="
          isBrowser && $v.form.password.$dirty ? !$v.form.password.$error : null
        "
        :class="{ 'border-danger': errorMessages }"
      ></b-form-input>

      <b-form-invalid-feedback id="input-password-invalid">
        Please enter new password.
      </b-form-invalid-feedback>
    </b-form-group>

    <b-form-group
      id="group-password-confirm"
      label
      label-for="input-password-confirm"
      description
    >
      <b-form-input
        id="input-password-confirm"
        v-model="form.passwordConfirm"
        type="password"
        required
        placeholder="Enter password again"
        :state="
          isBrowser && $v.form.passwordConfirm.$dirty
            ? !$v.form.passwordConfirm.$error
            : null
        "
        :class="{ 'border-danger': errorMessages }"
      ></b-form-input>

      <b-form-invalid-feedback id="input-password-confirm-invalid">
        Please enter your password again.
      </b-form-invalid-feedback>
    </b-form-group>

    <template v-if="successMessages || errorMessages">
      <b-row class="mb-2">
        <b-col v-if="successMessages" class="text-primary message-col">
          {{ successMessages }}
        </b-col>
        <b-col v-if="errorMessages" class="text-danger message-col">
          {{ errorMessages }}
        </b-col>
      </b-row>
    </template>

    <b-row>
      <b-col>
        <b-button
          type="submit"
          variant="primary"
          :disabled="(isBrowser && $v.form.$invalid) || loading"
        >
          <span v-if="loading" class="spinner spinner-white"></span>
          Change password
        </b-button>
      </b-col>
      <b-col class="text-right">
        <b-link :to="{ path: '/login' }">Back to login</b-link>
      </b-col>
    </b-row>
  </b-form>
</template>

<script>
import { mapState, mapGetters, mapActions } from 'vuex'
import { required, maxLength, sameAs } from 'vuelidate/lib/validators'

export default {
  name: 'PasswordResetBox',
  props: {
    authKey: { type: String, default: null }
  },
  asyncData() {
    return { isBrowser: process.browser }
  },
  data() {
    return {
      isBrowser: false,
      form: {
        password: '',
        passwordConfirm: ''
      }
    }
  },
  validations: {
    form: {
      password: {
        required,
        maxLength: maxLength(6)
      },
      passwordConfirm: {
        required,
        sameAsPassword: sameAs('password')
      }
    }
  },
  computed: {
    ...mapGetters('alert', ['errorMessages', 'successMessages']),
    ...mapState('auth', ['loading']),
    ...mapGetters('auth', ['isLoggedIn'])
  },
  mounted() {
    // this.isBrowser = process.browser

    if (this.isLoggedIn) {
      // Already logged in
      this.logout({ router: this.$router, slient: true })
    }
  },
  methods: {
    ...mapActions('auth', ['passwordReset', 'logout']),
    onSubmit() {
      this.$v.form.$touch()
      if (this.$v.form.$anyError) {
        return
      }

      // Form submit logic
      this.passwordReset({
        key: this.authKey,
        password: this.form.password,
        router: this.$router
      })
    }
  }
}
</script>
