<template>
    <h1 class="text-center">Log in to your account</h1>
    <form
      @submit.prevent="login"
      class="mt-7 w-4/5 m-auto p-10 border border-slate-700 rounded-md flex flex-col"
    >
      <label class="block text-sm md:text-base md:pb-2" for="Email"
        >Enter email address</label
      >
      <input
        class="bg-slate-300 md:bg-slate-200 w-full p-3 mt-1 rounded-lg outline-transparent"
        type="email"
        placeholder="youremail@example.com"
        v-model="email"
      />
      <span class="text-sm ml-1 mt-1 text-red-600"></span>
      <label class="block text-sm md:text-base md:pb-1 mt-3" for="password"
        >Password</label
      >
      <input
        class="bg-slate-300 md:bg-slate-200 text-sm w-full p-3 mt-1 rounded-lg outline-transparent"
        type="password"
        placeholder="Enter password here"
        v-model="password"
      />
      <span class="text-sm ml-1 mt-1 text-red-600">{{ errorMsg }}</span>
      <button
        type="submit"
        class="px-4 py-2 text-white bg-indigo-600 rounded-md mt-7 m-auto"
      >
        Log in
      </button>
      <NuxtLink to="/index.vue"><p  class="hover:underline">register</p></NuxtLink>
    </form>
  </template>
<script setup>
const router = useRouter()
const client = useSupabaseClient()
const email = ref('')
const password = ref(null)
const errorMsg = ref(null)

async function login() {
    try {
        const {error} = await client.auth.signInWithPassword({
            email: email.value,
            password: password.value
        })
        if (error) throw error
        router.push("/")
    } catch (error) {
        errorMsg.value = error.message
    }
}
</script>

