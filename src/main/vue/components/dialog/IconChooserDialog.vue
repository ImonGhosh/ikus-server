<template>
  <GenericDialog v-model="value" title="Icon auswählen" :width="550">
    <template v-slot:content>

      <span>
        Alle Icons finden Sie auch unter
        <br>
        <a href="https://material.io/resources/icons/?style=baseline" target="_blank">https://material.io/resources/icons/?style=baseline</a>
        <br>
        Lizensiert unter <a href="https://www.apache.org/licenses/LICENSE-2.0.html" target="_blank">Apache License, Version 2.0</a>
      </span>

      <v-text-field class="mt-4" v-model="query" label="Suchen" :disabled="loading" filled hide-details
                    @keydown.enter="search" @click:append="search" append-inner-icon="mdi-magnify"></v-text-field>

      <br>

      <Notice v-if="icons.length === 0" class="mt-6 mb-6" title="Keine Ergebnisse." />

      <v-card v-if="icons.length !== 0" class="bg-secondary">
        <v-card-text class="pl-2 pt-6">
          <v-virtual-scroll :items="icons" height="350" item-height="100">
            <template v-slot:default="{ item }">
              <div style="display: flex;" class="py-2">
                <div style="text-align: center; width: 110px;" v-for="(icon, index) in item" :key="index">
                  <m-icon @click="$emit('select', icon)" :icon="icon" style="cursor: pointer; font-size: 40px;" />
                  <div @click="$emit('select', icon)" style="cursor: pointer;">{{ icon }}</div>
                </div>
              </div>
            </template>
          </v-virtual-scroll>
        </v-card-text>
      </v-card>
    </template>

    <template v-slot:actions>
      <v-btn @click="value = false" color="black" :disabled="loading" variant="text">
        Schließen
      </v-btn>
    </template>
  </GenericDialog>
</template>

<script>
import allIcons from "@/assets/material-icons.json";
import GenericDialog from "@/components/dialog/GenericDialog";
import Notice from "@/components/Notice";
import MIcon from "@/components/MIcon";
import {showSnackbar} from "~/utils/snackbar";

export default {
  name: 'IconChooserDialog',
  components: {MIcon, Notice, GenericDialog},
  props: {
    modelValue: {
      type: Boolean,
      required: true
    },
  },
  data: () => ({
    loading: false,
    query: '',
    icons: []
  }),
  methods: {
    reset: function() {
      // reset input
      this.query = '';
      this.icons = [];
    },
    search: async function() {
      try {
        this.loading = true;
        const query = this.query.toLowerCase();
        this.icons = this.groupN(allIcons.filter(name => name.includes(query)));
      } catch (e) {
        showSnackbar('Ein Fehler ist aufgetreten');
      } finally {
        this.loading = false;
      }
    },
    groupN: function(items) {

      // create sub lists of 4 items
      // e.g. [1, 2, 3, 4, 5, 6] -> [ [1, 2, 3, 4], [5, 6] ]

      const result = [];
      for(let i = 0; i + 3 < items.length; i += 4) {
        result.push([items[i], items[i+1], items[i+2], items[i+3]])
      }

      if (items.length % 4 !== 0) {
        const lastRow = [];
        for(let i = items.length - items.length % 4; i < items.length; i++) {
          lastRow.push(items[i]);
        }
        result.push(lastRow);
      }

      return result;
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
  watch: {
    value: {
      immediate: false,
      handler: function(val) {
        if (val) {
          // virtual scroller somehow needs to be rerendered
          // otherwise it stays blank at 2nd dialog opening
          this.$nextTick(() => {
            this.icons = this.groupN(allIcons);
          });
        }
      }
    }
  }
}
</script>