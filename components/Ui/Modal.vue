<template>
  <Transition>
    <div @keypress.esc="$emit('cancel')" v-if="modelValue" position="absolute top-0 left-0" z="50" flex="~" place="items-center" justify="center" w="screen" h="screen" bg="primary dark:primaryOp opacity-5 dark:opacity-5" backdrop="~ blur-sm">
      <div ref="modalState" flex="~ col"
      :class="{
        'max-w-75% min-w-30% max-h-70% min-h-25%': size == 'sm',
        'max-w-75% min-w-35% max-h-70% min-h-50%': size == 'md',
        'max-w-75% min-w-50% max-h-70% min-h-55%': size == 'lg',
      }"
      >
        <!-- Header -->
        <!-- <div flex="~ gap-4" place="items-center" justify="between" bg="white dark:black opacity-80 dark:opacity-80" h="50px" p="y-3 x-8" text="lg" font="medium" class="backdrop-blur-[5px] opacity-90 border-rounded-t-[10px]">
          <IconClose @click="$emit('cancel')" w="4" h="4" cursor="pointer" rounded="full" text="primaryOp dark:primary"></IconClose>
          <div flex="grow" text="xl primaryOp dark:primary right">
            <slot name="title"></slot>
          </div>
        </div> -->

        <!-- Content -->
        <div flex="grow" p="t-6 b-8 x-8" text="whitesec" overflow="y-auto" bg="primary" border="4 secondary">
          <slot></slot>
        </div>

        <!-- Footer -->
        <!-- <div flex="~ gap-2" justify="center" m="t-4">
          <button v-if="confirm" @click="$emit('confirm')">
            {{ confirm }}
          </button>
          <button @click="$emit('cancel')">Cancel</button>
        </div> -->
      </div>
    </div>
  </Transition>
</template>

<script setup>
import { onKeyStroke, onClickOutside, ref } from "#imports";

const props = defineProps({
  modelValue: {
    type: Boolean,
    default: false,
  },
  size: {
    type: String,
    default: "sm",
  },
});

// Define Emits
const emit = defineEmits(["cancel"]);

const modalState = ref(null)

onClickOutside(
  modalState,
  (event) => {
    console.log(event)
    emit("cancel")
  },
)


// Close on click ESC
onKeyStroke(["Escape"], (e) => emit("cancel"));
</script>

<style scoped>
.v-enter-active,
.v-leave-active {
  transition: opacity 0.25s ease-in-out;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}
</style>
