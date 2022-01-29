<template>
  <v-dialog
    :value="value"
    @input="$emit('input', $event)"
    :max-width="500"
  >
    <v-card>
      <v-card-title class="card-heading py-2">
        <span class="focus--text" v-html="$t('app.general.label.apply-z-offset')"></span>
      </v-card-title>

      <v-divider></v-divider>

      <v-card-text class="px-4 pt-4">
        The new Z-Offset value was calculate. Do you want to save in the it now and restart klipper.
      </v-card-text>

      <v-divider />

      <v-card-actions class="pa-4">
        <v-spacer></v-spacer>
        <app-btn color="warning" text @click="$emit('input', false)" type="button">{{ $t('app.general.btn.later') }}</app-btn>
        <app-btn color="primary" text @click="handleSaveConfig" type="button">{{ $t('app.general.btn.save') }}</app-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator'

@Component({})
export default class ZApplyDialog extends Vue {
  @Prop({ type: Boolean, required: true })
  value!: boolean

  handleSaveConfig () {
    return true
  }
}
</script>
