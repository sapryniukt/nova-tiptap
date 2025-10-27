<template>
  <DdTooltipProvider>
    <DdTooltip>
      <DdTooltipTrigger as-child>
        <DdButton
          :variant="isActive ? 'primary' : 'tertiary'"
          type="button"
          size="sm"
          class="size-8 ring-0 has-[>svg]:px-0"
          @click="callClickMethod"
        >
          <template v-if="icon">
            <Icon :type="icon" class="size-5 stroke-2" />
          </template>

          <template v-else>
            <span class="font-bold" v-html="innerHtml"></span>
          </template>
        </DdButton>
      </DdTooltipTrigger>
      <DdTooltipContent align="start" :sideOffset="-8">
        {{ title }}
      </DdTooltipContent>
    </DdTooltip>
  </DdTooltipProvider>
</template>

<script>
export default {
  props: [
    "clickMethod",
    "clickMethodParameters",
    "title",
    "isActive",
    "isDisabled",
    "icon",
    "innerHtml",
  ],

  methods: {
    callClickMethod() {
      let tmpParams = this.clickMethodParameters;
      if (tmpParams) {
        if (!typeof tmpParams != "object") {
          tmpParams = [tmpParams];
        }

        this.clickMethod(...tmpParams);
      } else {
        this.clickMethod();
      }
    },
  },
};
</script>
