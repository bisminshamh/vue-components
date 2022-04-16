<template>
  <v-card>
    <v-list>
      <v-card-title class="ma-0">{{ item.title }} </v-card-title>

      <v-list-item v-for="(item, i) in item.items" :key="i">
        <v-list-item-icon>
          <v-icon :color="icon(item, 'icon_color')">
            {{ icon(item, "icon") }}
          </v-icon>
        </v-list-item-icon>

        <v-list-item-content>
          <v-list-item-title
            class="text-xl-body-1"
            v-text="item.title"
          ></v-list-item-title>
        </v-list-item-content>

        <v-list-item-action>
          <v-switch
            v-if="item.options.isSwitchable"
            v-model="item.value"
          ></v-switch>
          <v-chip outlined v-else>
            <v-list-item-action-text
              class="text-h6"
              v-text="item.value + ' ' + (item.unit ? item.unit : '')"
            >
            </v-list-item-action-text>
          </v-chip>
        </v-list-item-action>
      </v-list-item>
    </v-list>
  </v-card>
</template>
<script lang="ts">
import Vue, { PropType } from "vue";
import {
  entities,
  entitiesItem,
  icon_key,
  icon_object_key,
} from "../types/index";
/**
 * The Entity card gives you an overview of your entity’s state.
 * @displayName Entities
 */
export default Vue.extend({
  name: "Entities",

  props: {
    /**
     * item for entities
     */
    item: {
      required: true,
      type: Object as PropType<entities>,
    },
  },

  methods: {
    /**
     * display icon based on condition
     * @param {entitiesItem} item Entity item
     * @param {icon_key} name function executes bases on value of the name {icon/icon_color}
     */
    icon(item: entitiesItem, name: icon_key): string {
      switch (item.options.isSwitchable) {
        case true:
          return this.returnIconColor(item, name, item.value ? "on" : "off");
        case false:
          switch (item.options.condition) {
            case "<":
              if (item.options.conditionValue)
                return this.returnIconColor(
                  item,
                  name,
                  item.value < item.options.conditionValue ? "on" : "off"
                );
              break;
            case ">":
              if (item.options.conditionValue)
                return this.returnIconColor(
                  item,
                  name,
                  item.value > item.options.conditionValue ? "on" : "off"
                );
              break;
            case "===":
              if (item.options.conditionValue)
                return this.returnIconColor(
                  item,
                  name,
                  item.value === item.options.conditionValue ? "on" : "off"
                );
              break;
            default:
              return this.returnIconColor(item, name, "on");
          }
          return "";
        default:
          return this.returnIconColor(item, name, "on");
      }
    },
    /**
     * return statement for icon function
     * @param {entitiesItem} item Entity item
     * @param {icon_object_key} type Entity item value
     * @param {icon_key} name function executes bases on value of the name {icon/icon_color}
     */
    returnIconColor(
      item: entitiesItem,
      name: icon_key,
      type: icon_object_key
    ): string {
      return item.options[name][type];
    },
  },
});
</script>