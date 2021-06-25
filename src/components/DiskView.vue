<template>
  <div class="py-8 px-8 bg-gray-700 disk">
    <p> 剩余空间： {{ disk.available }}</p>
    <div
        v-for="(bit, count) in disk.bit_table"
        :key="count"
        class="rounded-sm w-4 h-4 mx-1 my-1 inline-flex"
        :style="{'background': bit? '#39d353': 'white'}"
    >
    </div>
  </div>
</template>

<script>
export default {
  name: "DiskView",
  props: {
    disk: Object
  }
}
</script>

<style scoped>
.disk {
  max-height: 45%;
  min-height: 45%;
  height: 45%;
  overflow-y: scroll;
}
p {
  color: white;
}
</style>