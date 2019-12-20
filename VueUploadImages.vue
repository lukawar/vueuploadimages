<template>
  <div>
    <v-layout row>
      <v-col cols="12" sm="4">
        <div v-if="imagesList.length>0">
          <utils-image :propData="imagePreview" @cropData="cropData"></utils-image>
        </div>
        <div v-else>
          <div class="add-files-empty" @click="addFiles()">
            <v-icon center color="orange">add_circle</v-icon>
            <p align="center">
              Dodaj
              <br />zdjęcia
            </p>
          </div>
        </div>
      </v-col>
      <v-col cols="12" sm="8">
        <v-row class="margin-top-12">
          <v-col
            cols="12"
            sm="2"
            v-for="(file, key) in imagesList"
            v-bind:key="key"
            class="file-listing"
            @click="changeHighlight(key)"
          >
            <div class="thumb-container">
              <v-layout row>
                <v-col cols="12" sm="12" class="b-icon">
                  <img :src="file.path" />
                </v-col>
              </v-layout>
              <v-row no-gutters>
                <v-col cols="12" sm="4" class="b-icon">
                  <popper trigger="click" :options="{placement: 'top'}">
                    <div class="popper popper-custom">{{ file.name }}</div>
                    <i slot="reference" class="cursor-pointer display-flex align-items-center">
                      <v-icon small>info</v-icon>
                    </i>
                  </popper>
                </v-col>
                <v-col cols="12" sm="4" class="b-icon">
                  <span v-on:click="editFile( key )">
                    <v-icon small>create</v-icon>
                  </span>
                </v-col>
                <v-col cols="12" sm="4" class="b-icon">
                  <span class="remove-file" v-on:click="removeFile( key )">
                    <v-icon small>delete</v-icon>
                  </span>
                </v-col>
              </v-row>
            </div>
          </v-col>
        </v-row>
      </v-col>
      <input
        style="display: none;"
        type="file"
        accept="image/*"
        id="files"
        ref="files"
        multiple
        v-on:change="handleFilesUpload()"
      />
      <input
        style="display: none;"
        type="file"
        accept="image/*"
        :id="idEdit"
        ref="editFile"
        v-on:change="handleEditFile"
      />
    </v-layout>

    <v-layout row v-if="imagesList.length>0">
      <v-col cols="12" sm="12">
        <v-btn btn btn-success outlined v-on:click="addFiles()">
          <v-icon left color="orange">add_circle</v-icon>Dodaj kolejne zdjęcia
        </v-btn>
      </v-col>
    </v-layout>

    <vue-image-lightbox-carousel
      ref="lightbox"
      :show="lightboxOn"
      @close="$store.commit('image/LIGHTBOX_OFF')"
      :images="images"
      @change="changeHighlight"
      :showCaption="false"
    ></vue-image-lightbox-carousel>
  </div>
</template>

<script>
import { forEach, findIndex, orderBy, cloneDeep } from "lodash";
// return import(/* webpackChunkName: "lodash" */ 'lodash').then(({ default: _ })
import Popper from 'vue-popperjs'
import VueImageLightboxCarousel from 'vue-image-lightbox-carousel'
import loadImage from "blueimp-load-image";
import utilsImage from "./VueUtilsImage";
export default {
  data() {
    return {
      editKey: null,
      file: "",
      files: [],
      images: [],
      previewPath: ""
    };
  },
  components: {
    // Popper: () => import("vue-popperjs"),
    // VueImageLightboxCarousel: () => import("vue-image-lightbox-carousel"),
    // loadImage: () => import("blueimp-load-image"),
    // utilsImage: () => import("./VueUtilsImage"),
    Popper,
    VueImageLightboxCarousel,
    loadImage,
    utilsImage
  },
  props: {
    idEdit: {
      type: String,
      default: "image-edit"
    },
    uploaded: {
      type: Array
    }
  },
  computed: {
    //first issue
    imagePreview() {
      let index = findIndex(this.images, { highlight: 1 });
      // console.log('index: ' + index);
      if (index > -1) {
        this.previewPath = this.images[index].path;
        // console.log('before: ' + this.previewPath);
      } else {
        this.previewPath = this.files.length ? this.files[0].path : "";
        // console.log('second: ' + this.previewPath);
      }
      // console.log('after: ' + this.previewPath);
      return this.previewPath;
    },
    imagesList() {
      if (this.uploaded && this.uploaded.length > 0) {
        this.images = this.uploaded;
        this.files = this.uploaded;
      }
      return this.images;
    },
    editMode() {
      return this.$store.state.image.editMode;
    },
    lightboxOn() {
      return this.$store.state.image.lightBox;
    }
  },
  methods: {
    dataURLtoFile(dataurl, filename) {
      var arr = dataurl.split(","),
        mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]),
        n = bstr.length,
        u8arr = new Uint8Array(n);
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }
      return new File([u8arr], filename, { type: mime });
    },
    cropData(crop) {
      // console.log(crop);
    //   console.log("path: " + this.images[this.currentIndexImage].path);
      this.images[this.currentIndexImage].path = crop;
      let name = this.files[this.currentIndexImage].name;
      this.files[this.currentIndexImage] = this.dataURLtoFile(crop, name);
      this.$emit("fileList", this.files);
    },
    addFiles() {
      this.$refs.files.click();
    },
    editFile(key) {
      this.editKey = key;
      this.$refs.editFile.click();
    },
    handleFilesUpload() {
      let uploadedFiles = this.$refs.files.files;
      for (var i = 0; i < uploadedFiles.length; i++) {
        this.files.push(uploadedFiles[i]);
        this.createImages(uploadedFiles[i]);
      }
      this.$emit("fileList", this.files);
    },
    handleEditFile(e) {
      let files = e.target.files || e.dataTransfer.files;
      if (!files.length) {
        return false;
      }
      this.file = this.$refs.editFile.files[0];
      this.files[this.editKey] = this.file;
      forEach(files, (value, index) => {
        this.editImage(value);
      });

      if (document.getElementById(this.idEdit)) {
        document.getElementById(this.idEdit).value = "";
      }

      /*this.file = this.$refs.editFile.files[0];
                this.files[this.editKey] = this.file;
                //this.createImages(this.file);*/

      this.$emit("fileList", this.files);
    },
    createImages(file) {
      let reader = new FileReader();
      reader.onload = e => {
        let dataURI = e.target.result;
        if (dataURI) {
          if (!this.images.length) {
            this.images.push({
              name: file.name,
              path: dataURI,
              file: e.target,
              highlight: 1,
              default: 1
            });
            this.currentIndexImage = 0;
          } else {
            if (this.editKey > 0) {
              //this.images.$set(this.editKey, {name: file.name, path: dataURI, file: e.target, highlight: 0, default: 0});
              this.images[this.editKey] = {
                name: file.name,
                path: dataURI,
                file: e.target,
                highlight: 0,
                default: 0
              };
              this.images.push();
              this.editKey = null;
            } else {
              this.images.push({
                name: file.name,
                path: dataURI,
                file: e.target,
                highlight: 0,
                default: 0
              });
            }
          }
        }
      };
      reader.readAsDataURL(file);
    },
    editImage(file) {
      let reader = new FileReader();
      reader.onload = e => {
        let dataURI = e.target.result;
        if (dataURI) {
          if (this.images.length && this.images[this.currentIndexImage]) {
            this.images[this.currentIndexImage].path = dataURI;
            this.images[this.currentIndexImage].name = file.name;
          }
        }
      };
      reader.readAsDataURL(file);
    },
    removeFile(key) {
      this.files.splice(key, 1);
      this.images.splice(key, 1);
      this.$refs.files.value = "";
    },
    changeHighlight(currentIndex) {
      this.currentIndexImage = currentIndex;
      let arr = this.images;
      this.images = [];
      arr.map((item, index) => {
        if (currentIndex === index) {
          item.highlight = 1;
        } else {
          item.highlight = 0;
        }
        return item;
      });
      this.images = arr;
    }
  }
};
</script>

<style lang="scss" scoped>
input[type="file"] {
  position: absolute;
  top: -500px;
}

div.file-listing {
  width: 200px;
}

span.remove-file {
  color: red;
  cursor: pointer;
  float: right;
}
.show-image {
  height: 430px;
  border: 15px solid #fff;
  outline: 1px solid #ededed;
  background: #575757;
  display: flex;
}
.show-image img {
  margin: auto;
  max-width: 400px;
  max-height: 400px;
  overflow: hidden;
}

.thumb-container {
  border: 12px solid #fff;
  outline: 1px solid #ededed;
  display: flow-root;
  overflow: hidden;
  vertical-align: center;
}

.thumb-container img {
  max-height: 100px;
  margin: auto;
}
.popper-custom {
  background: #000;
  padding: 10px;
  border: none;
  box-shadow: none;
  color: white;
  text-align: left;
  font-size: 12px;
}
.popper-custom .popper__arrow {
  border-color: #000 transparent transparent transparent !important;
}
.b-icon {
  padding: 6px 10px 0 10px !important;
  display: flex;
}
.add-files-empty {
  border: 1px solid #ededed;
  width: 130px;
  height: 130px;
  background: #fff;
  display: grid;
  cursor: pointer;
  i {
    padding: 15px;
    font-size: xx-large;
  }
  p {
    color: #333;
  }
}
</style>