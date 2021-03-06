<template>
  <label :for="id" :class="{ 'oc-cursor-pointer': !disabled }">
    <input
      :id="id"
      v-model="$_ocRadio_model"
      type="radio"
      name="radio"
      :aria-label="label"
      :class="$_ocRadio_classes"
      :value="option"
      :disabled="disabled"
    />
    <span v-if="!hideLabel" :aria-hidden="true" v-text="label" />
  </label>
</template>
<script>
import { getSizeClass } from "../utils/sizeClasses"
import uniqueId from "../utils/uniqueId"
/**
 * The radio element. Can be grouped to give the user to choose between different options.
 */
export default {
  name: "OcRadio",
  status: "review",
  release: "1.0.0",
  props: {
    /**
     * Id for the radio. If it's empty, a generated one will be used.
     */
    id: {
      type: String,
      required: false,
      default: uniqueId("oc-radio-"),
    },
    /**
     * Disables the radio button
     */
    disabled: {
      type: Boolean,
      default: false,
    },
    /**
     * The model of the radio button or radio button group. It determines, based on the option this radio button
     * represents, whether or not this radio button is checked. Provide it as value or bind it with v-model.
     *
     * When used in a radio group, provide a variable that tracks which of the `option` values of the group is checked.
     *
     * Can be any type.
     **/
    // eslint-disable-next-line vue/require-prop-types
    value: {
      required: false,
      default: false,
    },
    /**
     * The value of this radio button. Can be omitted if the radio button is not used in a group.
     *
     * Can be of any type.
     */
    // eslint-disable-next-line vue/require-prop-types
    option: {
      required: false,
      default: null,
    },
    /**
     * Label of the Radio.
     *
     * Always required for aria-label property. If you want to hide the label, use `hideLabel` property.
     **/
    label: {
      type: String,
      required: true,
      default: null,
    },
    /**
     * Is the label of the Radio visually hidden?
     **/
    hideLabel: {
      type: Boolean,
      required: false,
      default: false,
    },
    /**
     * Size of the Radio. Valid values are `small`, `medium` and `large`.
     * If not specified, defaults to `medium`
     */
    size: {
      type: String,
      required: false,
      default: "medium",
      validator: size => /(small|medium|large)/.test(size),
    },
  },
  computed: {
    $_ocRadio_model: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit("input", value)
      },
    },
    $_ocRadio_classes() {
      return ["oc-radio", "oc-radio-" + getSizeClass(this.size)]
    },
  },
}
</script>
<docs>
  ```
  <template>
    <div>
      <section>
        <h3 class="uk-heading-divider oc-mt-s">
          Radio button group
        </h3>
        <div class="oc-mb-s">
          <oc-radio
              v-for="o in availableOptions"
              :key="'option-' + o"
              v-model="selectedOption"
              :option="o"
              :label="o"
              class="oc-mr-s"
          />
        </div>
        Selected option: {{ selectedOption || "None" }}
      </section>
    </div>
  </template>
  <script>
    export default {
      data: () => ({
        availableOptions: ["Water", "Wine", "Beer"],
        selectedOption: null
      })
    }
  </script>
  ```
</docs>
