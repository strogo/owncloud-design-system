<template>
  <div class="oc-alerts" :class="$_ocNotifications_classes">
    <slot></slot>
  </div>
</template>
<script>
/**
 * Notifications are used to inform users about errors, warnings and as confirmations for their actions.
 *
 * The default slot can be filled with [oc-notification-message](#/oC%20Components/oc-notification-message) elements.
 *
 * ## Accessibility
 *
 * ### Notifications for screen reader users
 * This component uses so called live regions in order to announce its content to screen readers once the notification appeared (this is not the normal modus operandi for screen readers, since their reading order is usually the DOM order – when the user does not take shortcuts). There are two types of live regions: `aria-live="polite"` (equivalent to `role="status"`) and `aria-live="assertive"` (equivalent to `role="alert"`). The latter directly interrupts the current output of the screen reader, the former waits until the current output is finished and reads the announcement afterwards. Since 'assertive' should be used sparingly, only `<oc-notfication-message>`'s "danger" status prop value uses `aria-live="assertive"` (and `role="alert"`). Using `aria-live` and `role="assertive|status"` simultaneously is for compatibility reasons regarding different browser and assistive technology pairings.
 */
export default {
  name: "OcNotifications",
  status: "review",
  release: "1.0.0",
  props: {
    /**
     * Position of the notifications.
     * Valid values are: `top-left, top-center and top-right`
     */
    position: {
      type: String,
      required: false,
      default: "top-center",
      validator: value => {
        return value.match(/(top-left|top-center|top-right)/)
      },
    },
  },
  computed: {
    $_ocNotifications_classes() {
      return `uk-notification uk-notification-${this.position}`
    },
  },
}
</script>
<docs>
```vue
  <template>
    <section>
      <h3 class="uk-heading-divider">
        Notifications examples
      </h3>

      <oc-grid gutter="small">
        <div>
          <oc-button @click="show('center')">Show center notifications</oc-button>
          <oc-notifications position="top-center">
            <transition-group name="oc-alerts-transition" tag="div" class="oc-alerts-transition">
              <oc-notification-message
                  v-for="item in messages.center"
                  :key="item.title"
                  :status="item.status"
                  :title="item.title"
                  :message="item.message"
                  @close="removeNotification('center', item)"
              />
            </transition-group>
          </oc-notifications>
        </div>
        <div>
          <oc-button @click="show('right')">Show right notifications</oc-button>
          <oc-notifications position="top-right">
            <oc-notification-message
                v-for="(item, index) in messages.right"
                :key="index"
                :status="item.status"
                :title="item.title"
                :message="item.message"
                @close="removeNotification('right', item)"
            />
          </oc-notifications>
        </div>
      </oc-grid>
    </section>
  </template>
  <script>
    export default {
      data: () => {
        return {
          messages: {
            left: [],
            center: [],
            right: []
          }
        }
      },
      methods: {
        removeNotification(position, item) {
          this.messages[position] = this.messages[position].filter(el => el !== item);
        },
        show(position) {
          this.messages[position] = [
            {
              title: 'I do not have any message',
              status: 'primary'
            },
            {
              title: 'This is a very long title that spans more than just one line',
              message: 'Are you sure?',
              status: 'warning'
            },
            {
              title: 'Folder created',
              message: 'Just another day in paradise',
              status: 'success'
            },
            {
              title: 'Upload failed',
              message: 'Holy Shoot. Something broke',
              status: 'danger'
            },
          ]
        }
      }
    }
  </script>
```
</docs>
