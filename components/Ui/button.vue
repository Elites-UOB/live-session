<template>
    <div flex flex-col>
      <NuxtLink v-if="to" :to="to" class="text-white text-sm" no-underline type="button" bg="secondary hover:secondary/75s focus:secondary/75s" px="6px" py="10px" :disabled="disabled" border="none" flex justify-center>
        <slot/>
      </NuxtLink>
      <button v-else text="white" type="button" text-md cursor-pointer bg="secondary hover:secondary/75s focus:secondary/75s" px="6px" py="12px" :disabled="disabled" border="none" >
        <slot/>
      </button>
    </div>
</template>

<script>
export default {
    name: 'Button',
    props: {
        to: {
          type: String,
          default: null
        },
        disabled: {
          type: Boolean,
          default: null
        }
    },
};
</script>