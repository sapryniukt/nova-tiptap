<template>
  <span style="z-index: 10">
    <Dialog
      data-slot="dialog"
      :open="imageMenuIsActive"
      :default-open="imageMenuIsActive"
      @update:open="
        (open) => {
          if (!open) hideImageMenu();
        }
      "
      @showing="showImageMenu"
      @close-via-escape="hideImageMenu"
      data-testid="confirm-action-modal"
      tabindex="-1"
      role="dialog"
    >
      <DialogContent
        class="overflow-hidden bg-white p-0 py-4"
        @close="hideImageMenu"
      >
        <ScrollArea
          class="modal-scrollarea rounded-2xl [&>div>div]:flex [&>div>div]:flex-col [&>div>div]:gap-4"
        >
          <div class="px-6 py-1.5 bg-white">
            <div v-if="!imageIsActive && withFileUpload">
              <span
                class="inline-block cursor-pointer [&::first-letter]:uppercase capitalize font-bold text-base border-b-2 mr-4"
                :class="{
                  'text-primary border-primary': imageMode == 'file',
                  'text-foreground border-transparent': imageMode != 'file',
                }"
                @click="imageMode = 'file'"
                v-text="ttt('file upload')"
              >
              </span>

              <span
                class="inline-block cursor-pointer [&::first-letter]:uppercase capitalize font-bold text-base border-b-2"
                :class="{
                  'text-primary border-primary': imageMode == 'url',
                  'text-foreground border-transparent': imageMode != 'url',
                }"
                @click="imageMode = 'url'"
                v-text="ttt('external url')"
              >
              </span>
            </div>

            <div v-if="!imageIsActive" style="padding-top: 32px">
              <div v-if="withFileUpload" v-show="imageMode == 'file'">
                <div
                  class="flex items-center"
                  :class="{
                    'pointer-events-none opacity-50': uploading,
                  }"
                >
                  <label>
                    <input
                      ref="fileInput"
                      type="file"
                      @change="changeFile($event.target.files)"
                      accept="image/*"
                      class="h-10 absolute"
                      style="opacity: 0"
                    />
                    <DdOutlineButton size="lg" type="button">
                      <span
                        class="[&::first-letter]:uppercase capitalize"
                        v-text="ttt('select file')"
                      ></span>
                    </DdOutlineButton>
                  </label>

                  <div class="h-16 flex items-center" style="margin-left: 16px">
                    <span v-if="!preview" v-text="ttt('no file selected')">
                    </span>
                    <img
                      v-if="preview"
                      :src="preview"
                      class="w-auto"
                      style="height: 64px"
                    />
                  </div>

                  <div
                    v-if="file"
                    @click="removeFile()"
                    class="cursor-pointer text-xl text-primary"
                    style="margin-left: 16px"
                  >
                    <Icon type="trash" />
                  </div>
                </div>

                <div
                  class="w-full h-2"
                  :class="{
                    'bg-gray-200': uploading,
                  }"
                  style="margin-top: 16px"
                >
                  <div
                    class="bg-primary-400 h-full"
                    :style="{
                      width: uploadProgress + '%',
                    }"
                  ></div>
                </div>
              </div>

              <div class="" v-show="imageMode == 'url'">
                <div class="flex flex-col">
                  <label class="text-sm mb-1 ml-1" v-text="ttt('url')"></label>
                  <Input
                    class="h-10 bg-accent-foreground text-lg"
                    type="text"
                    v-model="url"
                    placeholder="https://"
                  />
                </div>
              </div>
            </div>

            <div style="padding-top: 32px">
              <div class="flex flex-col">
                <label
                  class="text-sm mb-1 ml-1"
                  v-text="ttt('custom css classes')"
                ></label>
                <Input
                  class="h-10 bg-accent-foreground text-lg"
                  type="text"
                  v-model="extraClasses"
                />
              </div>

              <div class="grid grid-cols-2 gap-3 mt-3">
                <div class="flex flex-col">
                  <label
                    class="text-sm mb-1 ml-1"
                    v-text="ttt('title')"
                  ></label>
                  <Input
                    class="h-10 bg-accent-foreground text-lg"
                    type="text"
                    v-model="title"
                  />
                </div>

                <div class="flex flex-col">
                  <label
                    class="text-sm mb-1 ml-1"
                    v-text="ttt('alt text')"
                  ></label>
                  <Input
                    class="h-10 bg-accent-foreground text-lg"
                    type="text"
                    v-model="alt"
                  />
                </div>
              </div>
            </div>
          </div>

          <DialogFooter
            class="sticky bottom-0 top-auto bg-white/90 px-6 pb-2 pt-4 shadow-footer backdrop-blur-sm"
          >
            <div class="ml-auto flex items-center gap-4">
              <DdTertiaryButton
                component="button"
                type="button"
                dusk="cancel-action-button"
                class="ml-auto h-10 6 [&::first-letter]:uppercase capitalize"
                @click="hideImageMenu"
              >
                {{ ttt("cancel") }}
              </DdTertiaryButton>

              <DdButton
                type="button"
                ref="runButton"
                dusk="confirm-action-button"
                class="h-10 px-6 [&::first-letter]:uppercase capitalize"
                :disabled="
                  !imageIsActive &&
                  ((imageMode == 'url' && !url) ||
                    (imageMode == 'file' && !file))
                "
                @click="
                  imageIsActive
                    ? updateImage($event)
                    : imageMode == 'url'
                    ? addImageFromUrl($event)
                    : uploadAndAddImage($event)
                "
                autofocus
              >
                {{
                  imageIsActive
                    ? ttt("update image")
                    : imageMode == "url"
                    ? ttt("add image")
                    : ttt("upload and add image")
                }}
              </DdButton>
            </div>
          </DialogFooter>
        </ScrollArea>
      </DialogContent>
    </Dialog>
    <!--  -->

    <span class="whitespace-nowrap">
      <base-button
        :isActive="imageIsActive"
        :isDisabled="mode != 'editor'"
        :clickMethod="showImageMenu"
        :title="!imageIsActive ? ttt('add image') : ttt('edit image')"
        icon="photograph"
      >
      </base-button>
    </span>
  </span>
</template>

<script>
import BaseButton from "./BaseButton.vue";

import translations from "../../mixins/translations";

export default {
  mixins: [translations],

  props: ["button", "editor", "field", "mode", "imageDisk", "imagePath"],

  data: function () {
    return {
      imageMenuIsActive: false,
      file: null,
      preview: null,
      url: "",
      uploadProgress: 0,
      uploading: false,
      extraClasses: "",
      imageMode: "url",
      title: "",
      alt: "",
    };
  },

  components: {
    BaseButton,
  },

  computed: {
    imageIsActive() {
      return this.editor ? this.editor.isActive("image") : false;
    },
    withFileUpload() {
      return (
        !this.field.imageSettings ||
        typeof this.field.imageSettings.withFileUpload != "boolean" ||
        this.field.imageSettings.withFileUpload
      );
    },
    defaultMode() {
      return this.withFileUpload ? "file" : "url";
    },
  },

  methods: {
    showImageMenu() {
      if (this.imageIsActive) {
        let attributes = this.editor.getAttributes("image");
        this.url = attributes.src;
        this.imageMode = attributes["tt-mode"]
          ? attributes["tt-mode"]
          : this.defaultMode;
        this.extraClasses = attributes.class ? attributes.class : "";
        this.title = attributes.title ? attributes.title : "";
        this.alt = attributes.alt ? attributes.alt : "";
      } else {
        this.url = "";
        this.imageMode = this.defaultMode;
        this.extraClasses = "";
        this.title = "";
        this.alt = "";
      }

      this.imageMenuIsActive = true;
    },

    hideImageMenu() {
      this.imageMenuIsActive = false;
    },

    removeFile() {
      this.file = null;
      this.preview = null;
      this.$refs.fileInput.value = null;
    },

    resetUploading() {
      this.uploading = false;
      this.uploadProgress = 0;
    },

    changeFile(files) {
      if (files.length) {
        this.file = files[0];
        this.preview = URL.createObjectURL(this.file);
      }
    },

    addImageFromUrl(e) {
      e.preventDefault();

      let attributes = {
        src: this.url,
        "tt-mode": "url",
        class: this.extraClasses,
        title: this.title,
        alt: this.alt,
      };

      this.editor.chain().focus().setImage(attributes).run();

      this.hideImageMenu();
    },

    uploadAndAddImage(e) {
      e.preventDefault();

      this.uploading = true;

      let data = new FormData();
      data.append("file", this.file);
      data.append("disk", this.imageDisk);
      data.append("path", this.imagePath);

      const config = {
        headers: {
          "Content-Type": "multipart/form-data",
        },
        onUploadProgress: (progressEvent) =>
          (this.uploadProgress =
            (progressEvent.loaded / progressEvent.total) * 100),
      };

      axios
        .post("/nova-tiptap/api/image", data, config)
        .then(
          function (response) {
            this.resetUploading();
            this.removeFile();

            let attributes = {
              src: response.data.url,
              "tt-mode": "file",
              class: this.extraClasses,
              title: this.title,
              alt: this.alt,
            };

            this.editor.chain().focus().setImage(attributes).run();

            this.hideImageMenu();
          }.bind(this)
        )
        .catch(
          function (error) {
            this.resetUploading();
            this.removeFile();
            console.log(error);
          }.bind(this)
        );
    },

    updateImage(e) {
      e.preventDefault();

      let attributes = {
        class: this.extraClasses,
        title: this.title,
        alt: this.alt,
      };

      this.editor.chain().focus().updateAttributes("image", attributes).run();

      this.hideImageMenu();
    },
  },
};
</script>
