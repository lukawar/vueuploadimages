<template>
    <v-container>
        <v-row class="show-image centered" >

            <v-img :src="imageSource" id="imagePreview" ref="imagePreview" @click="editMode == false ? $store.commit('image/LIGHTBOX_ON') : null" v-show="editMode == false" />
            <cropper
                    v-if="editMode == true"
                    classname="cropper"
                    :src="imageSource"
                    :restrictions="pixelsRestriction"
                    :minHeight="400"
                    :minWidth="400"
                    @change="change"
                    ref="cropper"
            ></cropper>

            <canvas
                    :width="canvasWidth"
                    :height="canvasHeight"
                    id="editorCanvas"
                    ref="editorCanvas"
                    style="display: block;"
                    v-show="false"
            >
            </canvas>
        </v-row>

        <v-row v-if="editMode == false" class="toolsContainer">
            <v-col cols="12" sm="6"><v-btn outlined @click="$store.commit('image/EDIT_MODE_ON')"><v-icon left small color="orange">format_shapes</v-icon> edycja obrazka</v-btn></v-col>
        </v-row>

        <v-row v-if="editMode == true" class="toolsContainer">
            <v-col cols="12" sm="2"><v-btn outlined @click="rotateLeft"><v-icon center small color="green">rotate_left</v-icon></v-btn></v-col>
            <v-col cols="12" sm="2"><v-btn outlined @click="rotateRight"><v-icon center small color="green">rotate_right</v-icon></v-btn></v-col>
            <v-col cols="12" sm="2"><v-btn outlined @click="crop"><v-icon center small color="green">crop</v-icon></v-btn></v-col>
            <v-col cols="12" sm="6"><v-btn outlined @click="$store.commit('image/EDIT_MODE_OFF')"><v-icon left  small color="orange" >clear</v-icon> zamknij edycję</v-btn></v-col>
        </v-row>

    </v-container>
</template>

<script>
    import { Cropper } from 'vue-advanced-cropper'
    export default {
        name: "utilsImage",
        components: {
            // Cropper: () => import('vue-advanced-cropper'),
            Cropper
        },
        data() {
            return {
                mode: 'preview',
                rotation: 0,
                canvasWidth: 0,
                canvasHeight: 0,
                coordinates: {
                    width: 0,
                    height: 0,
                    left: 0,
                    top: 0
                },
            }
        },
        props: {
            propData: {
                type: String
            },
            image: {
                type: String,
                default: ''
            },
        },
        methods: {
            rotateLeft() {
                this.rotateImage(-90);
            },
            rotateRight() {
                this.rotateImage(90);
            },
            preloadImage(src) {
                let image = this.$refs.imagePreview;
                let canvas = this.$refs.editorCanvas;
                let ctx = canvas.getContext("2d");
                image.onload = function() {
                    ctx.drawImage(image, 0, 0);
                    // re
                    // console.log('reloaded.toDataURL: ' + canvas.toDataURL());
                };
                image.src = src;

            },
            rotateImage(rotate) {
                this.preloadImage(this.imageSource);
                let image = new Image();
                image.src = this.imageSource;
                // console.log('this.imageSource: ' + this.imageSource);
                let canvas = this.$refs.editorCanvas;
                let ctx = canvas.getContext("2d");
                this.canvasWidth = image.naturalHeight;
                this.canvasHeight = image.naturalWidth;
                self = this;
                image.onload = function() {
                    ctx.clearRect(0, 0, canvas.width, canvas.height);
                    ctx.save();
                    let cwidth = image.height/2;
                    let cheight = image.width/2;
                    let cdimension = {cwidth, cheight};
                    let angle = (Math.PI / 180) * rotate;

                    ctx.translate(cwidth, cheight);
                    ctx.rotate(angle, cdimension);

                    ctx.translate(-cheight, -cwidth);
                    ctx.drawImage(image, 0, 0, image.width, image.height);

                    // console.log('canvas.toDataURL: ' + canvas.toDataURL());
                    let dataURL = canvas.toDataURL();
                    self.$emit('cropData', dataURL);
                    ctx.restore();
                };
            },
            crop() {
                const {coordinates, canvas} = this.$refs.cropper.getResult();
                this.coordinates = coordinates;
                this.$emit('cropData', canvas.toDataURL());
            },
            change({coordinates, canvas}) {
                //console.log(coordinates, canvas)
            },
            pixelsRestriction(minWidth, minHeight, maxWidth, maxHeight, imageWidth, imageHeight) {
                return {
                    minWidth: minWidth,
                    minHeight: minHeight,
                    maxWidth: maxWidth,
                    maxHeight: maxHeight,
                }
            },
        },

        mounted() {
            // this.canvas = this.$refs.editorCanvas;
            // this.context = this.canvas.getContext('2d');
        },
        computed: {
            imageSource() {
                // console.log('propData: ' + this.propData);
                return this.propData;
            },
            editMode() {
                return this.$store.state.image.editMode
            }
        },
        created() {

        }
    }
</script>

<style lang="scss" scoped>
    .show-image {
        height: 430px;
        border: 15px solid #fff;
        outline: 1px solid #ededed;
        background: #575757;
        display: flex;
        img {
            margin: auto;
            max-width: 400px;
            max-height: 400px;
            overflow: hidden;
        }
    }
    .cropper {
        height: 600px;
        background: #DDD;
    }

</style>