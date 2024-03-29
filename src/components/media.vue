<template>
  <v-card
    v-bind="cardStyle ? cardStyle : ''"
    :id="`media_player${item.cameraId}`"
    class="d-flex flex-column justify-center"
  >
    <div class="d-flex flex-column justify-space-around flex-grow-1">
      <v-card :elevation="0">
        <v-hover v-slot="{ hover }">
          <div
            class="camera-controls d-flex flex-column justify-space-between"
            :class="hover ? 'opacity-reduce' : 'opacity'"
          >
            <v-col
              class="d-flex justify-center"
              :class="isFullscreen ? 'pt-9' : ''"
            >
              <!-- up -->
              <v-btn
                v-if="cameraControlState('PT')"
                @click="controlEvent('up')"
                x-small
                fab
                dark
              >
                <v-icon>mdi-menu-up</v-icon>
              </v-btn>
            </v-col>
            <v-col class="d-flex justify-space-between align-center">
              <!-- left -->
              <div>
                <v-btn
                  v-if="cameraControlState('PT')"
                  @click="controlEvent('left')"
                  dark
                  fab
                  x-small
                >
                  <v-icon>mdi-menu-left</v-icon>
                </v-btn>
              </div>
              <div>
                <!-- zoom-out -->
                <v-btn
                  v-if="cameraControlState('zoom')"
                  @click="controlEvent('zoom-out')"
                  dark
                  fab
                  x-small
                >
                  <v-icon>mdi-minus-circle-outline</v-icon>
                </v-btn>

                <!-- reset -->
                <v-btn
                  v-if="cameraControlState('reset')"
                  @click="controlEvent('reset')"
                  dark
                  fab
                  x-small
                  class="mx-1"
                >
                  <v-icon>mdi-restore</v-icon>
                </v-btn>

                <!-- zoom-in -->
                <v-btn
                  v-if="cameraControlState('zoom')"
                  @click="controlEvent('zoom-in')"
                  dark
                  fab
                  x-small
                >
                  <v-icon>mdi-plus-circle-outline</v-icon>
                </v-btn>
              </div>
              <div>
                <!-- right -->
                <v-btn
                  v-if="cameraControlState('PT')"
                  @click="controlEvent('right')"
                  dark
                  fab
                  x-small
                >
                  <v-icon>mdi-menu-right</v-icon>
                </v-btn>
              </div>
            </v-col>
            <v-col
              class="d-flex justify-center align-end"
              :class="isFullscreen ? 'pb-9' : ''"
            >
              <!-- down -->
              <v-btn
                v-if="cameraControlState('PT')"
                @click="controlEvent('down')"
                x-small
                fab
                dark
              >
                <v-icon>mdi-menu-down</v-icon>
              </v-btn>
            </v-col>
          </div>
        </v-hover>
        <!-- 
        @slot for passing custom Html Elements
        -->
        <slot> </slot>
      </v-card>
    </div>
    <v-card-actions :class="isFullscreen ? 'video-controls py-0' : 'py-1'">
      <v-card-title :style="`font-size:${font.title}`" class="py-0 pl-0">{{
        item.friendlyName | capitalize
      }}</v-card-title>

      <v-spacer> </v-spacer>
      <v-hover v-slot="{ hover }">
        <div
          class="d-flex align-center justify-end"
          :class="{ 'flex-grow-1': hover }"
        >
          <!-- slider -->
          <v-slider
            v-if="hover"
            min="0"
            max="1"
            step="0.1"
            hide-details
            dense
            v-model="volume"
            @input="selectedVolume = $event"
          ></v-slider>

          <!-- speaker -->
          <v-btn v-if="options.volume" x-small @click="isMuted = !isMuted" icon>
            <v-icon v-if="isMuted">mdi-volume-off</v-icon>
            <v-icon v-else>mdi-volume-high</v-icon>
          </v-btn>
        </div>
      </v-hover>

      <!-- mic -->
      <v-btn
        class="pl-4"
        v-if="options.mic"
        x-small
        @click="isMicOn = !isMicOn"
        icon
      >
        <v-icon v-if="isMicOn">mdi-microphone</v-icon>
        <v-icon v-else>mdi-microphone-off</v-icon>
      </v-btn>

      <!-- fullscreen -->
      <v-btn
        x-small
        @click.stop="fullscreen(!isFullscreen ? 'enable' : 'disable')"
        icon
      >
        <v-icon v-if="isFullscreen">mdi-arrow-collapse-all</v-icon>
        <v-icon v-else>mdi-arrow-expand-all</v-icon>
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script lang="ts">
import Vue from "vue";
/**
 * Generic Media card which provied custom controls
 * @displayName Media Card
 */
export default Vue.extend({
  name: "MediaCard",

  data: () => ({
    isMicOn: false,
    isMuted: true,
    mediaPlayer: null as any,
    volume: 0,
    selectedVolume: 0,
    isFullscreen: false,
  }),
  props: {
    /**
     * The item for media card.
     */
    item: {
      required: true,
      type: Object,
      default() {
        return {
          cameraId: "1",
          src: "https://cdn.vuetifyjs.com/images/cards/cooking.png",
          friendlyName: "camera 1",
        };
      },
    },
    options: {
      required: true,
      type: Object,
      default() {
        return {
          PT: true,
          zoom: true,
          mic: true,
          volume: true,
        };
      },
    },
    cardStyle: {
      required: false,
      type: Object,
      default() {
        return {
          outlined: true,
          rounded: "lg",
        };
      },
    },
    font: {
      required: false,
      type: Object,
      default() {
        return {
          title: "",
        };
      },
    },
  },
  filters: {
    /**
     * Convert text case (Capitalization)
     */
    capitalize(data: string) {
      return data.replace(/\w\S*/g, (w) =>
        w.replace(/^\w/, (c) => c.toUpperCase())
      );
    },
  },
  watch: {
    /**
     * watch for isMuted prop change
     */
    isMuted(val: number): void {
      if (val) this.volume = 0;
      else this.volume = this.selectedVolume;
      this.controlEvent("mute");
    },
    /**
     * watch for isMicOn prop change
     */
    isMicOn() {
      this.controlEvent("mic");
    },
    /**
     * watch for volume prop change
     */
    volume(val: number): void {
      if (val > 0) {
        this.isMuted = false;
      } else this.isMuted = true;
      this.controlEvent("volume");
    },
    /**
     * watch for isFullscreen prop change
     */
    isFullscreen(): void {
      this.controlEvent("fullscreen");
    },
  },
  methods: {
    /**
     * show/hide camera controls
     * @public
     */
    cameraControlState(name: string): boolean {
      switch (name) {
        case "PT":
          return this.options.PT === true;

        case "zoom":
          return this.options.zoom === true;

        case "reset":
          return this.options.zoom === true || this.options.PT === true;

        default:
          return false;
      }
    },
    /**
     * Switching to fullscreen
     * @param {string} action type of action
     * @public
     */
    fullscreen(action: string): void {
      this.mediaPlayer = document.getElementById(
        `media_player${this.item.cameraId}`
      );
      if (this.mediaPlayer) {
        this.mediaPlayer.onfullscreenchange = () => {
          if (document.fullscreenElement) this.isFullscreen = true;
          else this.isFullscreen = false;
        };
      }
      switch (action) {
        case "enable":
          if (this.mediaPlayer) {
            if (this.mediaPlayer.requestFullscreen) {
              this.mediaPlayer
                .requestFullscreen()
                .catch((err: Record<string, unknown>) => {
                  alert(
                    `Error attempting to enable full-screen mode: ${err.message} (${err.name})`
                  );
                });
            }
            // Firefox
            else if (this.mediaPlayer.mozRequestFullScreen) {
              this.mediaPlayer
                .mozRequestFullScreen()
                .catch((err: Record<string, unknown>) => {
                  alert(
                    `Error attempting to enable full-screen mode: ${err.message} (${err.name})`
                  );
                });
            }
            // Chrome and Safari
            else if (this.mediaPlayer.webkitRequestFullscreen) {
              this.mediaPlayer
                .webkitRequestFullscreen()
                .catch((err: Record<string, unknown>) => {
                  alert(
                    `Error attempting to enable full-screen mode: ${err.message} (${err.name})`
                  );
                });
            }
          }
          break;
        /**
         * Exiting fullscreen
         */
        case "disable":
          if (document.fullscreenElement) {
            document
              .exitFullscreen()
              .then(() => {
                this.isFullscreen = false;
              })
              .catch((err) => console.error(err));
          }
          break;
      }
    },

    /**
     * define event type
     * @param {string} name name of the event
     * @public
     */
    controlEvent(name: string): void {
      /** Triggered when button is clicked
       * @event muted/unMuted/volume/micOn/micOff/up/down/left/right/zoom-out/zoom-in/reset
       * @type {Event}
       */
      switch (name) {
        case "mute":
          return this.emit({ type: this.isMuted ? "muted" : "unMuted" });

        case "volume":
          return this.emit({ type: name, value: this.volume });

        case "mic":
          return this.emit({ type: this.isMicOn ? "micOn" : "micOff" });

        case "fullscreen":
          return this.emit({ type: name, value: this.isFullscreen });

        default:
          return this.emit({ type: name });
      }
    },
    /**
     * emit event type
     * @param {object} data data for event
     * @public
     */
    emit(data: { type: string; value?: number | boolean }) {
      /** Triggered when button is clicked
       * @event control
       * @type {Event}
       */
      this.$emit("control", data);
    },
  },
});
</script>
<style scoped>
/* video controls on fullscreen */
.video-controls {
  position: absolute;
  bottom: 0%;
  width: 100%;
  background-color: white;
  z-index: 1;
}

/* opacity  */
.opacity {
  opacity: 0.5;
}
.opacity-reduce {
  opacity: 0.8;
}

.camera-controls {
  position: absolute;
  z-index: 1;
  width: 100%;
  height: 100%;
}

/* remove range slider thumb animation */
.v-slider__thumb::before {
  content: none !important;
}
</style>
<docs>
The Media Card card is used to display camera live feeds with easy to use controls.
### Features
* Pan, Tilt, Zoom controls
* Sound control
* Full screen control
</docs>