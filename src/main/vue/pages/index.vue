<template>
  <div class="bg-primary" style="min-height: 100vh; width: 100%; display: flex; flex-direction: column; align-items: center; justify-content: center">
    <div style="text-align: center">
      <img src="@/assets/logo-512-alpha.png" style="width: 200px" alt="logo">
      <p class="text-h4">Welcome to OVGU</p>
      <p class="text-h5">Administration</p>
      <v-card class="mt-10">
        <v-card-text>
          <v-text-field v-model="name" @keydown.enter="login" prepend-inner-icon="mdi-account" type="text"
                        placeholder="Name" :disabled="fetching" />
          <v-text-field v-model="password" @keydown.enter="login" prepend-inner-icon="mdi-key" type="password"
                        placeholder="Passwort" :disabled="fetching" hide-details/>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn @click="login" class="bg-success" :disabled="fetching" append-icon="mdi-arrow-right">
            Login
          </v-btn>
        </v-card-actions>
      </v-card>
    </div>

    <div id="version-container" class="white--text text-body-1">
      <span>Version {{ version }}</span>
      <br>
      <router-link to="/privacy" v-slot="{ href, navigate }" custom>
        <a @click="navigate" :href="href" class="text-white text-decoration-underline">Datenschutz</a>
      </router-link>
      <router-link to="/status" v-slot="{ href, navigate }" custom>
        <a @click="navigate" :href="href" class="text-white text-decoration-underline ml-2">Status</a>
      </router-link>
    </div>

  </div>
</template>

<script lang="ts">
import {getUserInfo, getVersion, login} from '@/api'
import {showSnackbar} from "~/utils/snackbar";

export default {
  name: 'Login',
  data: () => ({
    fetching: false,
    name: null as string | null,
    password: null as string | null,
    version: null as string | null
  }),
  methods: {
    login: async function() {
      if (!this.name || !this.password) {
        showSnackbar('Bitte Name und Passwort eingeben');
        return;
      }

      this.fetching = true;
      try {
        await login({ name: this.name, password: this.password });
        this.$emit('update-login');

        const user = getUserInfo()!;
        if (user.admin)
          await this.$router.push('/users');
        else
          await this.$router.push('/dashboard');
      } catch(status) {
        showSnackbar('Login fehlgeschlagen');
      } finally {
        this.fetching = false;
      }
    }
  },
  mounted: async function() {
    this.version = (await getVersion()).version;
  }
}
</script>

<style>
@media screen and (max-width: 600px) {
  #version-container {
    margin-top: 50px;
    margin-bottom: 50px;
    text-align: center;
  }
}

@media screen and (min-width: 601px) {
  #version-container {
    position: absolute;
    right: 10px;
    bottom: 10px;
    text-align: right;
  }
}
</style>
