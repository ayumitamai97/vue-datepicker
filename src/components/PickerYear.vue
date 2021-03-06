<template>
  <div class="picker-view">
    <slot name="beforeCalendarHeaderYear" />
    <PickerHeader
      :config="headerConfig"
      :next="nextDecade"
      :previous="previousDecade"
    >
      <span>
        {{ getPageDecade }}
      </span>
      <slot slot="nextIntervalBtn" name="nextIntervalBtn" />
      <slot slot="prevIntervalBtn" name="prevIntervalBtn" />
    </PickerHeader>

    <span
      v-for="year in years"
      :key="year.timestamp"
      :class="{ selected: year.isSelected, disabled: year.isDisabled }"
      class="cell year"
      @click.stop="selectYear(year)"
    >
      {{ year.year }}
    </span>
    <slot name="calendarFooterYear" />
  </div>
</template>
<script>
import pickerMixin from '~/mixins/pickerMixin.vue'
import { isYearDisabled } from '~/utils/DisabledDatesUtils'

export default {
  name: 'DatepickerYearView',
  mixins: [pickerMixin],
  props: {
    yearRange: {
      type: Number,
      default: 10,
    },
  },
  computed: {
    /**
     * Checks if the next decade is disabled or not
     * @return {Boolean}
     */
    isNextDisabled() {
      if (!this.disabledDates || !this.disabledDates.from) {
        return false
      }
      const yearFrom = this.utils.getFullYear(this.disabledDates.from)
      const lastCellYear = this.years[this.years.length - 1].year

      return yearFrom <= lastCellYear
    },
    /**
     * Checks if the previous decade is disabled or not
     * @return {Boolean}
     */
    isPreviousDisabled() {
      if (!this.disabledDates || !this.disabledDates.to) {
        return false
      }
      const yearTo = this.utils.getFullYear(this.disabledDates.to)
      const yearPageDate = this.utils.getFullYear(this.pageDate)

      return (
        Math.floor(yearTo / this.yearRange) * this.yearRange >=
        Math.floor(yearPageDate / this.yearRange) * this.yearRange
      )
    },
    /**
     * Get decade name on current page.
     * @return {String}
     */
    getPageDecade() {
      const yearPageDate = this.utils.getFullYear(this.pageDate)

      const decadeStart =
        Math.floor(yearPageDate / this.yearRange) * this.yearRange
      const decadeEnd = decadeStart + (this.yearRange - 1)
      const { yearSuffix } = this.translation
      return `${decadeStart} - ${decadeEnd}${yearSuffix}`
    },
    /**
     * Set an array with years for a decade
     * @return {Array}
     */
    years() {
      const d = this.pageDate
      const years = []
      const year = this.useUtc
        ? Math.floor(d.getUTCFullYear() / this.yearRange) * this.yearRange
        : Math.floor(d.getFullYear() / this.yearRange) * this.yearRange

      // set up a new date object to the beginning of the current 'page'7
      const dObj = this.useUtc
        ? new Date(Date.UTC(year, d.getUTCMonth(), d.getUTCDate()))
        : new Date(
            year,
            d.getMonth(),
            d.getDate(),
            d.getHours(),
            d.getMinutes(),
          )
      for (let i = 0; i < this.yearRange; i += 1) {
        years.push({
          year: this.utils.getFullYear(dObj),
          timestamp: dObj.valueOf(),
          isSelected: this.isSelectedYear(dObj),
          isDisabled: this.isDisabledYear(dObj),
        })
        this.utils.setFullYear(dObj, this.utils.getFullYear(dObj) + 1)
      }
      return years
    },
  },
  methods: {
    /**
     * Changes the year up or down
     * @param {Number} incrementBy
     */
    changeYear(incrementBy) {
      const date = this.pageDate
      this.utils.setFullYear(date, this.utils.getFullYear(date) + incrementBy)
      this.$emit('changed-decade', date)
    },
    /**
     * Whether a year is disabled
     * @param {Date} date
     * @return {Boolean}
     */
    isDisabledYear(date) {
      return isYearDisabled(date, this.disabledDates, this.utils)
    },
    /**
     * Whether the selected date is in this year
     * @param {Date} date
     * @return {Boolean}
     */
    isSelectedYear(date) {
      return (
        this.selectedDate &&
        this.utils.getFullYear(this.selectedDate) ===
          this.utils.getFullYear(date)
      )
    },
    /**
     * Increments the decade
     */
    nextDecade() {
      if (!this.isNextDisabled) {
        this.changeYear(this.yearRange)
      }
    },
    /**
     * Decrements the decade
     */
    previousDecade() {
      if (!this.isPreviousDisabled) {
        this.changeYear(-this.yearRange)
      }
    },
    /**
     * Emits a selectYear event
     * @param {Object} year
     */
    selectYear(year) {
      if (!year.isDisabled) {
        this.$emit('select-year', year)
      }
    },
  },
}
</script>
