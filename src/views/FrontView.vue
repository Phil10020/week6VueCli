<template>
    <FrontNavbar></FrontNavbar>
    <router-view></router-view>
    <!-- ProductsView -->
</template>

// 封裝navbar
<script>
import FrontNavbar from '@/components/FrontNavbar.vue'

export default {
  components: {
    FrontNavbar
  }
}
</script>
