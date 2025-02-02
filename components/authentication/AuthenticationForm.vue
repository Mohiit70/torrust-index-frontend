<template>
  <div class="px-2 lg:px-0" @click.self="close">
    <div class="px-6 py-6 text-themeText/50 rounded-2xl mx-auto w-auto max-w-md">
      <h2 class="text-2xl mb-4 font-semibold text-center text-themeText">
        {{ state === State.Signup ? "Sign up" : "Sign in" }}
      </h2>
      <form
        class="space-y-6"
        @submit.prevent="submit"
      >
        <div class="mt-1">
          <label for="username" class="px-2">Username</label>
          <input
            id="username"
            v-model="form.username"
            name="username"
            type="text"
            class="mt-1"
            required
          >
        </div>
        <template v-if="state === State.Signup && settings.email_on_signup !== Requirement.None">
          <div class="mt-1">
            <label for="email" class="px-2">Email address</label>
            <input
              id="email"
              v-model="form.email"
              name="email"
              type="text"
              autocomplete="email"
              class="mt-1"
              :required="settings.email_on_signup === Requirement.Required"
            >
          </div>
        </template>
        <div class="mt-1">
          <label for="password" class="px-2">Password</label>
          <input
            id="password"
            v-model="form.password"
            name="password"
            type="password"
            class="mt-1"
            :autocomplete="[state === State.Signup ? 'new-password' : 'current-password']"
            required
          >
        </div>
        <template v-if="state === State.Signup">
          <div class="mt-1">
            <label for="password" class="px-2">Confirm password</label>
            <input
              id="password-repeat"
              v-model="form.confirm_password"
              name="password-repeat"
              type="password"
              class="mt-1"
              autocomplete="new-password"
              required
            >
          </div>
        </template>
        <button
          type="submit"
          class="px-4 h-12 w-full bg-gradient-to-bl from-accent to-accent-dark hover:bg-tertiary text-themeText font-semibold rounded-2xl shadow-lg shadow-transparent hover:shadow-accent-dark/50 duration-1000"
        >
          <span>Sign {{ state === State.Signup ? 'up' : 'in' }}</span>
        </button>
      </form>
      <div class="relative mt-6">
        <div class="flex relative justify-center text-sm">
          <template v-if="state === State.Signup">
            <button class="px-2 font-semibold text-themeText/50 hover:text-themeText duration-200" @click="toggleState">
              Already have an account? Sign in
            </button>
          </template>
          <template v-else>
            <button class="px-2 font-semibold text-themeText/50 hover:text-themeText duration-200" @click="toggleState">
              Don't have an account? Sign up
            </button>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { Ref } from "vue";
import { Requirement } from "torrust-index-types-lib";
import { notify } from "notiwind-ts";
import { onMounted, ref, useAuthenticationModal, useRestApi, useRuntimeConfig, useSettings } from "#imports";

enum State {
  Login,
  Signup
}

type Form = {
  username: string,
  email: string,
  password: string,
  confirm_password: string
}

const config = useRuntimeConfig();
const rest = useRestApi();
const authModalOpen = useAuthenticationModal();
const settings = useSettings();

const state: Ref<State> = ref(State.Login);
const form: Ref<Form> = ref({
    username: "",
    email: "",
    password: "",
    confirm_password: ""
});

const props = defineProps({
    signup: Boolean
});

onMounted(() => {
    if (props.signup) {
        state.value = State.Signup;
    }
});

function toggleState () {
    switch (state.value) {
    case State.Login:
        state.value = State.Signup;
        break;
    case State.Signup:
        state.value = State.Login;
        break;
    }
}

function close () {
    authModalOpen.value = false;
}

function submit () {
    switch (state.value) {
    case State.Login:
        login();
        break;
    case State.Signup:
        signup();
        break;
    }
}

function login () {
    loginUser(form.value.username, form.value.password)
        .then(() => {
            notify({
                group: "foo",
                title: "Success",
                text: "You were logged in!"
            }, 4000);

            authModalOpen.value = false;
        })
        .catch((err) => {
            notify({
                group: "foo",
                title: "Error",
                text: err.response.data.error
            }, 4000); // 4s
        });
}

function signup () {
    rest.value.user.registerUser({
        username: form.value.username,
        email: form.value.email,
        password: form.value.password,
        confirm_password: form.value.confirm_password
    })
        .then(() => {
            notify({
                group: "foo",
                title: "Success",
                text: "Your account was registered!"
            }, 4000); // 4s
        })
        .catch((err) => {
            notify({
                group: "foo",
                title: "Error",
                text: err.response.data.error
            }, 4000); // 4s
        });
}
</script>

<style scoped>
input {
  @apply p-2.5 w-full bg-secondary/25 text-themeText placeholder-themeText/50 border-2 border-secondary hover:border-tertiary rounded-2xl duration-200 cursor-pointer;
}
</style>
