<template>
  <PanelItem :index="index" :field="field">
    <template #value>
      <template v-if="this.field.displayedAs">{{
        this.field.displayedAs
      }}</template>
      <template v-else-if="this.field.visibleOnDetail">
        <div @click.prevent.stop class="flex flex-row">
          <div
            v-if="this.visible"
            ref="theFieldValue"
            class="flex items-center overflow-hidden text-nowrap"
          >
            <IndexCopyableField
              class="truncate break-words block"
              :value="fieldValue"
              @copy="handleCopy"
            />
          </div>
          <div v-else>***********</div>
          <div
            @click.prevent.stop="toggleVisibility"
            class="w-max ml-1 rtl:ml-0 rtl:mr-1"
          >
            <div v-if="!this.visible" v-tooltip="this.field.showMessage">
              <Icon
                type="eye-off"
                class="size-4 text-destructive-foreground hover:text-primary-foreground"
              />
            </div>
            <div v-else v-tooltip="this.field.hideMessage">
              <Icon
                type="eye"
                class="size-4 text-destructive-foreground hover:text-primary-foreground"
              />
            </div>
          </div>
        </div>
      </template>
      <p v-else>***********</p>
    </template>
  </PanelItem>
</template>

<script>
export default {
  props: ["index", "resource", "resourceName", "resourceId", "field"],

  methods: {
    toggleVisibility() {
      if (this.visible) {
        this.visible = false;
      } else {
        this.visible = true;
      }
    },

    async handleCopy() {
      const value = this.fieldValue;

      if (!value) {
        return;
      }

      try {
        if (navigator.clipboard) {
          await navigator.clipboard.writeText(value);
        } else if (window.clipboardData) {
          window.clipboardData.setData("Text", value);
        } else {
          // Fallback for very old browsers (execCommand is deprecated but necessary for compatibility)
          const input = document.createElement("input");
          const [scrollTop, scrollLeft] = [
            document.documentElement.scrollTop,
            document.documentElement.scrollLeft,
          ];
          document.body.appendChild(input);
          input.value = value;
          input.focus();
          input.select();
          document.documentElement.scrollTop = scrollTop;
          document.documentElement.scrollLeft = scrollLeft;
          document.execCommand("copy");
          input.remove();
        }
        Nova.success(this.__("Copied to clipboard"));
      } catch (error) {
        Nova.error(this.__("Failed to copy to clipboard"));
        console.error("Failed to copy to clipboard:", error);
      }
    },
  },

  computed: {
    label() {
      return this.fieldName || this.field.name;
    },

    fieldValue() {
      if (!this.visible || !this.field.visibleOnDetail) {
        return "";
      }

      return this.field.valueDetail;
    },
  },

  data() {
    return {
      visible: false,
    };
  },
};
</script>
