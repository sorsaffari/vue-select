<style lang="scss">
  @import '../scss/vue-select.scss';
</style>

<template>
  <div class="v-select vs--single vs--unsearchable" :class="stateClasses">
    <div :id="`vs${uid}__combobox`" ref="toggle" @mousedown.prevent="toggleDropdown" class="vs__dropdown-toggle" role="combobox">

      <div class="vs__selected-options">
          <span class="vs__selected">
            {{ selectedValue.label }}
          </span>
      </div>

      <div class="vs__actions" ref="actions">
        <svg xmlns="http://www.w3.org/2000/svg" width="14" height="10" ref="openIndicator" role="presentation" class="vs__open-indicator">
          <path d="M9.211364 7.59931l4.48338-4.867229c.407008-.441854.407008-1.158247 0-1.60046l-.73712-.80023c-.407008-.441854-1.066904-.441854-1.474243 0L7 5.198617 2.51662.33139c-.407008-.441853-1.066904-.441853-1.474243 0l-.737121.80023c-.407008.441854-.407008 1.158248 0 1.600461l4.48338 4.867228L7 10l2.211364-2.40069z"/>
        </svg>
      </div>
    </div>

    <transition name="vs__fade">
      <ul ref="dropdownMenu" v-show="dropdownOpen" :id="`vs${uid}__listbox`" class="vs__dropdown-menu" role="listbox" @mousedown.prevent="onMousedown" @mouseup="onMouseUp">
        <template v-if="dropdownOpen">
          <li
            role="option"
            v-for="(option, index) in options"
            :key="JSON.stringigy(option)"
            :id="`vs${uid}__option-${index}`"
            class="vs__dropdown-option"
            :class="{ 'vs__dropdown-option--selected': isOptionSelected(option), 'vs__dropdown-option--highlight': index === typeAheadPointer }"
            @mouseover="typeAheadPointer = index"
            @mousedown.prevent.stop="select(option)"
          >
            {{ getOptionLabel(option) }}
          </li>
        </template>
      </ul>
    </transition>
  </div>
</template>

<script type="text/babel">
  import pointerScroll from '../mixins/pointerScroll'
  import typeAheadPointer from '../mixins/typeAheadPointer'
  import uniqueId from '../utility/uniqueId';

  const LABEL_KEY = "label";

  /**
   * @name VueSelect
   */
  export default {
    mixins: [pointerScroll, typeAheadPointer],

    props: {
      /**
       * Contains the currently selected value. Very similar to a
       * `value` attribute on an <input>. You can listen for changes
       * using 'change' event using v-on
       * @type {Object||null}
       */
      value: {},

      /**
       * An array of objects to be used as dropdown choices.
       * @type {Array}
       */
      options: {
        type: Array,
        default() {
          return []
        },
      },

      /**
       * Disable the entire component.
       * @type {Boolean}
       */
      disabled: {
        type: Boolean,
        default: false
      },

      /**
       * Equivalent to the `placeholder` attribute on an `<input>`.
       * @type {String}
       */
      placeholder: {
        type: String,
        default: ''
      },
    },

    data() {
      return {
        uid: uniqueId(),
        open: false,
        _value: [] // Internal value managed by Vue Select if no `value` prop is passed
      }
    },

    watch: {
      /**
       * Make sure selected option is correct.
       */
      options() {
        if (this.value && this.isTrackingValues) {
          this.setInternalValueFromOptions(this.value);
        }
      },

      /**
       * Make sure to update internal
       * value if prop changes outside
       */
      value(val) {
        if (this.isTrackingValues) {
          this.setInternalValueFromOptions(val)
        }
      },
    },

    created() {
      if (typeof this.value !== "undefined" && this.isTrackingValues) {
        this.setInternalValueFromOptions(this.value)
      }
    },

    methods: {
      /**
       * Verifies the existent of LABEL_KEY in 
       * the option object and returns its value.
       * @param  {Object} option
       * @return {String}
       */
      getOptionLabel() {
          if (!option.hasOwnProperty(LABEL_KEY)) {
            return console.warn(
              `[vue-select warn]: Label key "option.${LABEL_KEY}" does not` +
              ` exist in options object ${JSON.stringify(option)}.\n` +
              'https://vue-select.org/api/props.html#getoptionlabel'
            );
          }
          return option[LABEL_KEY];
      },

      /**
       * Set the internal value
       * @param  {Object} value
       * @return {void}
       */
      setInternalValueFromOptions(value) {
        this.$data._value = this.findOptionFromValue(value);
      },

      /**
       * Select a given option.
       * @param  {Object} option
       * @return {void}
       */
      select(option) {
        if (!this.isOptionSelected(option)) {
          this.updateValue(option);
        }

        this.onAfterSelect(option)
      },

      /**
       * Called from this.select after each selection.
       * @param  {Object} option
       * @return {void}
       */
      onAfterSelect(option) {
          this.open = !this.open
      },

      /**
       * Accepts a selected value, updates local
       * state when required, and triggers the
       * input event.
       *
       * @emits input
       * @param {Object} value
       */
      updateValue (value) {
        if (this.isTrackingValues) {
          // Vue select has to manage value
          this.$data._value = value;
        }

        this.$emit('input', value);
      },

      /**
       * Toggle the visibility of the dropdown menu.
       * @param  {Event} e
       * @return {void}
       */
      toggleDropdown ({target}) {
        if (!this.open && !this.disabled) {
          this.open = true;
        }
      },

      /**
       * Check if the given option is currently selected.
       * @param  {Object} option
       * @return {Boolean} True when selected | False otherwise
       */
      isOptionSelected(option) {
        if (this.getOptionLabel(this.selectedValue) === this.getOptionLabel(option)) {
          return true
        }
        return false;
      },

      /**
       * Finds an option from this.options
       * where a value matches the passed in value.
       *
       * @param {Object} value
       * @returns {*}
       */
      findOptionFromValue (value) {
        return this.options.find(option => JSON.stringify(option) === JSON.stringify(value)) || value;
      },

      /**
       * Event-Handler to help workaround IE11 (probably fixes 10 as well)
       * firing a `blur` event when clicking
       * the dropdown's scrollbar, causing it
       * to collapse abruptly.
       * @see https://github.com/sagalbot/vue-select/issues/106
       * @return {void}
       */
      onMousedown() {
        this.mousedown = true
      },

      /**
       * Event-Handler to help workaround IE11 (probably fixes 10 as well)
       * @see https://github.com/sagalbot/vue-select/issues/106
       * @return {void}
       */
      onMouseUp() {
        this.mousedown = false
      },
    },

    computed: {
      /**
       * Determine if the component needs to
       * track the state of values internally.
       * @return {boolean}
       */
      isTrackingValues () {
        return typeof this.value === 'undefined';
      },

      /**
       * The options that are currently selected.
       * @return {Array}
       */
      selectedValue () {
        let value = this.value;

        if (this.isTrackingValues) {
          // Vue select has to manage value internally
          value = this.$data._value;
        }

        if (value) {
          return [].concat(value);
        }

        return [];
      },

      /**
       * Holds the current state of the component.
       * @return {Object}
       */
      stateClasses() {
        return {
          'vs--open': this.dropdownOpen,
          'vs--disabled': this.disabled
        }
      },

      /**
       * Return the current state of the
       * dropdown menu.
       * @return {Boolean} True if open
       */
      dropdownOpen() {
        return this.open;
      },
    },

  }
</script>
