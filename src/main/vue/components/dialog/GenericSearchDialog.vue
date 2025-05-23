<template>
  <GenericDialog v-model="value" :title="dialogTitle" :width="550">
    <template v-slot:content>

      <v-text-field v-model="query" label="Suchen" :disabled="loading" filled hide-details
                    @keydown.enter="search" @click:append="search" append-icon="mdi-magnify"></v-text-field>

      <br>

      <LoadingIndicator v-if="loading" />
      <Notice v-if="!loading && items.length === 0" class="mt-6 mb-6" title="Keine Ergebnisse." />

      <template v-if="!loading">
        <div v-for="(item, index) in items" :key="index">
          <v-card class="bg-secondary mb-4">
            <v-card-text>
              <div style="display: flex; align-items: center;">
                <div style="flex: 1">
                  <slot :item="item"></slot>
                </div>
                <v-btn @click="$emit('select', item)" icon="mdi-check-circle" variant="text" />
              </div>
            </v-card-text>
          </v-card>
        </div>
      </template>

    </template>

    <template v-slot:actions>
      <v-btn @click="value = false" color="black" :disabled="loading" variant="text">
        Schließen
      </v-btn>
    </template>
  </GenericDialog>
</template>

<script>
import GenericDialog from "@/components/dialog/GenericDialog";
import Notice from "@/components/Notice";
import LoadingIndicator from "@/components/LoadingIndicator";
import {showSnackbar} from "~/utils/snackbar";

export default {
  name: 'GenericSearchDialog',
  components: {LoadingIndicator, Notice, GenericDialog},
  props: {
    modelValue: {
      type: Boolean,
      required: true
    },
    dialogTitle: {
      type: String,
      required: true
    },
    searchFunc: {
      type: Function,
      required: true
    }
  },
  data: () => ({
    loading: false,
    query: '',
    items: []
  }),
  methods: {
    reset: function() {
      // reset input
      this.query = '';
      this.items = [];
    },
    search: async function() {
      try {
        this.loading = true;
        this.items = await this.searchFunc({ query: this.query });
      } catch (e) {
        showSnackbar('Ein Fehler ist aufgetreten');
      } finally {
        this.loading = false;
      }
    }
  },
  computed: {
    value: {
      get() {
        return this.modelValue
      },
      set(value) {
        this.$emit('update:modelValue', value)
      }
    },
  },
}
</script>
