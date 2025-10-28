<template>
  <span>
    <Dialog
      data-slot="dialog"
      :open="linkMenuIsActive"
      :default-open="linkMenuIsActive"
      @update:open="
        (open) => {
          if (!open) hideLinkMenu();
        }
      "
      @showing="showLinkMenu"
      @close-via-escape="hideLinkMenu"
      data-testid="confirm-action-modal"
      tabindex="-1"
      role="dialog"
    >
      <DialogContent
        class="overflow-hidden bg-white p-0 py-4"
        @close="hideLinkMenu"
      >
        <ScrollArea
          class="modal-scrollarea rounded-2xl [&>div>div]:flex [&>div>div]:flex-col [&>div>div]:gap-4"
        >
          <div class="px-6 py-1.5 bg-white">
            <template v-if="withFileUpload">
              <span
                class="inline-block cursor-pointer [&::first-letter]:uppercase capitalize font-bold text-base border-b-2 mr-4"
                :class="{
                  'text-primary border-primary': linkMode == 'url',
                  'text-foreground border-transparent': linkMode != 'url',
                }"
                @click="linkMode = 'url'"
                v-text="ttt('url')"
              >
              </span>

              <span
                class="inline-block cursor-pointer [&::first-letter]:uppercase capitalize font-bold text-base border-b-2"
                :class="{
                  'text-primary border-primary': linkMode == 'file',
                  'text-foreground border-transparent': linkMode != 'file',
                }"
                @click="linkMode = 'file'"
                v-text="ttt('file upload')"
              >
              </span>
            </template>
          </div>

          <div class="px-6 bg-white" style="padding-bottom: 32px">
            <div v-show="linkMode == 'url'">
              <div class="flex flex-col">
                <label class="text-sm mb-1 ml-1" v-text="ttt('url')"> </label>

                <Input
                  class="h-10 bg-accent-foreground text-lg"
                  type="text"
                  v-model="url"
                  placeholder="https://"
                />

                <div
                  class="ml-1 mt-1 text-xs text-80"
                  v-text="
                    ttt('external links should start with http:// or https://')
                  "
                ></div>

                <div class="flex items-center mt-3">
                  <RekaCheckbox
                    :id="'openInNewWindow_' + field.attribute"
                    v-model="openInNewWindow"
                  />
                  <label
                    class="text-sm ml-2"
                    :for="'openInNewWindow_' + field.attribute"
                    v-text="ttt('open in new window')"
                  >
                  </label>
                </div>
              </div>
            </div>

            <div v-if="withFileUpload" v-show="linkMode == 'file'">
              <div
                class="flex items-center transition-opacity duration-150"
                :class="{
                  'pointer-events-none opacity-50': uploading,
                }"
              >
                <label>
                  <input
                    ref="fileInput"
                    type="file"
                    @change="changeFile($event.target.files)"
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
                  <span v-if="!file" v-text="ttt('no file selected')"> </span>

                  <span v-if="file" v-text="filename"> </span>
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
                class="w-full h-2 mt-3"
                :class="{
                  'bg-gray-200': uploading,
                }"
              >
                <div
                  class="bg-primary-400 h-full"
                  :style="{
                    width: uploadProgress + '%',
                  }"
                ></div>
              </div>
            </div>

            <div class="mt-8">
              <label
                class="block text-sm mb-1 ml-1"
                v-text="ttt('custom css classes')"
              >
              </label>

              <Input
                class="h-10 bg-accent-foreground text-lg"
                type="text"
                v-model="extraClasses"
              />

              <label class="block text-sm mt-3 mb-1 ml-1" v-text="ttt('title')">
              </label>

              <Input
                class="h-10 bg-accent-foreground text-lg"
                type="text"
                v-model="title"
              />

              <div
                class="mt-3"
                style="display: grid; grid-template-columns: 1fr 1fr 1fr"
              >
                <div class="flex items-center">
                  <RekaCheckbox
                    :id="'nofollow_' + field.attribute"
                    v-model="nofollow"
                  />
                  <label
                    class="text-sm ml-2"
                    :for="'nofollow_' + field.attribute"
                    v-text="'nofollow'"
                  >
                  </label>
                </div>

                <div class="flex items-center">
                  <RekaCheckbox
                    :id="'noopener_' + field.attribute"
                    v-model="noopener"
                  />
                  <label
                    class="text-sm ml-2"
                    :for="'noopener_' + field.attribute"
                    v-text="'noopener'"
                  >
                  </label>
                </div>

                <div class="flex items-center">
                  <RekaCheckbox
                    :id="'noreferrer_' + field.attribute"
                    v-model="noreferrer"
                  />
                  <label
                    class="text-sm ml-2"
                    :for="'noreferrer_' + field.attribute"
                    v-text="'noreferrer'"
                  >
                  </label>
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
                class="ml-auto h-10 px-6 [&::first-letter]:uppercase capitalize"
                @click="hideLinkMenu"
              >
                {{ ttt("cancel") }}
              </DdTertiaryButton>

              <DdButton
                type="button"
                ref="runButton"
                dusk="confirm-action-button"
                class="h-10 px-6 [&::first-letter]:uppercase capitalize"
                :disabled="
                  (linkMode == 'url' && !url) || (linkMode == 'file' && !file)
                "
                @click="
                  linkMode == 'url'
                    ? setLinkFromUrl($event)
                    : uploadAndAddFile($event)
                "
                autofocus
              >
                {{
                  linkMode == "url"
                    ? ttt("set link")
                    : ttt("upload and link file")
                }}
              </DdButton>
            </div>
          </DialogFooter>
        </ScrollArea>
      </DialogContent>
    </Dialog>

    <span class="whitespace-nowrap flex gap-0.5">
      <base-button
        :isActive="linkIsActive"
        :isDisabled="!linkCanBeUsed"
        :clickMethod="showLinkMenu"
        :title="!linkIsActive ? ttt('set link') : ttt('edit link')"
        icon="link"
      >
      </base-button>

      <base-button
        :isActive="linkIsActive"
        :isDisabled="!linkCanBeUsed"
        :clickMethod="unsetLink"
        :title="ttt('unset link')"
        icon="link-slash"
      >
      </base-button>
    </span>
  </span>
</template>

<script>
import buttonHovers from "../../mixins/buttonHovers";

import BaseButton from "./BaseButton.vue";

import translations from "../../mixins/translations";

export default {
  mixins: [buttonHovers, translations],

  props: ["button", "editor", "field", "mode", "fileDisk", "filePath"],

  data: function () {
    return {
      linkMenuIsActive: false,
      url: "",
      openInNewWindow: false,
      file: null,
      filename: null,
      uploadProgress: 0,
      uploading: false,
      extraClasses: "",
      linkMode: "url",
      title: "",
      nofollow: false,
      noopener: false,
      noreferrer: false,
    };
  },

  components: {
    BaseButton,
  },

  computed: {
    linkIsActive() {
      return this.editor ? this.editor.isActive("link") : false;
    },

    linkCanBeUsed() {
      return this.editor
        ? this.mode == "editor" && !this.editor.isActive("image")
        : true;
    },

    withFileUpload() {
      return (
        !this.field.linkSettings ||
        typeof this.field.linkSettings.withFileUpload != "boolean" ||
        this.field.linkSettings.withFileUpload
      );
    },
  },

  methods: {
    showLinkMenu() {
      if (this.linkIsActive) {
        let attributes = this.editor.getAttributes("link");
        this.url = attributes.href;
        this.openInNewWindow = attributes.target == "_blank" ? true : false;
        this.linkMode = attributes["tt-mode"] ? attributes["tt-mode"] : "url";
        this.extraClasses = attributes.class ? attributes.class : "";
        this.title = attributes.title ? attributes.title : "";
        this.nofollow =
          attributes.rel && attributes.rel.indexOf("nofollow") > -1
            ? true
            : false;
        this.noopener =
          attributes.rel && attributes.rel.indexOf("noopener") > -1
            ? true
            : false;
        this.noreferrer =
          attributes.rel && attributes.rel.indexOf("noreferrer") > -1
            ? true
            : false;
      } else {
        this.url = "";
        this.openInNewWindow = false;
        this.nofollow = false;
        this.noopener = false;
        this.noreferrer = false;
        this.linkMode = "url";
        this.extraClasses = "";
        this.title = "";
      }

      this.linkMenuIsActive = true;
    },

    hideLinkMenu() {
      this.linkMenuIsActive = false;
    },

    removeFile() {
      this.file = null;
      this.filename = null;
      this.$refs.fileInput.value = null;
    },

    resetUploading() {
      this.uploading = false;
      this.uploadProgress = 0;
    },

    changeFile(files) {
      if (files.length) {
        this.file = files[0];
        this.filename = this.file.name;
      }
    },

    uploadAndAddFile(e) {
      e.preventDefault();

      this.uploading = true;

      let data = new FormData();
      data.append("file", this.file);
      data.append("disk", this.fileDisk);
      data.append("path", this.filePath);

      const config = {
        headers: {
          "Content-Type": "multipart/form-data",
        },
        onUploadProgress: (progressEvent) =>
          (this.uploadProgress =
            (progressEvent.loaded / progressEvent.total) * 100),
      };

      axios
        .post("/nova-tiptap/api/file", data, config)
        .then(
          function (response) {
            this.resetUploading();
            this.removeFile();

            let startPosition = response.data.url.lastIndexOf("/");
            let filename = response.data.url.substr(startPosition + 1);

            let attributes = {
              href: response.data.url,
              "tt-mode": "file",
              download: filename,
            };

            this.setLink(attributes);

            this.hideLinkMenu();
          }.bind(this)
        )
        .catch(
          function (error) {
            this.resetUploading();
            this.removeFile();
          }.bind(this)
        );
    },

    setLinkFromUrl(e) {
      e.preventDefault();

      let attributes = {
        href: this.url,
        "tt-mode": "url",
      };

      if (this.openInNewWindow) {
        attributes.target = "_blank";
      } else {
        attributes.target = "_self";
      }

      this.setLink(attributes);

      this.hideLinkMenu();
    },

    setLink(attributes) {
      if (this.extraClasses) {
        attributes.class = this.extraClasses;
      }
      if (this.title) {
        attributes.title = this.title;
      }
      if (this.nofollow || this.noopener || this.noreferrer) {
        attributes.rel = "";
        if (this.nofollow) {
          attributes.rel += "nofollow ";
        }
        if (this.noopener) {
          attributes.rel += "noopener ";
        }
        if (this.noreferrer) {
          attributes.rel += "noreferrer ";
        }
        attributes.rel = _.trim(attributes.rel);
      }

      if (this.editor.isActive("image")) {
        let attributes = this.editor.getAttributes("image");
      } else {
        this.editor.chain().extendMarkRange("link").unsetLink().run();
        this.editor.chain().focus().setLink(attributes).run();
      }
    },

    unsetLink() {
      this.editor.chain().focus().unsetLink().run();
    },
  },
};
</script>
