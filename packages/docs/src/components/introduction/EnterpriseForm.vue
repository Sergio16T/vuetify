<template>
  <v-defaults-provider
    :defaults="{
      VCheckboxBtn: {
        color: 'primary',
        density: 'compact',
      },
      VTextField: {
        color: 'primary',
      },
      VTextarea: {
        color: 'primary',
      },
    }"
  >
    <v-card
      id="request-service"
      border
      class="mx-auto pa-2"
      max-width="440"
      title="Request Support"
      variant="flat"
    >
      <template #append>
        <v-img
          width="96"
          src="https://cdn.vuetifyjs.com/docs/images/logos/vuetify-logo-v3-slim-text-light.svg"
        />
      </template>

      <v-form
        ref="form"
        v-model="valid"
        @submit.prevent="onSubmit"
      >
        <v-card-text>
          <v-text-field
            v-model="name"
            :rules="[rules.required]"
            label="Name"
            name="name"
          />

          <v-text-field
            v-model="email"
            :rules="[rules.required, rules.email]"
            label="Email address"
            name="email"
          />

          <v-label>What services are you interested in?</v-label>

          <div class="py-2">
            <v-checkbox-btn
              v-model="upgrade"
              label="Upgrading an existing project"
              name="upgrade"
            />

            <v-checkbox-btn
              v-model="review"
              :false-value="false"
              label="Application performance review"
              name="review"
            />

            <v-checkbox-btn
              v-model="sla"
              label="Direct support or SLA"
              name="sla"
            />

          <!-- <v-checkbox label="Training & workshops" /> -->
          </div>

          <v-textarea
            v-model="questions"
            label="Questions and comments"
            name="questions"
          />

          <small class="text-medium-emphasis">
            *All service packages are custom per client with pricing based upon the requested services.
          </small>
        </v-card-text>

        <v-card-actions class="px-4 pb-4">
          <v-spacer />

          <v-btn
            :append-icon="!loading && success ? '$success' : undefined"
            :color="success ? 'success' : valid ? 'primary' : undefined"
            :disabled="loading || !valid"
            :loading="loading"
            block
            size="large"
            type="submit"
            variant="flat"
          >
            <span v-if="!success && !loading">Submit</span>

            <span v-else-if="!loading">Successful</span>
          </v-btn>
        </v-card-actions>
      </v-form>
    </v-card>
  </v-defaults-provider>
</template>

<script setup lang="ts">
  // Utilities
  import emailjs from '@emailjs/browser'
  import { ref, watch } from 'vue'

  const name = ref('')
  const email = ref('')
  const upgrade = ref(false)
  const review = ref(false)
  const sla = ref(false)
  const loading = ref(false)
  const questions = ref('')
  const valid = ref<boolean | null>(null)
  const success = ref(false)
  const form = ref<HTMLFormElement>()
  const rules = {
    required: (v: string) => !!v || 'Field is required',
    email: (v: any) => /.+@.+/.test(v) || 'E-mail must be valid',
  }

  watch(success, val => {
    setTimeout(() => {
      if (val) {
        form.value?.reset()

        // TODO: bug with resetting checkbox
        upgrade.value = false
        review.value = false
        sla.value = false
      }

      success.value = false
    }, 2000)
  })

  async function onSubmit () {
    loading.value = true

    try {
      const res = await emailjs.sendForm(
        import.meta.env.VITE_EMAILJS_SERVICE_ID,
        import.meta.env.VITE_EMAILJS_TEMPLATE_ID,
        form.value?.$el,
        import.meta.env.VITE_EMAILJS_PUBLIC_KEY,
      )

      success.value = res.text === 'OK'
    } catch (e) {
      console.log(e)
    }

    loading.value = false
  }
</script>