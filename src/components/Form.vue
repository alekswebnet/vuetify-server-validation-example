<template>
  <form class="pt-6" @submit.prevent="submit">
    <legend class="text-h4 mb-6">Register form</legend>

    <!-- Email field -->
    <v-text-field
      v-model="email.value.value"
      class="me-10 mb-2"
      :error-messages="email.errorMessage.value"
      label="E-mail"
    ></v-text-field>

    <!-- Password field -->
    <v-text-field
      v-model="password.value.value"
      :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
      :type="showPassword ? 'text' : 'password'"
      class="mb-4"
      label="Password"
      counter
      :error-messages="password.errorMessage.value"
      @click:append="showPassword = !showPassword"
    ></v-text-field>

    <v-btn
      class="me-4"
      type="submit"
    >
      submit
    </v-btn>

    <v-btn @click="handleReset">
      clear
    </v-btn>
  </form>
</template>

<script>
  import { ref } from 'vue'
  import { useField, useForm } from 'vee-validate'
  import { useFetch } from '@vueuse/core'

  export default {
    setup () {
      const { handleSubmit, handleReset, setErrors } = useForm({
        validationSchema: {
          email (value) {
            if (/^[a-z.-]+@[a-z.-]+\.[a-z]+$/i.test(value)) return true

            return 'Must be a valid e-mail.'
          },
          password (value) {
            if (value?.length >= 6) return true

            return 'Password length needs to be at least 6.'
          }
        },
      })
      const email = useField('email')
      const password = useField('password')
      const showPassword = ref(false)

      const submit = handleSubmit(async values => {
        const body = JSON.stringify({ 
          email: values.email, 
          password: values.password 
        })
        const { data } = await useFetch('https://reqres.in/api/register', {
          method: 'POST',
          headers: { 
            'Content-Type': 'application/json' 
          },
          body
        }).json()
        
        if (data.value.error) {
          const isPasswordError = data.value.error.includes('password')
          return setErrors({
            email: !isPasswordError && data.value.error,
            password: isPasswordError && data.value.error
          })
        }
        alert(JSON.stringify(data.value))
      })

      return { email, password, showPassword, submit, handleReset }
    },
  }
</script>