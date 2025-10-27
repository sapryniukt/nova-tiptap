<template>
  <div class="flex gap-1 p-1 leading-none text-xs">
    <DdButton
      v-for="button in buttons"
      :key="button.title"
      type="button"
      size="xs"
      @click="button.command()"
      :disabled="button.disabledIf()"
    >
      {{ button.title }}
    </DdButton>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      buttons: [
        {
          command: () => this.editor.chain().focus().addRowBefore().run(),
          disabledIf: () => !this.editor.can().addRowBefore(),
          title: "add row before",
        },
        {
          command: () => this.editor.chain().focus().addRowAfter().run(),
          disabledIf: () => !this.editor.can().addRowAfter(),
          title: "add row after",
        },
        {
          command: () => this.editor.chain().focus().deleteRow().run(),
          disabledIf: () => !this.editor.can().deleteRow(),
          title: "delete row",
        },
        {
          command: () => this.editor.chain().focus().addColumnBefore().run(),
          disabledIf: () => !this.editor.can().addColumnBefore(),
          title: "add col before",
        },
        {
          command: () => this.editor.chain().focus().addColumnAfter().run(),
          disabledIf: () => !this.editor.can().addColumnAfter(),
          title: "add col after",
        },
        {
          command: () => this.editor.chain().focus().deleteColumn().run(),
          disabledIf: () => !this.editor.can().deleteColumn(),
          title: "delete col",
        },
        {
          command: () => this.editor.chain().focus().mergeCells().run(),
          disabledIf: () => !this.editor.can().mergeCells(),
          title: "merge cells",
        },
        {
          command: () => this.editor.chain().focus().splitCell().run(),
          disabledIf: () => !this.editor.can().splitCell(),
          title: "split cell",
        },
        {
          command: () => this.editor.chain().focus().toggleHeaderColumn().run(),
          disabledIf: () => !this.editor.can().toggleHeaderColumn(),
          title: "header col",
        },
        {
          command: () => this.editor.chain().focus().toggleHeaderRow().run(),
          disabledIf: () => !this.editor.can().toggleHeaderRow(),
          title: "header row",
        },
        {
          command: () => this.editor.chain().focus().toggleHeaderCell().run(),
          disabledIf: () => !this.editor.can().toggleHeaderCell(),
          title: "header cell",
        },
        {
          command: () => this.editor.chain().focus().deleteTable().run(),
          disabledIf: () => !this.editor.can().deleteTable(),
          title: "delete table",
        },
      ],
    };
  },
  props: ["editor"],
};
</script>
