<template>
<div>
  <v-row align="start" justify="end">
    <v-col cols="6" class="text-right">
      <v-btn-toggle
        v-if="moveDistance"
        mandatory
        dense
        v-model="moveDistance"
        class="ml-2 d-inline-block"
      >
        <app-btn
          v-for="(value, i) in zAdjustValues"
          small
          class="px-1"
          :key="i"
          :disabled="!klippyReady"
          :min-width="36"
          :value="value"
        >
          {{ value }}
        </app-btn>
      </v-btn-toggle>
      <div class="mt-1">
        <span class="secondary--text">{{ $t('app.general.label.z_offset') }}&nbsp;</span>
        <span>{{ ZHomingOrigin }}mm</span>
      </div>
    </v-col>
    <v-col cols="6">
      <app-btn
        @click="sendZAdjustGcode('+', moveDistance, waits.onZAdjust)"
        :loading="hasWait('ZAdjust')"
        :disabled="!klippyReady"
        small>
        <v-icon small class="mx-0">$zUp</v-icon>
      </app-btn>
      <app-btn
        @click="sendZAdjustGcode('-', moveDistance, waits.onZAdjust)"
        :loading="hasWait('ZAdjust')"
        :disabled="!klippyReady"
        small
        class="ml-1">
        <v-icon small>$zDown</v-icon>
      </app-btn>
      <app-btn
        @click="handleZApplyDialog"
        :loading="hasWait('ZApply')"
        :disabled="!klippyReady || printerPrinting || (ZHomingOrigin == 0)"
        small
        class="ml-3">
        Apply
      </app-btn>
    </v-col>
  </v-row>
      <z-apply-dialog
        v-if="ZApplyDialogState.open"
        v-model="ZApplyDialogState.open"
      ></z-apply-dialog>

    </div>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import StateMixin from '@/mixins/state'
import { Waits } from '@/globals'
import ZApplyDialog from './ZApplyDialog.vue'

@Component({
  components: {
    ZApplyDialog
  }
})
export default class ZHeightAdjust extends Mixins(StateMixin) {
  waits = Waits
  moveDistance: number | null = null

  ZApplyDialogState: any = {
    open: false
  }

  get ZHomingOrigin () {
    // This is an array of 4 values, representing the homing origin.
    // It should be in the order of; X, Y, Z, E.
    if (
      this.$store.state.printer.printer.gcode_move.homing_origin &&
      this.$store.state.printer.printer.gcode_move.homing_origin.length >= 4
    ) {
      const origin = this.$store.state.printer.printer.gcode_move.homing_origin[2]
      return origin.toFixed(3)
    } else {
      return null
    }
  }

  get zAdjustValues () {
    return this.$store.state.config.uiSettings.general.zAdjustDistances
  }

  mounted () {
    this.moveDistance = this.zAdjustValues[0]
  }

  /**
   * Send a Z adjust gcode script.
   */
  sendZAdjustGcode (direction: '+' | '-', distance: string, wait?: string) {
    const zHomed = this.$store.getters['printer/getHomedAxes']('z')
    const gcode = `SET_GCODE_OFFSET Z_ADJUST=${direction}${distance} MOVE=${+zHomed}`
    this.sendGcode(gcode, wait)
  }

  get printerSettings () {
    return this.$store.getters['printer/getPrinterSettings']()
  }

  get printerHasProbe (): boolean {
    return 'probe' in this.printerSettings || 'bltouch' in this.printerSettings
  }

  handleZApplyDialog0 () {
    if (this.printerHasProbe) {
      this.sendGcode('Z_OFFSET_APPLY_PROBE', this.waits.onZApply)
    } else {
      this.sendGcode('Z_OFFSET_APPLY_ENDSTOP', this.waits.onZApply)
    }
    this.ZApplyDialogState = {
      open: true,
      handler: this.handleSaveConfig
    }
  }

  handleSaveConfig () {
    this.sendGcode('test', this.waits.onZApply)
  }

  handleZApplyDialog () {
    if (this.printerHasProbe) {
      this.sendGcode('Z_OFFSET_APPLY_PROBE', this.waits.onZApply)
    } else {
      this.sendGcode('Z_OFFSET_APPLY_ENDSTOP', this.waits.onZApply)
    }
    this.$confirm(
      this.$tc('app.general.simple_form.msg.apply_z_offset'),
      {
        title: this.$tc('app.general.label.apply_z_offset'),
        color: 'card-heading',
        icon: '$error',
        buttonTrueText: this.$tc('app.general.btn.save_restart'),
        buttonTrueColor: 'primary',
        buttonFalseText: this.$tc('app.general.btn.later'),
        buttonFalseColor: 'warning'
      }
    )
      .then(res => {
        if (res) {
          this.sendGcode('test', this.waits.onZApply)
        }
      })
  }
}
</script>
export interface VuetifyConfirmObject {
  buttonTrueText?: string
  buttonFalseText?: string
  buttonTrueColor?: string
  buttonFalseColor?: string
  color?: string
  icon?: string
  title?: string
  width? : number
}
