<template>
    <v-container class="gallery">
        <v-row>
            <v-col cols="12" lg="12">
                <v-bottom-navigation :value="pixelMode" color="primary">
                    <v-btn @click="changePixelModeTo8x8()">
                        <span>8x8 Pixel</span>
                        <v-icon>mdi-grid</v-icon>
                    </v-btn>

                    <v-btn @click="changePixelModeTo8x32()">
                        <span>8x32 Pixel</span>
                        <v-icon>mdi-grid</v-icon>
                    </v-btn>
                </v-bottom-navigation>
            </v-col>
        </v-row>
        <v-row v-if="pixelMode == 0">
            <v-col cols="12" lg="3" offset-lg="3">
                <v-card class="pa-3" elevation="4">
                    <Art :colors="colors" pixelCount="64" :func="onclick" />
                    <p></p>
                    <v-textarea filled outlined v-model="array8x8String" rows="5" hide-details></v-textarea>
                    <v-switch v-model="livedraw" label="Live draw" hide-details dense></v-switch>
                </v-card>
            </v-col>
            <v-col cols="12" lg="3">
                <v-card class="pa-3" elevation="4">
                    <v-color-picker v-model="colors" mode="hexa" dot-size="20" show-swatches swatches-max-height="240"></v-color-picker>
                </v-card>
            </v-col>
        </v-row>
        <v-row v-if="pixelMode == 1">
            <v-col cols="12" lg="7" offset-lg="1">
                <v-card class="pa-3" elevation="4">
                    <Art :colors="colors" pixelCount="256" :func="onclick" />
                    <p></p>
                    <v-textarea filled outlined v-model="array8x32String" rows="5" hide-details></v-textarea>
                    <v-switch v-model="livedraw" label="Live draw" :disabled="!sockedIsConnected" hide-details dense></v-switch>
                </v-card>
            </v-col>
            <v-col cols="12" lg="3">
                <v-card class="pa-3" elevation="4">
                    <v-color-picker v-model="colors" mode="hexa" dot-size="20" show-swatches swatches-max-height="250"></v-color-picker>
                </v-card>
            </v-col>
        </v-row>
    </v-container>
</template>
<script>
import Art from "../components/Art.vue";
export default {
    data() {
        return {
            colors: "#F44336",
            pixelMode: 0,
            active8x8Background: {},
            active8x32Background: {},
            array8x8String: "",
            array8x32String: "",
            livedraw: false,
        };
    },
    components: {
        Art,
    },
    methods: {
        onclick(id, color) {
            if (this.pixelMode == 0) {
                this.active8x8Background[id] = color.replace("#", "");
                this.array8x8String = "[";

                for (const hex of Object.values(this.active8x8Background)) {
                    this.array8x8String += rgb888ToRgb565(hexToRgb(hex)) + ",";
                }

                this.array8x8String = this.array8x8String.slice(0, -1);
                this.array8x8String += "]";
            } else {
                this.active8x32Background[id] = color.replace("#", "");
                this.array8x32String = "[";

                for (const hex of Object.values(this.active8x32Background)) {
                    this.array8x32String += rgb888ToRgb565(hexToRgb(hex)) + ",";
                }

                this.array8x32String = this.array8x32String.slice(0, -1);
                this.array8x32String += "]";
            }

            if (this.livedraw) {
                this.$socket.sendObj({
                    setScreen: {
                        bitmapAnimation: {
                            data: JSON.parse(`[${this.arrayString}]`),
                            animationDelay: 200,
                        },
                    },
                });
            }
        },
        sockedIsConnected() {
            return this.$store.state.socket.isConnected;
        },
        changePixelModeTo8x8() {
            this.pixelMode = 0;
        },
        changePixelModeTo8x32() {
            this.pixelMode = 1;
        },
    },
};

function rgb888ToRgb565(rgbArray) {
    return ((rgbArray.r & 0xf8) << 8) + ((rgbArray.g & 0xfc) << 3) + (rgbArray.b >> 3);
}

function hexToRgb(hex) {
    const shorthandRegex = /^#?([a-f\d])([a-f\d])([a-f\d])$/i;
    hex = hex.replace(shorthandRegex, function(m, r, g, b) {
        return r + r + g + g + b + b;
    });

    const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
    return result
        ? {
              r: parseInt(result[1], 16),
              g: parseInt(result[2], 16),
              b: parseInt(result[3], 16),
          }
        : null;
}
</script>
<style scoped>
#art,
.v-color-picker {
    padding-left: 0;
    padding-right: 0;
    margin-left: auto;
    margin-right: auto;
    display: block;
}
</style>
