<template>
  <b-tr
    name="slide"
    :id="'runId-' + run"
    :variant="variant"
    @click="selectRun(run)"
    @mouseover="setMouseOn(run)"
    borderless>
      <b-td v-show="mouseOn" class="align-middle">
        <b-form-checkbox
          v-model="isChecked"
          :value="run"
          :id="run"
          >
        </b-form-checkbox>
      </b-td>
      <b-td :colspan="mouseOn ? 6 : selected ? 6 : 2" class="text-truncate align-middle runID">{{run}}</b-td>
      <b-td :colsoan="1" class="align-middle" v-if="mouseOn || selected">
        <div class="d-flex align-items-end justify-content-center h-100">
          <font-awesome-icon class="icons" :icon="isInQueue ? ['fas', 'hourglass-start'] : finished ? ['fas', 'check-circle'] : error ? ['fas', 'exclamation-circle']: hasWarning ? ['fas', 'exclamation-triangle'] : ['fas', 'sync-alt']" :class="isInQueue ? 'secondary-dark' : finished ? 'success' : error ? 'danger': hasWarning ? 'warning' : 'primary'" size="lg" :spin="!finished && !error && !isInQueue && !hasWarning"></font-awesome-icon>
          </div>
        </b-td>
      <b-td colspan="5" v-show="!mouseOn && !selected"  class="text-center align-middle">
        <progress-bar
        @progress-finish="emitFinish(run)"
        :totalSteps="5"
        :variant="error ? 'danger' : hasWarning ? 'warning': step === 4 ? 'success' : 'primary'"
        :animated="step !== 4 && !error"
        :noWarning="!hasWarning"
        class="mt-1" :step="step + 1">
        </progress-bar>
      </b-td>
</b-tr>
</template>

<script lang="ts">
import Vue from 'vue'
import ProgressBar from '@/components/Track&Trace-Components/ProgressBar.vue'
import { mapState } from 'vuex'
import { RunData, Project } from '@/types/Run'
import { statusCode } from '@/types/dataTypes'

declare module 'vue/types/vue' {
  interface Vue {
    run: string;
    mouseOn: string;
    step: number;
    error: boolean;
    hidden: Array<string>;
    variant: string;
    finished: boolean;
    localHidden: string[];
    isChecked: boolean;
    runV2: Record<string, RunData>;
    projectDates: Record<string, {startedDate: Date; finishedDate?: Date}>;
  }
}

export default Vue.extend({
  name: 'run-status-table-row',
  components: {
    ProgressBar
  },
  data () {
    return {
      finished: false,
      threshold: 4
    }
  },
  props: {
    run: {
      type: String,
      required: true
    },

    mouseOn: {
      type: Boolean,
      required: false,
      default: false
    },

    step: {
      type: Number,
      required: true
    },

    error: {
      type: Boolean,
      required: false,
      return: false
    },

    hidden: {
      type: Array,
      required: true
    },

    selected: {
      type: Boolean,
      required: false,
      default: false
    }

  },
  methods: {
    /**
     * Changes the selected run
     * @param {String} run - run Id to select
     *
     * @emits 'select-run'
     * @returns {void}
     */
    selectRun (run: string): void {
      this.$emit('select-run', run)
    },

    /**
     * Noitifys run as finished
     * @param {String} run - run to pass on with status finished
     *
     * @emits 'progress-finish'
     * @returns {void}
     */
    emitFinish (run: string): void {
      this.finished = true
      this.$emit('progress-finish', run)
    },


    /**
     * Change run where mouse is on
     * @param {String} run - the new run
     *
     * @emits 'mouse-on'
     * @returns {void}
     */
    setMouseOn (run: string): void {
      this.$emit('mouse-on', run)
    }
  },
  computed: {
    ...mapState([
      'runV2',
      'projectDates'
    ]),
    /**
     * Returns variant class for row
     * 
     * @returns {danger, warning, success, primary, light}
     */
    variant (): string {
      if (this.selected) {
        return this.error ? 'danger' : this.hasWarning ? 'warning': this.step === 4 ? 'success' : 'primary'
      }
      return 'light'
    },
    localHidden: {
      /**
       * returns the hidden status of the object
       */
      get: function(): string[] {
        //@ts-ignore
        return this.hidden
      },
      /**
       * pushes row to hidden values
       * @emits update-hidden
       */
      set: function(value: string): void{
        //@ts-ignore
        this.$emit('update-hidden', value)
      }
    },
    
    isChecked: {
      /**
       * returns the checkbox status
       */
      get: function(): boolean | string {
        //@ts-ignore
        return this.localHidden.includes(this.run) ? false : this.run
      },
      /**
       * updates the hidden values to include current row
       * @param value - this run id
       */
      set: function (value: string): void {
        if (value) {
          //@ts-ignore
          this.localHidden = this.localHidden.filter((x: string) => x !== value)
        } else {
          //@ts-ignore
          this.localHidden = [this.run, ...this.localHidden]
        }
      }
    },
    /**
     * calculates if run is in queue
     */
    isInQueue (): boolean {
      return this.step === -1
    },
    /**
     * calculates if run has a warning
     * Goes through every project in the run to check for errors within the jobs
     */
    hasWarning (): boolean {
      //@ts-ignore
      const currentRun: RunData = this.runV2[this.run]
      const projects: {project: string; copyStatus: statusCode}[] = currentRun.projects.map((project: Project) => {return {project: project.projectID, copyStatus: project.getStatus()}})
      let warning = false
      projects.forEach((project) => {
        //@ts-ignore
        const dates: {startedDate: Date; finishedDate?: Date} = this.projectDates[project.project]
        
        if (dates && !warning) {
          if (!dates.finishedDate && project.copyStatus !== statusCode.finished) {
            if ((Date.now() - dates.startedDate.getTime()) / 3600 > this.threshold) {
              warning = true
            }
          }
        }
      })
      return warning
    }
  }
})
</script>

<style lang="scss" scoped>
.icons {
  height: 1vw;
  width: 1vw;
}

.runID {
  font-size: 1vw;
}

.success {
    color: $success;
}
.primary {
    color: $primary
}
.secondary {
    color: $gray-300
}
.secondary-dark {
  color: $secondary
}
.warning {
    color: $warning
}
.danger {
    color: $danger
}
.warning {
  color: $warning
}
</style>
