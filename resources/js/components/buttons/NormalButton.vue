<template>
  <base-button
    :isActive="buttonIsActive"
    :isDisabled="mode != 'editor'"
    :clickMethod="toggleButton"
    :title="__(button)"
    :icon="icon"
  >
  </base-button>
</template>

<script>
import BaseButton from "./BaseButton.vue";

export default {
  props: ["button", "editor", "mode"],

  components: {
    BaseButton,
  },

  computed: {
    buttonIsActive() {
      if (!this.editor) {
        return false;
      }

      return this.editor.isActive(this.button);
    },

    icon() {
      if (this.button == "bold") {
        return "bold";
      } else if (this.button == "italic") {
        return "italic";
      } else if (this.button == "highlight") {
        return "paint-brush";
      } else if (this.button == "strike") {
        return "strikethrough";
      } else if (this.button == "underline") {
        return "underline";
      } else if (this.button == "blockquote") {
        return "blockquote";
      } else if (this.button == "bulletList") {
        return "list-bullet";
      } else if (this.button == "orderedList") {
        return "list-numbered";
      } else if (this.button == "horizontalRule") {
        return "minus";
      } else if (this.button == "table") {
        return "table";
      } else if (this.button == "subscript") {
        return "code-squared";
      } else if (this.button == "superscript") {
        return "code";
      } else if (this.button == "paragraph") {
        return "paragraph";
      } else if (this.button == "hardBreak") {
        return "next-line";
      } else if (this.button == "code") {
        return "code-squared";
      } else if (this.button == "codeBlock") {
        return "code";
      }

      return null;
    },
  },

  methods: {
    toggleButton() {
      var command = this.editor.chain().focus();
      if (this.button == "bold") {
        command.toggleBold();
      } else if (this.button == "code") {
        command.toggleCode();
      } else if (this.button == "italic") {
        command.toggleItalic();
      } else if (this.button == "highlight") {
        command.toggleHighlight();
      } else if (this.button == "strike") {
        command.toggleStrike();
      } else if (this.button == "underline") {
        command.toggleUnderline();
      } else if (this.button == "subscript") {
        command.toggleSubscript();
      } else if (this.button == "superscript") {
        command.toggleSuperscript();
      } else if (this.button == "blockquote") {
        command.toggleBlockquote();
      } else if (this.button == "bulletList") {
        command.toggleBulletList();
      } else if (this.button == "orderedList") {
        command.toggleOrderedList();
      } else if (this.button == "codeBlock") {
        command.toggleCodeBlock();
      } else if (this.button == "horizontalRule") {
        command.setHorizontalRule();
      } else if (this.button == "table") {
        command.insertTable({ rows: 3, cols: 3, withHeaderRow: true });
      } else if (this.button == "hardBreak") {
        command.setHardBreak();
      } else if (this.button == "paragraph") {
        command.setParagraph();
      }

      command.run();
    },
  },
};
</script>
