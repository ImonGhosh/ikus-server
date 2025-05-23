<template>
  <GenericDialog v-model="value" :title="dialogTitle" persistent>
    <template v-slot:content>
      <v-row>
        <v-col cols="6">
          <v-select
              label="Gruppe"
              v-model="channel"
              :items="channels" item-title="name.en" item-value="id"
              filled
              hide-details
              return-object
          ></v-select>
        </v-col>
        <v-col cols="6"></v-col>
        <v-col cols="6">
          <v-text-field v-model="infoEn" label="Beschreibung (EN)" :disabled="loading" filled hide-details />
        </v-col>
        <v-col cols="6">
          <v-text-field v-model="infoDe" label="Beschreibung (DE)" :disabled="loading" filled hide-details />
        </v-col>
        <v-col cols="12">
          <v-text-field v-model="urlEn" :label="urlEnLabel" placeholder="https://example.com" :disabled="loading" filled hide-details />
        </v-col>
        <v-col cols="12">
          <v-checkbox v-model="sameUrl" label="deutsche Seite = englische Seite" :disabled="loading" hide-details />
        </v-col>
        <v-col cols="12">
          <v-text-field v-show="!sameUrl" v-model="urlDe" label="Adresse (DE)" placeholder="https://example.com" filled :disabled="loading || sameUrl" hide-details />
        </v-col>
      </v-row>
    </template>

    <template v-slot:actions>
      <v-btn @click="value = false" color="black" :disabled="loading">
        Abbrechen
      </v-btn>
      <v-btn @click="submit" color="primary" :loading="loading" :prepend-icon="submitIcon" variant="elevated">
        {{ submitText }}
      </v-btn>
    </template>
  </GenericDialog>
</template>

<script lang="ts">
import GenericDialog from "@/components/dialog/GenericDialog.vue";
import {showSnackbar} from "~/utils/snackbar";
import type {ChannelDto, CreateLinkDto, LinkDto} from "~/models";

export default {
  name: 'LinkDialog',
  components: {GenericDialog},
  props: {
    modelValue: {
      type: Boolean,
      required: true
    },
    updating: {
      type: Boolean,
      required: true
    },
    loading: {
      type: Boolean,
      required: true
    },
    channels: {
      type: Array,
      required: true
    }
  },
  data: () => ({
    channel: {} as ChannelDto,
    infoEn: '',
    infoDe: '',
    urlEn: '',
    urlDe: '',
    sameUrl: true
  }),
  methods: {
    reset: function(channel: ChannelDto) {
      // reset input
      this.infoEn = '';
      this.infoDe = '';
      this.urlEn = '';
      this.urlDe = '';
      this.sameUrl = true;

      // apply predefined group
      this.channel = channel;
    },
    load: function(link: LinkDto) {
      // apply
      this.infoEn = link.info.en;
      this.infoDe = link.info.de;
      this.urlEn = link.url.en;
      this.urlDe = link.url.de;
      this.sameUrl = link.url.en === link.url.de;
    },
    submit: async function() {

      if (!this.infoEn || !this.infoDe) {
        showSnackbar('Bitte Beschreibungen eingeben');
        return;
      }

      if ((this.sameUrl && !this.urlEn) || (!this.sameUrl && (!this.urlEn || !this.urlDe))) {
        showSnackbar('Bitte Internet-Adresse eingeben');
        return;
      }

      if (!this.channel.id) {
        showSnackbar('Bitte Gruppe festlegen');
        return;
      }

      const url = this.sameUrl ? { en: this.urlEn, de: this.urlEn } : { en: this.urlEn, de: this.urlDe };

      if (!url.en.startsWith('http://') && !url.en.startsWith('https://')) {
        showSnackbar('Seite muss mit http:// oder https:// beginnen');
        return;
      }

      if (!url.de.startsWith('http://') && !url.de.startsWith('https://')) {
        showSnackbar('Seite muss mit http:// oder https:// beginnen');
        return;
      }

      this.$emit('submit', {
        channelId: this.channel.id,
        info: { en: this.infoEn, de: this.infoDe },
        url,
      } as CreateLinkDto);
    }
  },
  computed: {
    value: {
      get() {
        return this.modelValue
      },
      set(value: boolean) {
        this.$emit('update:modelValue', value)
      }
    },
    dialogTitle: function() {
      return this.updating ? 'Link bearbeiten' : 'Neuer Link';
    },
    urlEnLabel: function() {
      return this.sameUrl ? 'Adresse' : 'Adresse (EN)';
    },
    submitText: function() {
      return this.updating ? 'Speichern' : 'Erstellen';
    },
    submitIcon: function() {
      return this.updating ? 'mdi-content-save' : 'mdi-plus';
    }
  },
  watch: {
    urlEn: function(newVal) {
      if (this.sameUrl) {
        this.urlDe = newVal;
      }
    },
    sameUrl: function(newVal) {
      if (newVal) {
        this.urlDe = this.urlEn;
      }
    }
  }
}
</script>
