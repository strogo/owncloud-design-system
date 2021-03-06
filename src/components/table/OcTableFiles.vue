<template>
  <oc-table
    :data="resources"
    :fields="fields"
    :highlighted="highlighted"
    :row-height="rowHeight"
    :sticky="true"
    @highlight="showDetails"
  >
    <template #selectHeader>
      <div class="oc-table-files-select-all">
        <oc-checkbox
          id="oc-table-files-select-all"
          :label="$gettext('Select all resources')"
          :hide-label="true"
          :value="areAllResourcesSelected"
          @input="toggleSelectionAll"
        />
      </div>
    </template>
    <template #select="rowData">
      <oc-checkbox
        :id="`oc-table-files-select-${rowData.item.id}`"
        :label="$gettext('Select')"
        :hide-label="true"
        size="large"
        :value="selection"
        :option="rowData.item.id"
        @input="emitSelect"
        @click.native.stop
      />
    </template>
    <template #name="rowData">
      <oc-resource
        :resource="rowData.item"
        :is-path-displayed="arePathsDisplayed"
        :is-preview-displayed="arePreviewsDisplayed"
        :target-route="targetRoute"
        :is-resource-clickable="isResourceClickable"
        @click="emitFileClick(rowData.item)"
      />
    </template>
    <template #status="rowData">
      <!-- @slot Status column -->
      <slot name="status" :resource="rowData.item" />
    </template>
    <template #sharedWith="rowData">
      <oc-avatar-group
        class="oc-table-files-people"
        :users="rowData.item.sharedWith"
        :stacked="true"
        :max-displayed="3"
        :is-tooltip-displayed="true"
      />
    </template>
    <template #size="rowData">
      <oc-resource-size :size="rowData.item.size" />
    </template>
    <template #owner="rowData">
      <oc-avatar-group
        class="oc-table-files-people"
        :users="rowData.item.owner"
        :is-tooltip-displayed="true"
      />
    </template>
    <template #actions="rowData">
      <div class="oc-table-files-actions">
        <!-- @slot Add quick actions directly next to the `showDetails` button in the actions column -->
        <slot name="quickActions" :resource="rowData.item" />
        <oc-button
          :aria-label="$gettext('Show details')"
          class="oc-table-files-btn-show-details"
          variation="raw"
          @click="showDetails(rowData.item)"
        >
          <oc-icon name="more_vert" aria-hidden="true" />
        </oc-button>
      </div>
    </template>
  </oc-table>
</template>

<script>
import { DateTime } from "luxon"

import OcTable from "./OcTable.vue"
import OcResource from "../resource/OcResource.vue"
import OcIcon from "../OcIcon.vue"
import OcAvatarGroup from "../avatars/OcAvatarGroup.vue"
import OcCheckbox from "../OcCheckbox.vue"
import OcButton from "../OcButton.vue"
import OcResourceSize from "../resource/OcResourceSize.vue"

export default {
  name: "OcTableFiles",
  status: "review",
  release: "2.1.0",
  components: { OcTable, OcResource, OcIcon, OcAvatarGroup, OcCheckbox, OcButton, OcResourceSize },
  model: {
    prop: "selection",
    event: "select",
  },
  props: {
    /**
     * Resources to be displayed in the table.
     * Required fields:
     * - name: The name of the resource containing the file extension in case of a file
     * - path: The full path of the resource
     * - type: The type of the resource. Can be `file` or `folder`
     * Optional fields:
     * - preview
     * - size: The size of the resource
     * - modificationDate: The date of the last modification of the resource
     * - shareDate: The date when the share was created
     * - deletionDate: The date when the resource has been deleted
     * - status: The status of the share. Contains also actions to accept/decline the share
     */
    resources: {
      type: Array,
      required: true,
    },
    /**
     * Asserts whether resources path should be shown in the resource name
     */
    arePathsDisplayed: {
      type: Boolean,
      required: false,
      default: false,
    },
    /**
     * Asserts whether icons should be replaced with previews for resources which provide them
     */
    arePreviewsDisplayed: {
      type: Boolean,
      required: false,
      default: true,
    },
    /**
     * V-model for the selection
     */
    selection: {
      type: Array,
      default: () => [],
    },
    /**
     * Asserts whether actions are available
     */
    hasActions: {
      type: Boolean,
      required: false,
      default: true,
    },
    /**
     * The ids of highlighted resources. Null or an empty string/array for no highlighting.
     */
    highlighted: {
      type: [String, Array],
      required: false,
      default: null,
    },
    /**
     * Height of the row in pixels
     */
    rowHeight: {
      type: Number,
      required: false,
      default: 64,
    },
    /**
     * Target route path used to build the link when navigating into a resource
     */
    targetRoute: {
      type: String,
      required: false,
      default: "",
    },
    /**
     * Asserts whether clicking on the resource name triggers any action
     */
    isResourceClickable: {
      type: Boolean,
      required: false,
      default: true,
    },
  },
  computed: {
    fields() {
      if (this.resources.length === 0) {
        return []
      }

      const firstResource = this.resources[0]
      const fields = [
        {
          name: "select",
          title: "",
          type: "slot",
          headerType: "slot",
          width: "shrink",
        },
      ]
      fields.push(
        ...[
          {
            name: "name",
            title: this.$gettext("Name"), // How do we get the translations here?
            type: "slot",
            width: "expand",
          },
          {
            name: "size",
            title: this.$gettext("Size"),
            type: "slot",
            alignH: "right",
          },
          {
            name: "sharedWith",
            title: this.$gettext("Shared with"),
            type: "slot",
            alignH: "right",
          },
          {
            name: "status",
            title: this.$gettext("Status"),
            type: "slot",
            alignH: "right",
          },
          {
            name: "owner",
            title: this.$gettext("Owner"),
            type: "slot",
            alignH: "right",
          },
          {
            name: "mdate",
            title: this.$gettext("Modified"),
            type: "callback",
            alignH: "right",
            callback: date => this.formatDate(date),
          },
          {
            name: "sdate",
            title: this.$gettext("Shared"),
            type: "callback",
            alignH: "right",
            callback: date => this.formatDate(date),
          },
          {
            name: "ddate",
            title: this.$gettext("Deleted"),
            type: "callback",
            alignH: "right",
            callback: date => this.formatDate(date),
          },
        ].filter(field => Object.prototype.hasOwnProperty.call(firstResource, field.name))
      )

      if (this.hasActions) {
        fields.push({
          name: "actions",
          title: this.$gettext("Actions"),
          type: "slot",
          alignH: "right",
        })
      }

      return fields
    },

    areAllResourcesSelected() {
      return this.selection.length === this.resources.length
    },
  },
  methods: {
    showDetails(resource) {
      /**
       * Triggered when the showDetails button in actions column is clicked
       * @property {object} resource resource for which the event is triggered
       */
      this.$emit("showDetails", resource)
    },

    formatDate(date) {
      return DateTime.fromJSDate(new Date(date)).toRelative()
    },

    emitSelect(ids) {
      /**
       * Triggered a checkbox for selecting resource or all is clicked
       * @property {array} ids ids of selected resources
       */
      this.$emit("select", ids)
    },

    toggleSelectionAll() {
      if (this.areAllResourcesSelected) {
        return this.emitSelect([])
      }

      this.emitSelect(this.resources.map(resource => resource.id))
    },

    emitFileClick(resource) {
      /**
       * Triggered when a default action is triggered on a file
       * @property {object} resource resource for which the event is triggered
       */
      this.$emit("fileClick", resource)
    },
  },
}
</script>

<style lang="scss">
.oc-table-files {
  &-people {
    position: absolute;
    right: $space-xsmall;
    top: 50%;
    transform: translateY(-50%);
  }

  &-actions {
    align-items: center;
    display: flex;
    flex-flow: row wrap;
    gap: $space-xsmall;
    justify-content: flex-end;
  }

  &-select-all {
    align-items: center;
    display: flex;
    justify-content: center;
  }
}
</style>

<docs>
```vue
<template>
  <div>
    <oc-table-files :resources="resources" :highlighted="highlighted" v-model="selected" class="oc-mb" @showDetails="highlightResource" @action="handleAction">
      <template v-slot:quickActions="props">
        <oc-button @click.stop variation="raw" aria-label="Share">
          <oc-icon name="group-add" aria-hidden="true" />
        </oc-button>
        <oc-button @click.stop variation="raw" aria-label="Create a public link">
          <oc-icon name="link-add" aria-hidden="true" />
        </oc-button>
      </template>
    </oc-table-files>
    <div>
      Selected resources: {{ selected }}
    </div>
  </div>
</template>
<script>
export default {
  data: () => ({
    selected: ['notes'],
    highlighted: 'forest'
  }),
  computed: {
    resources() {
      return [
        {
          id: "forest",
          name: "forest.jpg",
          path: "images/nature/forest.jpg",
          preview: "https://cdn.pixabay.com/photo/2015/09/09/16/05/forest-931706_960_720.jpg",
          indicators: [],
          type: "file",
          size: "111000234",
          mdate: "Mon, 11 Jan 2021 14:34:04 GMT"
        },
        {
          id: "notes",
          name: "notes.txt",
          path: "/Documents/notes.txt",
          icon: "text",
          indicators: this.indicators,
          type: "file",
          size: "1245",
          mdate: "Mon, 11 Jan 2021 14:34:04 GMT"
        },
        {
          id: "documents",
          name: "Documents",
          path: "/Documents",
          icon: "folder",
          indicators: [],
          type: "folder",
          size: "5324435",
          mdate: "Mon, 11 Jan 2021 14:34:04 GMT"
        }
      ]
    },
    indicators() {
      return [
        {
          id: 'files-sharing',
          label: "Shared with other people",
          visible: true,
          icon: 'group',
          handler: (resource, indicatorId) => alert(`Resource: ${resource.name}, indicator: ${indicatorId}`)
        },
        {
          id: 'file-link',
          label: "Shared via link",
          visible: true,
          icon: 'link',
        }
      ]
    },
  },
  methods: {
    highlightResource(resource) {
      this.highlighted = resource.id
    },
    handleAction(resource) {
      alert(`Clicked ${resource.name}`)
    }
  }
}
</script>
```

## Shared with others files table
```vue
<template>
  <oc-table-files :resources="resources" :arePathsDisplayed="true" v-model="selected" />
</template>
<script>
export default {
  data: () => ({
    selected: []
  }),
  computed: {
    resources() {
      return [
        {
          name: "forest.jpg",
          path: "images/nature/forest.jpg",
          preview: "https://cdn.pixabay.com/photo/2015/09/09/16/05/forest-931706_960_720.jpg",
          indicators: [],
          type: "file",
          sdate: "Mon, 11 Jan 2021 14:34:04 GMT",
          sharedWith: this.sharedWith
        },
        {
          name: "notes.txt",
          path: "/Documents/notes.txt",
          icon: "text",
          indicators: [],
          type: "file",
          sdate: "Mon, 11 Jan 2021 14:34:04 GMT",
          sharedWith: this.sharedWithOverlapping
        },
        {
          name: "Documents",
          path: "/Documents",
          icon: "folder",
          indicators: [],
          type: "folder",
          sdate: "Mon, 11 Jan 2021 14:34:04 GMT",
          sharedWith: this.sharedWithSingle
        }
      ]
    },
    sharedWith() {
      return [
        {
          id: "bob",
          username: "bob",
          displayName: "Bob",
          avatar: "https://images.unsplash.com/photo-1610216705422-caa3fcb6d158?ixid=MXwxMjA3fDB8MHxzZWFyY2h8MTB8fGZhY2V8ZW58MHwyfDB8&ixlib=rb-1.2.1&auto=format&fit=crop&w=800&q=60"
        },
        {
          id: "marie",
          username: "marie",
          displayName: "Marie",
          avatar: "https://images.unsplash.com/photo-1584308972272-9e4e7685e80f?ixid=MXwxMjA3fDB8MHxzZWFyY2h8Mzh8fGZhY2V8ZW58MHwyfDB8&ixlib=rb-1.2.1&auto=format&fit=crop&w=800&q=60"
        },
        {
          id: "john",
          username: "john",
          displayName: "John Richards Emperor of long names"
        }
      ]
    },
    sharedWithOverlapping() {
      return [
        {
          id: "bob",
          username: "bob",
          displayName: "Bob",
          avatar: "https://images.unsplash.com/photo-1610216705422-caa3fcb6d158?ixid=MXwxMjA3fDB8MHxzZWFyY2h8MTB8fGZhY2V8ZW58MHwyfDB8&ixlib=rb-1.2.1&auto=format&fit=crop&w=800&q=60"
        },
        {
          id: "marie",
          username: "marie",
          displayName: "Marie",
          avatar: "https://images.unsplash.com/photo-1584308972272-9e4e7685e80f?ixid=MXwxMjA3fDB8MHxzZWFyY2h8Mzh8fGZhY2V8ZW58MHwyfDB8&ixlib=rb-1.2.1&auto=format&fit=crop&w=800&q=60"
        },
        {
          id: "john",
          username: "john",
          displayName: "John Richards Emperor of long names"
        },
        {
          id: "einstein",
          username: "einstein",
          displayName: "Einstein"
        }
      ]
    },
    sharedWithSingle() {
      return [
        {
          id: "bob",
          username: "bob",
          displayName: "Bob",
          avatar: "https://images.unsplash.com/photo-1610216705422-caa3fcb6d158?ixid=MXwxMjA3fDB8MHxzZWFyY2h8MTB8fGZhY2V8ZW58MHwyfDB8&ixlib=rb-1.2.1&auto=format&fit=crop&w=800&q=60"
        }
      ]
    }
  }
}
</script>
```

## Shared with me files table
```vue
<template>
  <oc-table-files :resources="resources" :arePathsDisplayed="true" v-model="selected">
    <template v-slot:status="props">
      <div class="uk-flex uk-flex-middle uk-flex-right">
        <oc-button
          v-if="props.resource.status === 1 || props.resource.status === 2"
          variation="raw"
          class="oc-mr-xs"
          v-text="'Accept'"
        />
        <oc-button
          v-if="props.resource.status === 1 || props.resource.status === 0"
          variation="raw"
          class="oc-mr-xs"
          v-text="'Decline'"
        />
        <span
          v-text="shareStatus(props.resource.status)"
        />
      </div>
    </template>
  </oc-table-files>
</template>
<script>
export default {
  data: () => ({
    selected: []
  }),
  computed: {
    resources() {
      return [
        {
          name: "forest.jpg",
          path: "images/nature/forest.jpg",
          preview: "https://cdn.pixabay.com/photo/2015/09/09/16/05/forest-931706_960_720.jpg",
          indicators: [],
          type: "file",
          sdate: "Mon, 11 Jan 2021 14:34:04 GMT",
          owner: [{
            id: "bob",
            username: "bob",
            displayName: "Bob",
            avatar: "https://images.unsplash.com/photo-1610216705422-caa3fcb6d158?ixid=MXwxMjA3fDB8MHxzZWFyY2h8MTB8fGZhY2V8ZW58MHwyfDB8&ixlib=rb-1.2.1&auto=format&fit=crop&w=800&q=60"
          }],
          status: 1
        },
        {
          name: "notes.txt",
          path: "/Documents/notes.txt",
          icon: "text",
          indicators: [],
          type: "file",
          sdate: "Mon, 11 Jan 2021 14:34:04 GMT",
          owner: [{
            id: "einstein",
            username: "einstein",
            displayName: "Einstein"
          }],
          status: 0
        },
        {
          name: "Documents",
          path: "/Documents",
          icon: "folder",
          indicators: [],
          type: "folder",
          sdate: "Mon, 11 Jan 2021 14:34:04 GMT",
          owner: [{
            id: "marie",
            username: "marie",
            displayName: "Marie",
            avatar: "https://images.unsplash.com/photo-1584308972272-9e4e7685e80f?ixid=MXwxMjA3fDB8MHxzZWFyY2h8Mzh8fGZhY2V8ZW58MHwyfDB8&ixlib=rb-1.2.1&auto=format&fit=crop&w=800&q=60"
          }],
          status: 2
        }
      ]
    }
  },
  methods: {
    shareStatus(status) {
      switch (status) {
        case 0:
          return 'Accepted'

        case 1:
          return 'Pending'

        case 2:
          return 'Declined'
      }
    }
  }
}
</script>
```

## Trashbin files table
```vue
<template>
  <oc-table-files :resources="resources" :arePathsDisplayed="true" :arePreviewsDisplayed="false" v-model="selected" />
</template>
<script>
export default {
  data: () => ({
    selected: []
  }),
  computed: {
    resources() {
      return [
        {
          name: "forest.jpg",
          path: "images/nature/forest.jpg",
          icon: "image",
          indicators: [],
          type: "file",
          ddate: "Mon, 11 Jan 2021 14:34:04 GMT"
        },
        {
          name: "notes.txt",
          path: "/Documents/notes.txt",
          icon: "text",
          indicators: [],
          type: "file",
          ddate: "Mon, 11 Jan 2021 14:34:04 GMT"
        },
        {
          name: "Documents",
          path: "/Documents",
          icon: "folder",
          indicators: [],
          type: "folder",
          ddate: "Mon, 11 Jan 2021 14:34:04 GMT"
        }
      ]
    },
  }
}
</script>
```

## With disabled actions
```vue
<template>
  <oc-table-files :resources="resources" v-model="selected" :hasActions="false" :isResourceClickable="false" />
</template>
<script>
export default {
  data: () => ({
    selected: []
  }),
  computed: {
    resources() {
      return [
        {
          id: "forest",
          name: "forest.jpg",
          path: "images/nature/forest.jpg",
          preview: "https://cdn.pixabay.com/photo/2015/09/09/16/05/forest-931706_960_720.jpg",
          indicators: [],
          type: "file",
          size: "111000234",
          mdate: "Mon, 11 Jan 2021 14:34:04 GMT"
        },
        {
          id: "notes",
          name: "notes.txt",
          path: "/Documents/notes.txt",
          icon: "text",
          indicators: [],
          type: "file",
          size: "1245",
          mdate: "Mon, 11 Jan 2021 14:34:04 GMT"
        },
        {
          id: "documents",
          name: "Documents",
          path: "/Documents",
          icon: "folder",
          indicators: [],
          type: "folder",
          size: "5324435",
          mdate: "Mon, 11 Jan 2021 14:34:04 GMT"
        }
      ]
    }
  }
}
</script>
```
</docs>
