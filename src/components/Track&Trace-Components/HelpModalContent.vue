<template>
  <b-container>
    <b-table :fields="colorFields" :items="colorItems" borderless small>
      <template v-slot:cell(color)="data">
        <font-awesome-icon :class="data.value" icon="square"></font-awesome-icon>
      </template>
    </b-table>
    <b-table :fields="iconFields" :items="iconItems" borderless small>
      <template v-slot:cell(status)="data">
        <div class="d-flex justify-content-left">
          <font-awesome-icon :class="data.value.color" :icon="data.value.icon"></font-awesome-icon>
        </div>
      </template>
    </b-table>
    <b-table :fields="stepFields" :items="stepItems" borderless small></b-table>
  </b-container>
</template>

<script lang="ts">
import Vue from 'vue'

declare module 'vue/types/vue' {
  interface Vue {
    iconFields: Array<string | {key: string; label: string}>;
    iconItems: {name: string; status: {icon: string; color: string}; description: string}[];
    colorFields: Array<string | {key: string; label: string}>;
    stepFields: string[];
    stepItems: {step: number; name: string; description: string}[];
  }
}
export default Vue.extend({
  name: 'help-modal-content',
  data (): object {
    return {
      iconFields: [
        { key: 'status', label: 'Icon' },
        'name',
        'description'
      ],
      iconItems: [
        { name: 'Finished', status: { icon: 'check-circle', color: 'success' }, description: 'Success, Finished' },
        { name: 'Waiting', status: { icon: 'hourglass-start', color: 'secondary' }, description: 'Waiting, On hold, Paused' },
        { name: 'Error', status: { icon: 'exclamation-circle', color: 'danger' }, description: 'Error, Run/Project Failed' },
        { name: 'Warning', status: { icon: 'exclamation-triangle', color: 'warning' }, description: 'Warning, Something is not right' },
        { name: 'Message', status: { icon: 'envelope-square', color: 'secondary' }, description: 'Comments are added to this instance' }
      ],
      colorFields: [
        { key: 'color', label: 'Color' },
        'description'
      ],
      colorItems: [
        { color: 'primary', description: 'Selected or Running' },
        { color: 'danger', description: 'Error, somthing has crashed or stopped working' },
        { color: 'success', description: 'Finished, Complete, Done' },
        { color: 'warning', description: 'Warning, something is wrong' },
        { color: 'secondary', description: 'Not currently active, Waiting' }
      ],
      stepFields: [
        'step',
        'name',
        'description'
      ],
      stepItems: [
        { step: 1, name: 'Demultiplexing', description: 'Creating FastQ files' },
        { step: 2, name: 'Copying raw data', description: 'Copying raw data files to processing storage location' },
        { step: 3, name: 'Running', description: 'Running the pipelines that belong to the unique run' },
        { step: 4, name: 'Copying results', description: 'Copying pipeline results to final storage location' },
        { step: 5, name: 'Finished', description: 'Step is reached when all results have been copied' }
      ]
    }
  }
})
</script>

<style lang="scss" scoped>

.success {
    color: $success;
}
.primary {
    color: $primary
}
.secondary {
    color: $secondary
}
.warning {
    color: $warning
}
.danger {
    color: $danger
}

</style>
