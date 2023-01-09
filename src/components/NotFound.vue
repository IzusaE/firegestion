<template>
  <template v-if="loader">

  </template>
  <template v-else>

  </template>
</template>

<script>
export default {
  data() {
    return {
        loader: true,
        msg: "Not found"      
    };
  },           
};
</script>
