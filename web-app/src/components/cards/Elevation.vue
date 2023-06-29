<template>
  <div>
    <ModeButton :value="mode" @input="setMode" />
    <TimeButtons :value="timeAgo" @input="setTimeAgo" :zoomed-in="zoomedIn" @reset-zoom="zoomedIn = false" />
    <CurrentView v-if="mode === 'current' && measurements.length">
      <template v-slot:header1>Estimated</template>
      <template v-slot:value1>{{ averageElevation }} meters</template>
    </CurrentView>
    <CurrentView v-else-if="mode === 'current'">
      <template v-slot:header1>Estimated</template>
      <template v-slot:value1>N/A</template>
    </CurrentView>
    <Graph
      v-else
      chart-type="area"
      :name="sensor.label"
      :measurements="measurements"
      :options="chartOptions"
      :zoomed-in="zoomedIn"
      @zoomed-in="zoomedIn = true"
      />
    <!-- Don't show this on the dashboard -->
    <BookmarkButton v-if="!sensor.settings" :mode="mode" :sensor-id="sensor.id" :time-ago="timeAgo" />
  </div>
</template>

<script>
import BookmarkButton from '../BookmarkButton'
import CurrentView from '../CurrentView'
import Graph from '../Graph'
import ModeButton from '../ModeButton'
import TimeButtons from '../TimeButtons'

export default {
  components: {
    BookmarkButton,
    CurrentView,
    Graph,
    ModeButton,
    TimeButtons
  },
  props: {
    sensor: {
      required: true,
      type: Object
    }
  },
  data() {
    return {
      // Hydrate from sensor "settings" if this is on the dashboard
      mode: this.sensor.settings?.mode || 'current',
      timeAgo: this.sensor.settings?.timeAgo || Infinity,
      zoomedIn: false
    }
  },
  computed: {
    averageElevation() {
      const sum = this.measurements.reduce((acc, el) => acc + Number(el.value), 0)
      return Math.round((sum / this.measurements.length) * 10) / 10
    },
    measurements() {
      if (this.timeAgo === Infinity) {
        return this.sensor.measurements
      }
      const now = new Date().getTime()
      return this.sensor.measurements
        // Filter down to the last 48 hours
        .filter(m => now - Math.round(new Date(m.created_at).getTime()) <= this.timeAgo)
    }
  },
  methods: {
    setMode(newMode) {
      this.mode = newMode
      // State of graph gets reset with mode changes
      this.zoomedIn = false
      this.$emit('change-mode', this.mode)
    },
    setTimeAgo(timeAgo) {
      this.timeAgo = timeAgo
      this.$emit('change-time-ago', this.timeAgo)
    }
  }
}
</script>