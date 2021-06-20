<template>
  <div class="inline-flex">
    <el-button
        class="hover:shadow-lg"
    >
      <div class="inline-flex h-20 w-20 bg-white focus:overflow-ellipsis">
        <svg
            v-if="data.type === 0"
            class="icon" viewBox="0 0 1024 1024" xmlns="http://www.w3.org/2000/svg"
            width="100" height="80">
          <path
              d="M853.333333 256H469.333333l-85.333333-85.333333H170.666667c-46.933333 0-85.333333 38.4-85.333334 85.333333v170.666667h853.333334v-85.333334c0-46.933333-38.4-85.333333-85.333334-85.333333z"
              fill="#FFA000"></path>
          <path
              d="M853.333333 256H170.666667c-46.933333 0-85.333333 38.4-85.333334 85.333333v426.666667c0 46.933333 38.4 85.333333 85.333334 85.333333h682.666666c46.933333 0 85.333333-38.4 85.333334-85.333333V341.333333c0-46.933333-38.4-85.333333-85.333334-85.333333z"
              fill="#FFCA28"></path>
        </svg>

        <svg
            v-if="data.type === 1"
            class="icon" viewBox="0 0 1024 1024"
            xmlns="http://www.w3.org/2000/svg" width="100" height="80"
        >
          <path d="M853.333333 960H170.666667V64h469.333333l213.333333 213.333333z" fill="#90CAF9"></path>
          <path d="M821.333333 298.666667H618.666667V96z" fill="#E1F5FE"></path>
          <path
              d="M341.333333 448h362.666667v42.666667H341.333333zM341.333333 533.333333h277.333334v42.666667H341.333333zM341.333333 618.666667h362.666667v42.666666H341.333333zM341.333333 704h277.333334v42.666667H341.333333z"
              fill="#1976D2"></path>
        </svg>
      </div>

      <div class="break-all truncate w-24 overflow-hidden">
        {{ data.label }}
      </div>
    </el-button>
  </div>
</template>

<script>


export default {
  name: "File",
  props: {
    data: Object,
  },
  data() {
    return {}
  }
}
</script>

<style scoped>

</style>