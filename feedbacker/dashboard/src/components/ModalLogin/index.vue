<template>
  <div class="flex justify-between">
    <h1 class="font-black text-4xl text-gray-800">
      Entre na sua conta
    </h1>
    
    <button
    @click="close" 
      class="text-4xl text-gray-600 focus:outline-none"
    >
      &times;
    </button>

  </div>

  <div class="mt-16">
    <form @submit.prevent="handleSubmit">
      <label class="block">
        <span class="text-lg font-medium text-gray-800">E-mail</span>
        <input 
          type="email"
          v-model="state.email.value"
          :class="{
            'border-brand-danger': !!state.email.errorMessage
          }"
          class="block w-full px-4 py-3 mt-1 text-lg bg-gray-100 border-2 border-transparent rounded"
          placeholder="jane.dae@gmail.com"
        >
        <span
          v-if="!!state.email.errorMessage"
          class="block font-medium text-brand-danger"
        >
          {{ state.email.errorMessage }}
        </span>
      </label>

      <label class="block mt-7">
        <span class="text-lg font-medium text-gray-800">Password</span>
        <input 
          type="password"
          v-model="state.password.value"
          :class="{
            'border-brand-danger': !!state.password.errorMessage
          }"
          class="block w-full px-4 py-3 mt-1 text-lg bg-gray-100 border-2 border-transparent rounded"
          placeholder="********"
        >
        <span
          v-if="!!state.password.errorMessage"
          class="block font-medium text-brand-danger"
        >
          {{ state.password.errorMessage }}
        </span>
      </label>

      <button
        type="submit"
        :disabled="state.isLoading"
        :class="{
          'opacity-50': state.isLoading
        }"
        class="bg-brand-main text-white font-bold py-3 px-14 border-transparent rounded-full mt-10 focus:outline-none transition-all duration-150"
      >
      <icon v-if="state.isLoading" name="loading" class="animate-spin" />
      <span v-else>Entrar</span>
      </button>

    </form>
  </div>
</template>

<script>
import { reactive } from 'vue'
import { useField } from 'vee-validate'
import useModal from '../../hooks/useModal'
import { validateEmptyOrLength3, validateEmptyAndEmail } from '../../../utils/validators'
import services from '../../services/'
import { useRouter } from 'vue-router'
import { useToast } from 'vue-toastification'
import Icon from '../Icons/'

export default {
    components: { Icon },
    setup () {
      const modal = useModal()
      const router = useRouter();
      const toast = useToast();
      const {
        value: emailValue,
        errorMessage: emailErrorMessage
      } = useField('email', validateEmptyAndEmail);

      const {
        value: passwordValue,
        errorMessage: passwordErrorMessage
      } = useField('password', validateEmptyOrLength3);

      const state = reactive({
        hasErrors: false,
        isLoading: false,
        email: {
          value: emailValue,
          errorMessage: emailErrorMessage
        },
        password: {
          value: passwordValue,
          errorMessage: passwordErrorMessage
        }
      })
    
    async function handleSubmit () {
      try {
        state.isLoading = true
        const { data, errors } = await services.auth.login(
        {
          email: state.email.value, 
          password: state.password.value
        })

        if (!errors) {
          state.isLoading = false
          window.localStorage.setItem('token', data.token)
          router.push({name: 'Feedbacks'})
          modal.close()
          return;
        }

        if (errors.status === 404) {
          toast.error('E-mail não encontrado!')
        }

        if (errors.status === 401) {
          toast.error('E-mail/Senha inválidos!')
        }

        if (errors.status === 400) {
          toast.error('Ocorreu um erro ao fazer o login!')
        }

        state.isLoading = false

      } catch (error) {
        state.isLoading = false
        state.hasErrors = !!error
        toast.error('Ocorreu um erro ao fazer o login!')
      }
    }

    return {
      state,
      handleSubmit,
      close: modal.close
    }

  }
}
</script>