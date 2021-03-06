<template>
  <v-dialog
    v-model="visible"
    persistent
    max-width="500px"
  >
    <form
      name="contact-speaker"
      method="POST"
      data-netlify="true"
      data-netlify-honeypot="bot-field"
      @submit.prevent="handleSubmit"
    >
      <input
        type="hidden"
        name="form-name"
        value="ask-question"
      >
      <two-button-modal
        :title="$t('contact.title', [name])"
        @cancel="$emit('close')"
      >
        {{ $t('contact.body', [name]) }}

        <v-text-field
          v-model="form.name"
          :label="$t('contact.yourName')"
          :error-messages="nameErrors"
          @input="$v.form.name.$touch()"
          @blur="$v.form.name.$touch()"
        />
        <v-text-field
          v-model="form.email"
          :label="$t('contact.email')"
          :error-messages="emailErrors"
          @input="$v.form.email.$touch()"
          @blur="$v.form.email.$touch()"
        />
        <v-textarea
          v-model="form.message"
          :label="$t('contact.message')"
          :error-messages="messageErrors"
          @input="$v.form.message.$touch()"
          @blur="$v.form.message.$touch()"
        />
        <v-alert
          v-if="invalid"
          color="error"
          dense
          text
          class="mb-0"
        >
          {{ $t('validations.allRequired') }}
        </v-alert>
      </two-button-modal>
    </form>
  </v-dialog>
</template>


<script>
import TwoButtonModal from '@/components/cards/TwoButtonModal.vue';
import axios from 'axios';
import { validationMixin } from 'vuelidate';
import { required, email } from 'vuelidate/lib/validators';

export default {
  components: {
    TwoButtonModal,
  },
  mixins: [validationMixin],
  props: {
    show: {
      type: Boolean,
      required: true,
    },
    speaker: {
      type: Object,
      default: () => ({}),
    },
  },
  validations: {
    form: {
      name: {
        required,
      },
      email: {
        required,
        email,
      },
      message: {
        required,
      },
    },
  },
  data: () => ({
    form: {
      name: '',
      email: '',
      message: '',
    },
  }),
  computed: {
    name() {
      try {
        return this.speaker.get('name_en');
      } catch (e) {
        return '';
      }
    },
    visible() {
      return this.show;
    },
    invalid() {
      return this.$v.$error;
    },
    nameErrors() {
      const errors = [];
      if (!this.$v.form.name.$dirty) return errors;
      if (!this.$v.form.name.required) { errors.push(this.$t('validations.fieldRequired', [this.$t('contact.name')])); }
      return errors;
    },
    emailErrors() {
      const errors = [];
      if (!this.$v.form.email.$dirty) { return errors; }
      if (!this.$v.form.email.email) { errors.push(this.$t('validations.validEmail')); }
      if (!this.$v.form.email.required) { errors.push(this.$t('validations.fieldRequired', [this.$t('contact.email')])); }
      return errors;
    },
    messageErrors() {
      const errors = [];
      if (!this.$v.form.message.$dirty) { return errors; }
      if (!this.$v.form.message.required) { errors.push(this.$t('validations.fieldRequired', [this.$t('contact.message')])); }
      return errors;
    },
  },
  methods: {
    resetForm() {
      this.$set(this.form, 'name', '');
      this.$set(this.form, 'email', '');
      this.$set(this.form, 'message', '');
      this.$v.$reset();
    },
    encode(data) {
      return Object.keys(data)
        .map(
          (key) => `${encodeURIComponent(key)}=${encodeURIComponent(data[key])}`,
        )
        .join('&');
    },
    handleSubmit() {
      this.$v.$touch();
      if (this.$v.$invalid) {
        return;
      }
      const axiosConfig = {
        header: { 'Content-Type': 'application/x-www-form-urlencoded' },
      };
      this.form.speaker = this.speaker.name;
      axios.post(
        '/',
        this.encode({
          'form-name': 'contact-speaker',
          ...this.form,
        }),
        axiosConfig,
      );
      this.resetForm();
      this.$emit('submit');
    },
  },
};
</script>
